# Simple Notification Service

PubSub Message Bus pushing data. 
Topics can have multiple subscribers. Routing and filtering is part of the middleware.

Types:
* Standard
* FIFO with ordering guarantee

Protocols/Destinations:
* SQS topic (standard or fifo)
* email text or json
* SMS
* HTTP POST
* Push notification platforms 
    * Firebase Cloud Messaging
    * Apple Push Notification Service
    * Amazon Device Messaging
    * Windows Push Notification Service (WNS) for Windows 8+ and Windows Phone 8.1+
    * Microsoft Push Notification Service (MPNS) for Windows Phone 7+
    * Baidu Cloud Push for Android devices in China

Traits:
	* messages are pushing, no polling needed
	* messages are provided as JSON usually, but you can request "raw delivery as given"
	* subscribers specify the protocol and endpoint that is to be called (e-mail address, URL, phone number)
	* messages are in topics (256 char, [a-zA-Z_\-]), which naming is unique within an account
	* SQS can be a consumer, to turn the synchronous pub/sub into async message processing
	* a topic can support multiple subscribers with different protocols
	* messages can be filtered per subscriber before distribution
	* topics can be recreated from scratch after deletion within 30-60 s
	* messages can have attributes, but it seems to depend on the sink (SNS and mobile push only?)
	* messages have an individual TTL (relative to time of publishing)
	* there is a "delivery status feature" measuring success/failure rates, dwell times for messages destined for some mobile platforms; can be used for alarms

Quotas:
	* topics can have up to 100 FIFO subscribers, 10m for standard
	* a topic can have 200 filter policies per region per account; can be increased per request
	* message size: 256kb of text and for SMS subscribers, i is 140 bytes; character limit depends on the encoding (160 GSM, 140 ASCII, 70 UCS-2)


Billing:

* 0.50$ per 1m requests
    * a request has a max size of 64kb, so 256kb message count as 4
* 0.06$ per 100k http deliveries
* 2.00 per 100k email deliveries
* sms delivery depends on destination country
* there is a free tier where an initial amount of requests and deliveries is free
