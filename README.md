# Notification System

This is a notification system implementing the Outbox pattern to ensure reliable and consistent message delivery. The system consists of several microservices working together to process, send, and track notifications.

## Features

* Reliable notification delivery using the Outbox pattern and Kafka.

* Ability to retrieve full notification history with statuses.

## Architecture and Components

The system includes the following microservices:

* [gRPC Gateway](https://github.com/coffee-proj/notification-gateway) — receives notification requests via gRPC.

* [Notification Manager](https://github.com/coffee-proj/notification-manager) — reads pending notifications from the outbox table in PostgreSQL and publishes them to Kafka. It also listens for delivery status updates.

* [Notification Worker](https://github.com/coffee-proj/notification-worker) — consumes notification messages from Kafka, sends the notifications, and publishes delivery status messages back to Kafka.

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
