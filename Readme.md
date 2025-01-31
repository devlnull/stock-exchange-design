## Stock Exchange Design
To design an stock exchange project

### Functional Requirements
- **Only Stock**
- **Operations: New Order, Cancel Order**
- **Support After-Hours trading? No**
- **100 symbols**
- **millions of orders per day**
- **10 thousands of concurrent users**
- **business rules: maximum 1 million of a single stock in a day**
- **historical charts: ignore**

### Non-Functional Requirements
- **Availability:** At least 99.99%. Availability is crucial for exchanges. Downtime, even seconds, can harm reputation.
- **Fault tolerance:** Fault tolerance and fast recovery mechanism are needed to limit the impact of a production incident.
- **Latency:** Th round-trip latency should be at the milisecond level, with a particular focus on the 99th percentile latency. The round trip latency is measured from the moment a market order enters the exchange to the point where the market order returns as a filled execution. A persistently high 99th percentile latency causes a terrible user experience for a small number of users.
- **Security:** The exchange should have an account management system. For legal and compilance, the exchange performs KYC(Know Your Client) check to verify a user's identity before a new account is opened. For public resources, such as web pages containing market data, we should prevent distribute denial-of-service(DDoS) attacks.

### Back-of-the-envelop estimation
- **100 symbols**
- **500 million orders per day**
- **Stock market** is open Saturday through Thursday from 9:00 AM to 12:30 PM Asia/Tehran.  
  That’s **3.5 hours in total**.
- **QPS (Queries Per Second):**  
  QPS = 500 million ÷ (3.5 × 3,600) ≈ 40,000
- **Peak QPS:**  
  Peak QPS = 5 × QPS = 200,000  

  The trading volume is significantly higher when the market first opens in the morning and before it closes in the afternoon.

### Busienss Knowledge
#### Broker
Brokers provide a user friendly interface for retail users to place trades and view market data.

#### Institutional Clients
Institutional clients trade large volumes using special trading softwares, They don't trade frequently but when they do, it's a large volume, so they need some functions like order splitting to minimize the market impact.

#### Limit Order
A limit order is a buy or sell with a fixed price, it might not match immediately or it might just be partially matched.

#### Market Order
Market order doesn't specify a price, it is executed at a prevailing market price immediately, A market order sacrifices cost is order to guarantee the execution. it is usefull in certain fast-moving market conditions.

#### FIX
FIX protocol, which stands for Financial Information exchange protocol, was created in 1991. It is a vendur-neutral communication protocol for exchanging securities transaction information. Below shows an example of FIX:
```
8=FIX.4.2 | 9=176 | 35=8 | 49=PHLX | 56=PERS | 52=20071123-05:30:00.000 | 11=ATOMNOCCC9990900 | 20=3 | 150=E | 39=E | 55=MSFT | 167=CS |
54=1 | 38=15 | 40=2 | 44=15 | 58=PHLX EQUITY TESTING | 59=0 | 47=C | 32=0 | 31=0 | 151=15 | 14=0 | 6=0 | 10=128 |
```

### High-Level Design
![High Level Design](./assets/StochExchange_HighLevelDesign.svg)

#### Critical Path
- 1:      A client places an order via the broker's web or mobile app.
- 2:      The broker sends the order to the exchange.
- 3:      The broker enters the exchange through the client gateway. The client gateway performs basic gatekeeeping functions such as input validation, rate limiting, authentication, normalizationm etc, The client gateway then forwards the order to thec order manager.
- 4~5:    The order manager performs risk checks based on rules set by the risk manager.
- 6:      After passing risk checks, the order manager verifies therre are sufficient funds in the wallet for the order.
- 7~9:    The order is sent to the matching engine. When a match is found, the matching engine emits two execution(also called fills), with one each for the buy and sell sides. To guarantee that matching results are deterministic when replayed, both orders and executions are sequenced in the sequencer.
- 10~14:  The execution are returned to the client.

#### Reporting Flow
- 1~2:      The reporter collects all the necessary reporting fields(e.g. client_id, price, quantity, order_type, filled_quantity, remaining_quantity) from orders and executions, and writes the consolidated records to the database.


### API Design
### Data Model
### Performance