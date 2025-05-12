# Notification System

It is a notification system that provides reliable and flexible message delivery with the ability to receive a history and view the status of sending. 

The system consists of several microservices working together to process, send, and track notifications.

## Features

* **Reliable delivery**: The outbox pattern ensures atomicity of operations within the system, while kafka ensures reliable message transmission across the system.

* **Gateway**: The gateway provides gRPC methods for working with the full cycle of sending notifications: from creating a query to receiving a query history.

* **History**: The system provides a complete notification history and status:

    * Getting a list of queries.
    * Getting all the query data (content, metadata, etc.).
    * Getting the notification status for each individual recipient (The query supports sending a message to multiple recipients).

* **Scheduled sending**: The system provides the possibility of sending scheduled notifications. The system also provides a way to cancel sending a notification if it is scheduled to be sent (if the time for sending has not yet passed). Canceling a sending can be useful if, for example, plans have changed and the sending is no longer needed. Another possible case where you may need to cancel a sending is SAGA: you can make a scheduled sending and cancel the sending in the case of a compensating transaction.

* **Gateway Idempotence**: Gateway supports idempotence keys (UUID identifiers) to ignore duplicate queries. This allows you to implement things like the retry logic of interacting with the gateway without worrying about duplicate sending the notification.

## Architecture and Components

The system includes the following microservices:

* [**Gateway**](https://github.com/coffee-proj/notification-gateway) — receives notification requests via gRPC.

* [**Manager**](https://github.com/coffee-proj/notification-manager) — reads pending notifications from the outbox table in PostgreSQL and publishes them to Kafka. It also listens for delivery status updates.

* [**Worker**](https://github.com/coffee-proj/notification-worker) — consumes notification messages from Kafka, sends the notifications, and publishes delivery status messages back to Kafka.

The interaction of the services is shown in the picture below:

<div align="center">
    <img src="./assets/system.png"></img>
</div>

## How It Works

1. A client sends a notification request to either the gRPC gateway.

2. The gateway writes the request into the outbox table in PostgreSQL and saves the notification history.

3. The Notification Manager reads new entries from the outbox table and publishes them as messages to a Kafka topic.

4. The Worker consumes messages from Kafka, sends the actual notifications (e.g., email, SMS, push), then publishes a delivery status message (success, failure, error details) back to a different Kafka topic.

5. The Notification Manager listens to the status topic, updates the notification history in PostgreSQL accordingly.

6. Users can query the notification history and their statuses through either of the gateways.
