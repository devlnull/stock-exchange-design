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