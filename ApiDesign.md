# API Design
Clients interact with the stock exchange via the brokers to place orders, view executions, etc. We use the RESTful conventions for the API below to specify the interface between the brokers and the client gateway. 
Note that the RESTful API might not satisfy the latency requirements of institutional clients like hedge funds. The specialized software built for these institutions likely uses a different protocol, but no matter what it is, the basic functionality mentioned below needs to be supported.

## Order
`POST /vl/order` \
This endpoint places an order. It requires authentication.
##### Parameters
- `symbol`: the stock symbol. `String`
- `side`: buy or sell. `String`
- `price`: the price of the limit order. `Long`
- `quantity`: the quantity of the order. `Long`
##### Response
##### Body:
- `id`: the ID of the order. `Long`
- `creationTime`: the system creation time of the order. `Long`
- `filledQuantity`: the quantity that has been successfully executed. `Long`
- `remainingQuantity`: the quantity still to be executed. `Long`
- `status`: new/canceled/filled. `String`
- rest of the attributes are the same as the input parameters
##### Code:
- `200`: `successful`
- `40x`: `parameter error/access denied/unauthorized`
- `500`: `server error`

## Execution
`GET /v1/execution?symbol={:symbol}&order!d={:orderId}&startTime={:startTime}& endTime={:endTime}` \
This endpoint queries execution info. It requires authentication.
##### Parameters
- `symbol`: the stock symbol. `String`
- `orderld`: the ID of the order. Optional. `String`
- `startTime`: query start time in epoch. `Long`
- `endTime`: query end time in epoch. `Long`
##### Response
##### Body:
`executions`: array with each execution in scope (sec attributes below), Array
- `id`: the ID of the execution, `Long`
- `orderld`: the ID of the order. `Long`
- `symbol`: the stock symbol. `String`
- `side`: buy or sell. `String`
- `price`: the price of the execution. `Long`
- `quantity`: the filled quantity. `Long`
##### Code:
- `200`: `successful`
- `40x`: `parameter error/access denied/unauthorized`
- `500`: `server error`

## Order book
`/v1/marketdata/order-book/L2?symbol={:symbol}&depth={:depth}` \
This endpoint queries L2 order book information for a symbol with designated depth.
##### Parameters
- `symbol`: the stock symbol. `String`
- `depth`: order book depth per side. `Integer`
- `startTime`: query start time in epoch. `Long`
- `endTime`: query end time in epoch. `Long`
##### Response
##### Body:
- `bids`: array with price and size. `Array`
- `asks`: array with price and size. `Array`
##### Code:
- `200`: `successful`
- `40x`: `parameter error/access denied/unauthorized`
- `500`: `server error`