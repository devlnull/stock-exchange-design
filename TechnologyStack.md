## Technology Stack
What technologies we should use to implement this project?

### Client Gateway:
   - Function: Acts as the entry point for client requests, handling authentication, validation, rate limiting, and normalization.   
   - Technology Recommendations: Implement using a high-performance web framework such as `ASP.NET` or `Spring Boot`. For authentication and authorization, consider using `OAuth 2.0 protocols`
### Order Manager:
   - Function: Processes incoming orders, performs risk checks, verifies user funds, and assigns sequence IDs. Designed as a library to be utilized by both the Matching Engine and Reporter for performance optimization.   
   - Technology Recommendations: Develop as a shared library in a language like `.NET` or `C++` or `Java` to ensure high performance and compatibility.
### Sequencer:
   - Function: Assigns unique sequence IDs to incoming orders before they are processed by the Matching Engine and stamps executions from the Matching Engine.   
   - Technology Recommendations: Implement using a lightweight service in `C++` or `Rust` to achieve low-latency processing.
### Matching Engine:
   - Function: Maintains the order book, matches buy and sell orders, and generates executions.   
   - Technology Recommendations: Given the performance-critical nature, implement in a compiled language like `C++` or `Rust`. Utilize in-memory data structures and consider lock-free programming techniques to minimize latency.
### Reporter
   - Function: Generates reports on executed trades, order statuses, and other relevant metrics.   
   - Technology Recommendations: Use `.NET` or `Java` for data processing and report generation. Integrate with data visualization libraries or tools like `Grafana` for real-time monitoring.
### Data Storage:
   - Function: Stores order data, trade executions, and other relevant information.   
   - Technology Recommendations: `PostgreSQL` OR time-series databases like `TimescaleDB`(Based on PostgreSQL) OR `InfluxDB`
### Performance Considerations:
   - Design Adjustments: The decision to implement the Order Manager as a shared library aims to reduce inter-process communication overhead, thereby enhancing performance.   
   - Technology Recommendations: Ensure that the shared library is thread-safe and optimized for concurrent access. Utilize profiling tools to identify and address performance bottlenecks.