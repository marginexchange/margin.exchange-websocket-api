# Public WEBSOCKET API for **https://margin.exchange**
 
The base endpoint is: **https://ws.margin.exchange**
* All requests to API return a JSON.
* All requests to API expects JSON.

- After connection to WEBSOCKET server you should `subscribe` for events.
- If your subscription includes private data - you should `authorize` first.
- After subscribing to any event you will receive confirmation and a snapshoot "of this event type".


How to send subscribe request :
```javascript
{
	"CMD": "SUBSCRIBE",
	'DATA': [
		...list of subscription objects...
	]
}
```

After each subscribe request you will receive back a list of your current subscribtions.
```javascript
{
	"CMD": "SUBSCRIPTIONS",
	'DATA': [
		...list of subscription objects...
	]
}
```


# Subscription types

## TICK_DATA
 - Requires auth: No
 
Parameters

Type | Required | Description
----- | ------ | -------------
MARKET | Yes  | Market UID or "\*" (\* - receive all active markets tick data)

Request example:

```javascript
{
	"CMD": "SUBSCRIBE",
	"DATA": [
		{
			"TYPE" : "TICK_DATA",
			"MARKET" : "BTCUSD"
		}
	]
}
```

### TICK_DATA event example
```javascript
{
	"CMD" : "TICK_DATA",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"LAST" : "6003.22",
	"BUY" : "6003.20",
	"SELL" : "6003.22"
}
```



## ORDER_BOOK
 - Requires auth: No
 
Parameters

Type | Required | Description
----- | ------ | -------------
MARKET | Yes  | Market UID
ROWS | No | Number or rows you want to get. All `change events` will also be limited to this number of rows. Default = market default.
GROUP_TO | No | Price round indicator. GROUP_TO = 0 will round price to 0 digits after point. -1/1 => to 1 digit after/before point. Default = market default.

Request example:

```javascript
{
	"CMD": "SUBSCRIBE",
	'DATA': [
		{
			"TYPE" : "ORDER_BOOK",
			"MARKET" : "BTCUSD",
			"ROWS" : 10,
			"GROUP_TO" : -1			// will round all prices to xxxxxx.y  (only 1 "y")
		}
	]
}
```

- Once you send ORDER_BOOK subscribe request you will receive SNAPSHOT first.
- Look below for SNAPSHOT data format description.
- There is 3 event types you will receive after this type of subscribtion.
- Server will maintain the number of "ROWS" you've subscriberd. 
So, if there is a price that should appear on top of list - you will receive "REMOVE" of last order first, and than "ADD" new one.


#### ORDER_BOOK event types
```javascript
{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "ADD",	// you should add this order (price) to your order book
	"ORDER" : { ... order data ...}
}

{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "UPDATE",		// you should update this order (price) at your order book
	"ORDER" : { ... order data ...}
}
	
{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "REMOVE",		// you should add this order (price) to your order book
	"ORDER" : { ... order data ...}
}
```

ORDER is:
```javascript
{
	"TYPE" : "BUY",
	"QTY" : "0.33123",
	"PRICE" : "6122.1",			// this will be rounded to "GROUP_TO" param 
	"TOTAL" : "2027.82318",
	"COUNT" : 3							// number or orders on this price
}
```

### ORDER_BOOK snapshoot event
```javascript
{
	"CMD" : "SNAPSHOT",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"TYPE" : "ORDER_BOOK",
	"DATA" : {
		"BUY" : [ ... list of orders ... ],
		"SELL" : [ ... list of orders ... ]
	}
}
```




## ORDER_HISTORY (TRADES)
 - Requires auth: No
 
Parameters

Type | Required | Description
----- | ------ | -------------
MARKET | Yes  | Market UID

Request example:

```javascript
{
	"CMD": "SUBSCRIBE",
	'DATA': [
		{
			"TYPE" : "ORDER_HISTORY",
			"MARKET" : "BTCUSD"
		}
	]
}
```

### ORDER_HISTORY snapshot event
```javascript
{
	"CMD" : "SNAPSHOT",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"TYPE" : "ORDER_HISTORY",
	"DATA" : [ ... list of my orders ... ]
}
```

#### ORDER_HISTORY (Trade) events
```javascript
{
	"CMD": "MARKET_ORDER_HISTORY",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"ORDER" : {
		"ID":4056831,
		"DT":"2019-02-15 13:05:25",
		"TYPE":"BUY",
		"QTY":"0.03042688",
		"PRICE":"3681.1",
		"TOTAL":"112.00438796"
	}
}

```





# AUTHorization to WEBSOCKET server
In order to authorize you should send request like this.
 - PAYLOAD_DATA = any string. lets take "abcd" for example. It is good to have random each time.
 - NONCE = any integer, that should be +1 each next request.
 - PAYLOAD = encode_base64(PAYLOAD_DATA + NONCE)
 - SIGANTURE = encode_base64(hmac sa256(PAYLOAD, API_SECRET))
