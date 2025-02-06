# Data Model
There arc two main types of data in the stock exchange.
-	Product, order, and execution
-	Order book
### Product, order, execution
A product describes the attributes of a traded symbol, like product type, trading symbol, UI display symbol, settlement currency, lot size, tick size, etc. This data doesn’t change frequently. It is primarily used for UI display. The data can be stored in any database and is highly cacheable.
An order represents the inbound instruction for a buy or sell order. An execution represents the outbound matched result. An execution is also called a fill. Not every order has an execution. The output of the matching engine contains two executions, representing the buy and sell sides of a matched order.

![Data Model](./assets/StockExchange_DataModel_ProductOrderExecution.svg)

Orders and executions are the most important data in the exchange. \
We encounter them in both flows mentioned in the high-level design, in slightly different forms. 
-	In the critical trading path, orders and executions are not stored in a database. To achieve high performance, this path executes trades in memory and leverages hard disk or shared memory to persist and share orders and executions. Specifically, orders and executions are stored in the sequencer for fast recovery, and data is archived after the market closes.
-	The reporter writes orders and executions to the database for reporting use cases like reconciliation and tax reporting.

### Order Book
An order book is a list of buy and sell orders for a specific security or financial instrument, organized by price level [L2] [L3], It is a key data structure in the matching engine for fast order matching. \
An efficient data structure for an order book must satisfy these requirements:
-	Constant lookup time. Operation includes: getting volume at a price level or between price levels.
-	Fast add/cancel/execute operations, preferably `O(1)` time complexity. Operations include: placing a new order, canceling an order, and matching an order.
-   Query best bid/ask.
-	Iterate through price levels.

![Order Share](./assets/StockExchange_OrderShare.svg)

In the example above, there is a large market buy order for 2,700 shares of X. 
The buy order matches all the sell orders in the best ask queue and the first sell order in the 100.11 price queue.
After fulfilling this large order, the bid/ask spread widens, and the price increases by one level (best ask is 100.11 now).

```csharp
class PriceLevel{
    public Price LimitPrice {get; private set;}
    public long TotalVolume {get; private set;}
    public LinkedList<Order> Orders {get; private set;}
}
class Book<Side>{
    public Side Side {get; private set;}
    public Dictionary<Price, PriceLevel> LimitDictionary {get; private set;}
}
class OrderBook{
    public Book<Buy> BuyBook {get; private set;}
    public Book<Sell> SellBook {get; private set;}
    public PriceLevel BestBid {get; private set;}
    public PriceLevel BestOffer {get; private set;}
    public Dictionary<OrderID, Order> OrderDictionary {get; private set;}
}
```
Let's review how we achieve O(1) time complexity for these operations:
1.	Placing a new order means adding a new Order to the tail of the PriceLevel. This is O(1) time complexity for a doubly-linked list.
2.	Matching an order means deleting an Order from the head of the PriceLevel. This is O(1) time complexity for a doubly-linked list.
3.	Canceling an order means deleting an Order from the OrderBook. 
We leverage the helper data structure Dictionary<OrderID, Order> OrderDictionary in the OrderBook to find the Order to cancel in 0(1) time.
Once the order is found, if the “Orders” list was a singly-linked list, the code would have to traverse the entire list to locate the previous pointer in order to delete the order. 
That would have taken O(n) time. \
Since the list is now doubly-linked, the order itself has a pointer to the previous order, which allows the code to delete the order without traversing the entire order list.

![Buy Book](./assets/StockExchange_BuyBook.svg)