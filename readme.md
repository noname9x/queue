# DADI Queue

A lightweight, high-performance task queue built on Node JS and Redis. Includes a broker for queue management and routing, plus a worker framework for processing messages.

## Overview

- Messages are stored in a Redis queue
- Messages are received by the broker which routes them to workers for processing
- Messages are hidden from the queue while being processed
- Messages are unhidden from the queue after timeout or failure
- Messages are deleted only after successful processing or after exceeding the maximum retry value
- Super-fast async task processing
- Throttling to avoid congestion
- Scheduled processing to defer specific messages
- Logging and robust error handling

## Installation

1. Install the **@dadi/queue** module to your project:

   `npm install @dadi/queue --save`

2. Ensure you have a Redis server running and accessible

3. Ensure your project contains a *config* directory in the root, plus a *log* directory and a *workers* directory

4. Copy and rename the sample config file from the **@dadi/queue** module:

   *config.development.json*

5. Amend the config file according to the following section

6. Require the **@dadi/queue** module from your project:

   `require('@dadi/queue')`

7. Run the project to start listening for messages:

   `npm start`

## Config

* **queue**
  * **host**: (*ipaddress; default = 127.0.0.1*) The queue server host IP
  * **port**: (*port; default = 6379*) The queue server port number
* **broker**
  * **queue**: (*string; default = ''*) The queue name
  * **interval**: (*array; default = [ 0, 1, 5, 10 ]*) The polling intervals of the message broker, in seconds
  * **retries**: (*number; default = 10*) The number of times a message will be retried after failing
  * **timeout**: (*number; default = 30*) The number of seconds until a message is placed back on the queue
  * **throttle**: (*number; default = 5*) The number of workers that should execute concurrently
* **workers**
  * **path**: (*string; default = './workers'*) The absolute or relative path to the directory for worker modules
* **logging**
  * **enabled**: (*boolean; default = false*) Enable or disable logging
  * **level**: (*string; default = 'info'*) The minimum error level to be logged
  * **path**: (*string; default = './log'*) The absolute or relative path to the directory for log files
  * **filename**: (*string; default = 'error'*) The name to use for the log file, without extension
  * **extension**: (*string; default = 'log'*) The extension to use for the log file
  * **accessLog**
    * **enabled**: (*boolean; default = false*) Enable or disable access logging

## Sending messages

In most cases, the easiest way to send message to the queue is to use **@dadi/queue-wrapper** from within your app.

See the following related projects for other ways to interact with the queue:

* rsmq-cli
* rest-rsmq

## Receiving messages

Messages sent to the queue will be received by the broker and routed to a worker. During this time the message will be unavailable to other worker processes.

### Simple addressing

A message will be routed to a worker module in the *workers* directory if one exists with a matching filename.

For example, in the **@dadi/queue** module there is an example worker in a file called *hello-world.js*.

This worker would be executed when the broker receives the message: `hello-world`.

### Compound addressing

Messages can optionally contain multiple addresses separated by a colon. In this case, the broker will attempt to traverse a hierarchy in the *workers* folder.

For example, the following message…

`sms:send-reminder`

…would be routed to the following worker…

*sms/send-reminder.js*

### Message data

In addition to an address, messages can also contain data that will be passed to a worker when the message is processed.

Any part of the message following the worker address is passed as data to the worker.

To continue the example above, the following message…

`sms:send-reminder:123456`

…would be routed to the following worker…

*sms/send-reminder.js*

…with the string '123456' passed as data.

## Workers

A worker should export a function that receives 3 parameters:

* `req`
  * `message` : The full message
  * `address` : The parts of the message containing its route
  * `data` : Any additional parts of the message
  * `retries` : The remaining number of times that this message will be retried before being deleted
  * `timeout` : A Date object of when this message will be placed back on the queue
  * `age` : A Date object of when this message was received by the broker
  * `sent` : A Date object of when this message was sent
* `queue` : An instance of the queue itself for sending further messages
* `done`: A function to call when processing is complete

### Success

On success, a worker should call `done()`, which will notify the broker to delete the message. This will also release the throttle if it is currently in operation.

### Error

On error, a worker should call `done(err)`, passing either an error string or an `Error` object, which will notify the broker to log the error. The message will remain in the queue and will be retried if the message has any attempts remaining.

### Failure

Messages are deleted after they exceed the maximum number of retry attempts. Workers that need to perform additional processing when a message fails should test  `if (!req.retries)` in their error handling.

### Timeout

Workers **must** restrict their processing time to less then the timeout value specified in the config. After the timeout value the message will be unhidden from the queue and may be processed by other workers.

Be aware of any 3rd party APIs and ensure the appropriate timeout values are set.

## Common uses for a task queue

- Image processing
- Push notifications
- Big Data processing
- API integrations
- Sending email / SMS
- Replacing CRON
- Processing webhooks

## Benefits of using a task queue

- Decoupling: by creating a layer in-between processes with an implicit, data-based interface
- Redundancy: by persisting data until it has been fully processed
- Scalability: increase the processing rate by simply adding another process
- Resiliency: messages can still be added to the queue even if the processing worker is offline
- Guarantees: delivery is guaranteed and messages are processed in the order received
- Scheduling: processing can be deferred until system resources are optimal

## Contributors

## License
