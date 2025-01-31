### Busienss Knowledge
#### Broker
Brokers provide a user friendly interface for retail users to place trades and view market data.

#### Institutional Clients
Institutional clients trade large volumes using special trading softwares, They don't trade frequently but when they do, it's a large volume, so they need some functions like order splitting to minimize the market impact.

#### Limit Order
A limit order is a buy or sell with a fixed price, it might not match immediately or it might just be partially matched.

#### Market Order
Market order doesn't specify a price, it is executed at a prevailing market price immediately, A market order sacrifices cost is order to guarantee the execution. it is usefull in certain fast-moving market conditions.

#### Market Data Levels
Most of stock markets has three tiers of price quotes: L1 (level 1), L2, and L3. 
L1 market data contains the best bid price, ask price, and quantities.
Bid price refers to the highest price a buyer is willing to pay for a stock. Ask price refers to the lowest price a seller is willing to sell the stock.
##### L1
![Level1](./assets/StockExchange_L1.svg)
##### L2
![Level2](./assets/StockExchange_L2.svg)
##### L3
![Level3](./assets/StockExchange_L3.svg)

#### FIX
FIX protocol, which stands for Financial Information exchange protocol, was created in 1991. It is a vendur-neutral communication protocol for exchanging securities transaction information. Below shows an example of FIX:
```
8=FIX.4.2 | 9=176 | 35=8 | 49=PHLX | 56=PERS | 52=20071123-05:30:00.000 | 11=ATOMNOCCC9990900 | 20=3 | 150=E | 39=E | 55=MSFT | 167=CS |
54=1 | 38=15 | 40=2 | 44=15 | 58=PHLX EQUITY TESTING | 59=0 | 47=C | 32=0 | 31=0 | 151=15 | 14=0 | 6=0 | 10=128 |
```