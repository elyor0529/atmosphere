```
 _______ _______ _______  _____  _______  _____  _     _ _______  ______ _______
 |_____|    |    |  |  | |     | |______ |_____] |_____| |______ |_____/ |______
 |     |    |    |  |  | |_____| ______| |       |     | |______ |    \_ |______

```
Robust RPC/Jobs Queue for Node.JS Web Apps Backed By RabbitMQ

# Features

* Robust: timeouts, retries, error-handling, auto-reconnect, etc
* Flexible: Supports multiple job queueing models
* Efficient: thin, early release of resources
* Scales: RPC and Task sub-division allows jobs to be spread across mulitple CPUs
* "Fixes" Heroku Routing: You control how and when Atmosphere distributes work
* Proven: Backed by RabbitMQ, used in production

# Usage Models

## RPC

### Local (makes request)
```coffeescript
#Include Module
atmosphere = require("atmosphere").rainMaker

#Connect to Atmosphere
atmosphere.init "requester", (err) ->
	# Check for errors
	if err?
		console.log "Could not initialize.", err
		return
	# Submit job (make the remote procedure call)
	job = 
		type: "remoteFunction" #the job type/queue name
		name: "special job" #name for this work request (passed to remote router)
		data: {msg: "useful"} #arbitrary serializable object to pass to remote function
		timeout: 30 #seconds
	atmosphere.submit job, (error, data) ->
		if error?
			console.log "Error occurred executing function.", error
			return
		console.log "RPC completed and returned:", data
```

### Remote (fulfills request)
```coffeescript
#Include Module
atmosphere = require("atmosphere").rainCloud

#Local Function to Execute (called remotely)
# -- ticket = {type, name, id}
# -- data = copy of data object passed to submit
localFunction = (ticket, data) ->
	console.log "Doing #{data.msg} work here!"
	atmosphere.doneWith()

#Which possible jobs should this server register to handle
handleJobs = 
	remoteFunction: localFunction #object key must match job.type

#Connect to Atmosphere
atmosphere.init "responder", (err) ->
	# Check for errors
	if err?
		console.log "Could not initialize.", err
		return
	#All set now we're waiting for jobs to arrive
```

## Sub-Dividing Complex Jobs

## Logging/Monitoring



# Stuff

From original code file:
1. worker functions in rain cloud apps get called like this:
  your_function(ticket, jobData)
2. When done, call doneWith(..) and give the ticket back along with any response data (must serialize to JSON)...
  atmosphere.thunder ticket, responseData
  
## Initialize (Connect to Server)

```coffeescript
#Init Cloud (Worker Server -- ex. EDA Server)
atmosphere.init.rainCloud jobTypes, (err) ->
```
  
```coffeescript
#Init Rainmaker (App Server)
atmosphere.init.rainMaker (err) ->
```

## Submit a Job

### Example Response

Message:
```json
{ data: <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64 21>,
  contentType: 'application/octet-stream' } 
```

Headers:
```json
{}
```

deliveryInfo:
```json
{ contentType: 'application/octet-stream',
  queue: 'testQ',
  deliveryTag: 1,
  redelivered: false,
  exchange: '',
  routingKey: 'testQ',
  consumerTag: 'node-amqp-19144-0.19309696881100535' }
```