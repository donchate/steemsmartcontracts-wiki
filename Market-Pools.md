Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
  * Token Pair Naming
* Administrative Actions
  * [create](#create)
* Swapping Actions
  * [swapTokensForExactTokens](#swapTokensForExactTokens)
  * [swapExactTokensForTokens](#swapExactTokensForTokens)
* Liquidity Actions
  * [addLiquidity](#addLiquidity)
  * [removeLiquidity](#removeLiquidity)
* [Tables available](#tables-available)
  * [params](#params)
  * [batches](#batches)

# Introduction

This contract implements a constant product formula (CPF) automated market maker (AMM).
The AMM holds quantities of Hive Engine tokens whose relative price is measured according to its reserves. A constant product pricing function maps the quantities of the assets in reserves to their marginal price.
Hive Engine token holders may trade using this smart contract at the price specified by the pricing function, which is continually updated as reserves change after each trade.
They may also provide liquidity to allow for a certain pair of tokens to be traded.

Simplified pricing function: ```k = x * y```
```
where k is a constant
x, y are the available token reserves at time of trade
```

## Token Pair Naming

A price quote is defined as the price of one token in terms of another. Throughout the contract we show two tokens, the base token, which appears first and the quote token, which appears last. The price of the first token is always reflected in units of the second token.

# Actions available

### create
A trading pair must be created before liquidity can be added and trades made.
A fee of 1000 BEE is required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```

* example:
```
{
  "tokenPair": "GLD:SLV",
  "isSignedWithActiveKey": true
}
```
## Swapping Actions
The following two actions allow traders to exchange the token pair in either direction.

### swapTokensForExactTokens
A trader may wish to receive automatically traded tokens for an exact amount of input tokens. In the example below: 1 GLD is requested out of the swap and the amount required will be automatically calculated and deducted from the holder's token balance.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * tokenSymbol (string): Valid and configured token symbol that is desired
  * tokenOut (string): Amount of tokenSymbol to output

* example:
```
{
  "tokenPair": "GLD:SLV",
  "tokenSymbol": "GLD",
  "tokenOut": "1",
  "isSignedWithActiveKey": true
}
```

### swapExactTokensForTokens
A trader may wish to send an exact amount of tokens to the swap contract, and receive an automatically traded amount of tokens back. In the example below: 1 GLD is the amount being sent to the contract and a relative amount of SLV will be returned.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * tokenSymbol (string): Valid and configured token symbol that is being sent to the contract
  * tokenIn (string): Amount of tokenSymbol to send to the contract

* example:
```
{
  "tokenPair": "GLD:SLV",
  "tokenSymbol": "GLD",
  "tokenIn": "1",
  "isSignedWithActiveKey": true
}
```

## Liquidity Actions
To provide the resources for trading to occur, liquidity must be sent and stored by the smart contract.

### addLiquidity
This action will send both tokens in a given pair to the smart contract to provide liquidity for traders to swap tokens. If this is being called on a newly created market pool, the initial liquidity provider sets the starting price for the pair. If the pair tokens have orders in the HE order book, the initial price will be compared to the last price on the order book market for deviation beyond a specified value.
For existing pools, liquidity can only be added in such a way that the trading price is maintained.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * baseQuantity (string): Amount to deposit into the base token reserve (first token in pair)
  * quoteQuantity (string): Amount to deposit into the quote token reserve (second token in pair)

* example:
```
{
  "tokenPair": "GLD:SLV",
  "baseQuantity": "1000",
  "quoteQuantity": "16000",
  "isSignedWithActiveKey": true
}
```

### removeLiquidity
This action allows a liquidity provider to withdraw their tokens from the market pool. This can only be performed in such a way that the trading price is maintained.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * baseQuantity (string): Amount to withdraw from the base token reserve (first token in pair)
  * quoteQuantity (string): Amount to withdraw from the quote token reserve (second token in pair)

* example:
```
{
  "tokenPair": "GLD:SLV",
  "baseQuantity": "1000",
  "quoteQuantity": "16000",
  "isSignedWithActiveKey": true
}
```

# Tables available

## params:
contains internal contract parameters
* fields
  * poolCreationFee = the cost in BEE to start a market pool for a token pair

## pools:
contains the instance configurations
* fields
  * *_id = MongoDB internal primary key
  * *tokenPair = market pool token pair
  * baseQuantity = base token reserve
  * quoteQuantity = quote token reserve
  * basePrice = trading price base per quote (usual way of reading a pair)
  * quotePrice = trading price quote per base (reverse price)
  * baseVolume = total base tokens traded
  * quoteVolume = total quote tokens traded
  * precision = market pool token precision (minimum of either pair)
  * active = whether pool is active or suspended
  * creator = account name of initial pool creator

## liquidityPosition:
* fields
  * _id = MongoDB internal primary key
  * account = account name of the LP
  * tokenPair = token pair in which there is a LP
  * baseQuantity = amount of base token deposited
  * quoteQuantity = amount of quote token deposited
