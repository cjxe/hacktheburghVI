# Optiver OptiTrade Challenge

**VERY IMPORTANT**: Come by our booth to get login credentials for the exchange.

Look at your positions here: http://35.179.45.135:5006/optitrade

During this challenge, you will be listening to an exchange, analyse its data and then,
if you'd like, develop an automated trading system. Of course you are also welcome to
do whatever else comes to your mind! Use your imagination!

The exchange is broadcasting prices and trades on two famous stock market [indices](https://admiralmarkets.com/education/articles/general-trading/index-trading?regulator=fca):
S&P500 (SP) and Eurostoxx (ESX). That means you see how the price changes over time for those indices,
and what trades from the outside world are made (this is simulated). Your goal is to look at the data and come up with
clever strategies to (hopefully) make money!


Wanna know what we do? Watch this video: https://www.youtube.com/watch?v=gm7SrS9bs3s

### Basic Terminology
- ```bid price```: The price that someone else is willing to *buy* at. You can **sell to them** at this price.
- ```ask price```: The price that someone else is willing to *sell* at. You can **buy from them** at this price.



## The Exchange

The system is composed by:

  * The exchange, broadcasting through **UDP** the public information messages. To listen to this information,
    connect to:
    
    ```ip: 35.179.45.135, port: 7001```
    
    This is used for the following message types: ```PRICE``` and ```TRADE``` (see the protocol section for more info).

  * The exchange execution gateway, with which you communicate in order to send orders &mdash; and hopefully trade!
    To send orders to the exchange, and listen for a reply, connect to (also **UDP**):
    
    ```ip: 35.179.45.135, port: 8001```
    
    This is used for the following message types: ```ORDER``` and ```ORDER_ACK``` (see the protocol section for more info).


### Using Captured Data

If for some reason you are having difficulties connecting to our exchange, you can also download CSV files of
captured data. There are two files, ```market_data.csv``` and ```trades.csv```.

The first file contains a few hours worth of price updates to the ```ESX-FUTURE``` and ```SP-FUTURE```.
The second file contains trades that occurred on the exchange.

You can find the files here in this repo:
- ```market_data.csv```: [market_data.csv](https://github.com/dovito/optiver_hacktheburgh/blob/master/market_data.csv)
- ```trades.csv```: [trades.csv](https://github.com/dovito/optiver_hacktheburgh/blob/master/trades.csv)


### Measure Your Performance

Trading is one thing, but of course you also want to know how much money you have made or lost, right?
For that, it might be smart to track your own profit and loss (PnL), alongside your positions (which is your risk). You can visualize the order book, or come up with other metrics that can help you find an edge. Give it some thought.

You can also look at your official PnL here: http://35.179.45.135:5006/optitrade


### Bankruptcy [IMPORTANT]

You start with **20,000 EUR**. If you end up losing all of it, you go bankrupt!
When this happens, your team is out of the game for **10 minutes**. Use this time wisely to rethink your strategy and feel free to ask the Optiverians for help!

After these 10 minutes, you'll start again with 20,000 euros.

**All of you are trading on the same exchange, which means that you will compete for the same opportunities...**

**Last thing: every trade will impact your position &mdash; keep an eye on it to manage your risk exposure!**

## Protocol specification
The communication from and towards the exchange is done via a text-based, ASCII, protocol.

Every message is specific to an *instrument*, identified by a *feedcode* (i.e. `SP-FUTURE` or `ESX-FUTURE`).
Every message is composed by several fields separated by `|`.

**Important**: Send ```TYPE=SUBSCRIPTION_REQUEST``` to ```ip: 35.179.45.135, port: 7001``` as the very first message in order to subscribe to the data feed.

### Info protocol specifications

The exchange broadcasts two messages (via UDP):

  * Price message: this tells you the current price and volume for an *instrument*
  * Trade message: this tells you trades on the exchange, at which price and for how much volume


#### Price message fields

| Field | Description | Value
|------|-------------|-----------------
| TYPE | The type of message | Fixed value: `PRICE` |
| FEEDCODE | The instrument's feedcode | String | 
| BID_PRICE | The price at which you can sell | Float |
| BID_VOLUME | The volume available on the bid | Int |
| ASK_PRICE | The price at which you can buy |Float  |
| ASK_VOLUME | The volume available on the offer | Int |

Example: ```TYPE=PRICE|FEEDCODE=FOOBAR|BID_PRICE=10.0|BID_VOLUME=100|ASK_PRICE=11.0|ASK_VOLUME=20```


#### Trade message fields

| Field | Description | Value
--------|------------|----------------
| TYPE | The type of message | Fixed value: `TRADE` |
| FEEDCODE | The instrument's feedcode | String | 
| SIDE | The side of the trade | String, `BID` or `ASK` | 
| PRICE | The traded price | Float |
| VOLUME | The traded volume| Int |

Example: ```TYPE=TRADE|FEEDCODE=FOOBAR|SIDE=BID|PRICE=22.0|VOLUME=100```


### Execution protocol specification
In order to trade, you need to send an order to the exchange (via UDP). Every order is acknowledged by the exchange.

#### Order message fields

| Field | Description | Value
-------|--------------|---------------
| TYPE | The type of message | Fixed value: `ORDER` |
| USERNAME | The unique username identifying your team. Can be found on the credential section  of the handout. | String |
| PASSWORD| The password associated with your username. Can be found on the credential section  of the handout. | String |
| FEEDCODE | The instrument's feedcode | String |
| ACTION | The order action | String, `BUY` or `SELL` | 
| PRICE | The order price | Float |
| VOLUME | The order volume | Int |

Example: ```TYPE=ORDER|USERNAME=Optiver|PASSWORD=My_Password|FEEDCODE=FOOBAR|ACTION=BUY|PRICE=22.0|VOLUME=100```


#### Order ack message fields

| Field | Description | Value
-------|-------------|----------------
| TYPE | The type of message | Fixed value: `ORDER_ACK` |
| FEEDCODE | The instrument's feedcode | String |
| PRICE | The traded price | Float, optional, only set if the order traded |
| TRADED_VOLUME | The traded volume, positive if you bought, negative if you sold | Int, optional, only set if the order has been successfully processed |
| ERROR | The errord description | If something went wrong, this field is set, otherwise it is empty | String, optional, only set on error |

Examples:
- ```TYPE=ORDER_ACK|FEEDCODE=FOOBAR|PRICE=22.0|TRADED_VOLUME=100```
- ```TYPE=ORDER_ACK|FEEDCODE=FOOBAR|TRADED_VOLUME=0```
- ```TYPE=ORDER_ACK|ERROR=Invalid login credentials supplied```
