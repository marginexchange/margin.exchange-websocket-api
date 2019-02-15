# Public WEBSOCKET API for **https://margin.exchange**
 
The base endpoint is: **https://ws.margin.exchange**
* All requests to API return a JSON object.
* All requests to API expects JSON object as parameters.

- After connection to WEBSOCKET server you should `subscribe` for events first.
- If your subscription includes private data - you should authorize first.
- After subscribing to any EVENT you will receive confirmation and a snapshoot "of this event type".


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
MARKET | Yes  | Market ID or "*" (* - receive all active markets tick data)

Request example:

```javascript
{
	"CMD": "SUBSCRIBE",
	'DATA': [
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
	"CMD" : "TICK_DATA',
	"MARKET_ID" : "BTCUSD",
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
MARKET | Yes  | Market ID
ROWS | No | Number or rows you want to get. All hanges will also be limited to this number of rows. Default = market default.
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

- Once you send ORDER_BOOK subscribe request you will receive SNAPSHOOT first.
- Look below for SNAPSHOOT data format description.
- There is 3 event types you will receive after this type of subscribtion.
- Server will maintain the number of "ROWS" you've subscriberd. 
So, if there is a price that should appear on top of list - you will receive "REMOVE" of last order first, and than "ADD" new one.


#### ORDER_BOOK
```javascript
{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "ADD",						// you should add this order (price) to your order book
	"ORDER" : { ... order data ...}
}

{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "UPDATE",				// you should update this order (price) at your order book
	"ORDER" : { ... order data ...}
}
	
{
	"CMD": "ORDER_BOOK_EVENT",
	"EVENT_TYPE" : "REMOVE",				// you should add this order (price) to your order book
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
	"CMD" : "SNAPSHOOT",
	"MARKET_ID" : "BTCUSD",
	"TYPE" : "ORDER_BOOK",
	"DATA" : {
		"BUY" : [ ... list of orders ... ],
		"SELL" : [ ... list of orders ... ]
	}
}
```

# AUTHorization to WEBSOCKET server
In order to authorize you should send request like this.
 - PAYLOAD_DATA = any string. lets take "abcd" for example. It is good to have random each time.
 - NONCE = any integer, than should be +1 each next request.
 - PAYLOAD = encode_base64(PAYLOAD_DATA + NONCE)
 - SIGANTURE = encode_base64(hmac sa256(PAYLOAD, API_SECRET))
For more details and examples see (REST API){https://github.com/marginexchange/margin.exchange-rest-api} doc.

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
