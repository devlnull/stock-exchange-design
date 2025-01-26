## Stock Exchange Design
To design an stock exchange project

### Functional Requirements
- Only Stock
- Operations: New Order, Cancel Order
- Support After-Hours trading? No
- 100 symbols
- millions of orders per day
- 10 thousands of concurrent users
- business rules: maximum 1 million of a single stock in a day
- historical charts: ignore

### Non-Functional Requirements
- Availability: At least 99.99%. Availability is crucial for exchanges. Downtime, even seconds, can harm reputation.
- Fault tolerance: Fault tolerance and fast recovery mechanism are needed to limit the impact of a production incident.
- Latency: Th round-trip latency should be at the milisecond level, with a particular focus on the 99th percentile latency. The round trip latency is measured from the moment a market order enters the exchange to the point where the market order returns as a filled execution. A persistently high 99th percentile latency causes a terrible user experience for a small number of users.
- Security: The exchange should have an account management system. For legal and compilance, the exchange performs KYC(Know Your Client) check to verify a user's identity before a new account is opened. For public resources, such as web pages containing market data, we should prevent distribute denial-of-service(DDoS) attacks.

### Back-of-the-envelop estimation
- **100 symbols**
- **500 million orders per day**
- **Stock market** is open Saturday through Thursday from 9:30 am to 4:00 pm Asia/Tehran.  
  That’s **6.5 hours in total**.
- **QPS (Queries Per Second):**  
  QPS = 500 million ÷ (6.5 × 3,600) ≈ 21,500
- **Peak QPS:**  
  Peak QPS = 5 × QPS = 107,500  

  The trading volume is significantly higher when the market first opens in the morning and before it closes in the afternoon.

### High-Level Design
### API Design
### Data Model
### Performance