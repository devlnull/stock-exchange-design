# Matching Engine
The matching engine is also called the cross engine. Here are the primary responsibilities of the matching engine:
1.	Maintain the order book for each symbol. An order book is a list of buy and sell orders for a symbol.
2.	Match buy and sell orders. A match results in two executions (one from the buy side and the other from the sell side). The matching function must be fast and accurate.
3.	Distribute the execution stream as market data.
A highly available matching engine implementation must be able to produce matches in a deterministic order. That is, given a known sequence of orders as an input, the matching engine must produce the same sequence of executions (fills) as an output when the sequence is replayed.