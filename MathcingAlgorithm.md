# Matching Algorithm

## Core Function :
```csharp
public class OrderHandler
{
    private int nextSequence;
    private string symbol;
    
    public Context HandleOrder(OrderBook orderBook, OrderEvent orderEvent)
    {
        if (orderEvent.SequenceId != nextSequence)
        {
            return Context.Error("OUT_OF_ORDER", nextSequence);
        }
        
        if (!ValidateOrder(symbol, orderEvent))
        {
            return Context.Error("INVALID_ORDER", orderEvent);
        }
        
        var order = CreateOrderFromEvent(orderEvent);
        
        return orderEvent.MsgType switch
        {
            "NEW" => HandleNew(orderBook, order),
            "CANCEL" => HandleCancel(orderBook, order),
            _ => Context.Error("INVALID_MSG_TYPE", orderEvent.MsgType)
        };
    }

    private Context HandleNew(OrderBook orderBook, Order order)
    {
        return order.Side == "BUY" ? Match(orderBook.SellBook, order) : Match(orderBook.BuyBook, order);
    }

    private Context HandleCancel(OrderBook orderBook, Order order)
    {
        if (!orderBook.OrderDictionary.ContainsKey(order.OrderId))
        {
            return Context.Error("CANNOT_CANCEL_ALREADY_MATCHED", order);
        }
        
        RemoveOrder(order);
        SetOrderStatus(order, "CANCELED");
        return Context.Success("CANCEL_SUCCESS", order);
    }

    private Context Match(Dictionary<Price, PriceLevel> book, Order order)
    {
        int leavesQuantity = order.Quantity - order.MatchedQuantity;
        
        if (book.TryGetValue(order.Price, out var priceLevel))
        {
            var orderQueue = priceLevel.Orders;
            while (orderQueue.Count > 0 && leavesQuantity > 0)
            {
                var nextOrder = orderQueue.Dequeue();
                int matched = Math.Min(nextOrder.Quantity, leavesQuantity);
                order.MatchedQuantity += matched;
                leavesQuantity -= matched;
                GenerateMatchedFill();
            }
        }
        
        return Context.Success("MATCH_SUCCESS", order);
    }

    private bool ValidateOrder(string symbol, OrderEvent orderEvent)
    {
        // Implement validation logic
        return true;
    }
    
    private Order CreateOrderFromEvent(OrderEvent orderEvent)
    {
        // Convert OrderEvent to Order
        return new Order();
    }
    
    private void RemoveOrder(Order order)
    {
        // Implement order removal logic
    }
    
    private void SetOrderStatus(Order order, string status)
    {
        // Implement order status update logic
    }
    
    private void GenerateMatchedFill()
    {
        // Implement fill generation logic
    }
}
```

### How it works?
This Code uses the FIFO (First In First Out) matching algorithm.  \
The order that comes in first at a certain price level gets matched first, and the last one gets matched last. \
There are many matching algorithms. These algorithms are commonly used in futures trading. \
For example, a FIFO with LMM (Lead Market Maker) algorithm allocates a certain quantity to the LMM based on a predefined ratio ahead of the FIFO queue, which the LMM firm negotiates with the exchange for the privilege.
##### [More Matching Algorithms Here](https://www.cmegroup.com/education/matching-algorithm-overview.html)


## Structures
```csharp
class Context
{
    public static Context Error(string errorCode, object data) => new Context { ErrorCode = errorCode, Data = data };
    public static Context Success(string successCode, object data) => new Context { SuccessCode = successCode, Data = data };
    
    public string? ErrorCode { get; private set; }
    public string? SuccessCode { get; private set; }
    public object? Data { get; private set; }
}

class OrderEvent
{
    public int SequenceId { get; private set; }
    public string MsgType { get; private set; } = string.Empty;
}

class Order
{
    public int OrderId { get; private set; }
    public string Side { get; private set; } = string.Empty;
    public decimal Price { get; private set; }
    public int Quantity { get; private set; }
    public int MatchedQuantity { get; private set; }
}

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