For more details and examples see ***REST API***(https://github.com/marginexchange/margin.exchange-rest-api) doc.

```javascript
{
	"CMD" : "AUTH_API",
	"APIKEY" : "your_api_key_as_is", 
	"PAYLOAD_DATA" : "",
	"PAYLOAD" : "",
	"SIGNATURE" : "",
	"NONCE" : "autoincrementing integer"
}
```

Auth reply
```javascript
{
	"CMD" : "AUTH_API_RESULT",
	"DATA" : 1, 				// 1 = auth ok, 0 = failed
	"MESSAGE" : "Auth Successfull."		//or error message
}
```


## MY_ORDERS
 - Requires auth: No
 
Parameters

Type | Required | Description
----- | ------ | -------------
MARKET | Yes  | Market ID or "\*" for all markets
MARKET_TYPE | Required if MARKET="\*" | "EXCHANGE" or "FUNDING"

### MY_ORDERS snapshot event
```javascript
{
	"CMD" : "SNAPSHOT",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"TYPE" : "MY_ORDERS",
	"DATA" : [ ... list of my orders ... ]
}
```

**NOTE** You will not get snapshot packet if you have no active orders on this market.
**NOTE** Snapshot event can arrive in different way, splitting data packets into multiple (if you will have more than 100 orders).
*You need to join all "DATA" arrays to get full snapshot.*
In this case you will receive this:

```javascript
1)
{
	"CMD" : "PARTIAL_DATA",
	"DATA" : [ data ]
}
	
2,3,4,5,...n)
{
	"CMD" : "PARTIAL_DATA",
	"DATA" : [ data ]
}
	
...

and last packet will be:
{
	"CMD" : "PARTIAL_DATA_FINISH",
	"HEADER" : {
		"CMD" : "SNAPSHOT",
		"MARKET_UID" : "BTCUSD",
		"MARKET_ID" : 3,
		"TYPE": "MY_ORDERS"
	}
}

```

#### MY_ORDER event types
```javascript
{
	"CMD": "MY_ORDER_EVENT",
	"EVENT_TYPE" : "ADD",	// you have new order
	"ORDER" : { ... my order data ...}
}

{
	"CMD": "MY_ORDER_EVENT",
	"EVENT_TYPE" : "UPDATE",		// order data changed
	"ORDER" : { ... my order data ...}
}
	
{
	"CMD": "MY_ORDER_EVENT",
	"EVENT_TYPE" : "REMOVE",		// order is removed
	"ORDER" : { ... my order data ...}
}
```

MY ORDER is:  (maximum information is shown, some fields may be missing, depends on ORDER_TYPE and EXCHANGE_TYPE)
```javascript
{
	"MARKET_NAME": "BTC/USD",
	"MARKET_UID": "BTCUSD",
	"MARKET_ID": 3,
	"MARKET_TYPE" : "EXCHANGE",
	
	"ID": 342465233,
	"DT": "2019-01-15 12:19:38",
	"END_DT": "2019-01-16 02:10:00",	// will appear for MY_ORDER_HISTORY event
	"USER" : 44212,
	"TYPE" : "BUY",	//BUY or SELL
	"ORDER_TYPE" : "stop",		//limit, stop, conditional
	"STOP_TYPE" : "limit",	//limit, market, trailing
	"STOP_LIMIT_PRICE" : 5900,
	"STOP_TRAILING_DISTANCE" : 20,
	"CONDITIONAL_TYPE" : "limit", //limit, market
	"CONDITIONAL_LIMIT_PRICE" : 5900,
	"CONDITIONAL_SIDE" : "H",	//H or L
	"EXCHANGE_TYPE": "margin",	// margin or exchange
	"ACC_TYPE" : "MARGIN",	// wallet used for funds "EXCHANGE" or "MARGIN" or "FUNDING"
	"MARGIN_LEVERAGE": 2,
	"MARGIN_GROUP": 0,
	"QTY" : "0.35",		//current pending QTY
	"TOTAL_QTY" : "0.35",	//total order QTY
	"INITIAL_QTY" : "0.35",		// only for MY_ORDER_HISTORY event. 
	"PRICE" : "5900",			// this will be rounded to "GROUP_TO" param 
	"TOTAL" : "2065",
	"HIDDEN" : 0,		// 0 or 1
	"TIME_IN_FORCE" : 1,	// 0 or 1
	"TIME_IN_FORCE_DT": "2019-01-16 19:10:00",
	"OPTIONS": {
		"MARGIN_ACTIVE_REDUCE_ONLY": 1,
	},
	"MSG" : "",
	"NOTE" : ""
}
```

## MY_ORDER_HISTORY
 - Requires auth: Yes
 
Parameters

Type | Required | Description
----- | ------ | -------------
MARKET | Yes  | Market ID or "\*" for all markets
MARKET_TYPE | Required if MARKET="\*" | "EXCHANGE" or "FUNDING"

Will return last 20 (may change in future) your orders (cancelled /or fulfilled) sorted by END_DT.

Request example:

```javascript
{
	"CMD": "SUBSCRIBE",
	'DATA': [
		{
			"TYPE" : "MY_ORDER_HISTORY",
			"MARKET" : "BTCUSD"
		}
	]
}
```

### ORDER_HISTORY snapshot event
```javascript
{
	"CMD" : "SNAPSHOT",
	"MARKET_UID" : "BTCUSD",
	"MARKET_ID" : 3,
	"TYPE" : "MY_ORDER_HISTORY",
	"DATA" : [ ... list of my orders ... ]
}
```

DATA will hold your completed orders. ORDER structure is same as for "MY_ORDER" event.

##What happens when your order executed (partially or fully) or cancelled ?
- You will get ORDER_BOOK event with changes (if any)
- You will get MY_ORDER_EVENT with EVENT_TYPE="REMOVE"
- You will get MY_ORDER_HISTORY event with this order.
*Note* events may arrive in any order.