# Module 9 — Software Architecture: Publisher

Vegard Fathul Islam — 2406365332

This repository contains the publisher side of the event-driven architecture
tutorial. It connects to a RabbitMQ broker and publishes five `user_created`
events per run, to be consumed by the subscriber.

## Q&A

### a. How much data does the publisher send to the message broker in one run?

The publisher sends **5 events** per run — five calls to
`p.publish_event("user_created", ...)` with `user_id` values 1 through 5 and
`user_name` values `2406365332-Amir`, `2406365332-Budi`, `2406365332-Cica`,
`2406365332-Dira`, and `2406365332-Emir`. Each event is a
`UserCreatedEventMessage` struct containing two `String` fields, serialized
with Borsh. The byte size on the wire depends on Borsh encoding, but the
logical payload is 5 messages per run.

### b. What does `amqp://guest:guest@localhost:5672` mean?

The publisher uses the same connection URI as the subscriber because both must
connect to the same broker:

- `amqp://` — the protocol scheme (Advanced Message Queuing Protocol)
- `guest` (first) — username
- `guest` (second) — password
- `localhost` — host (the RabbitMQ broker is running on this machine)
- `5672` — port that RabbitMQ listens on for AMQP

The whole point of the architecture is that publisher and subscriber don't
talk to each other directly — they meet at this shared address (the broker),
which acts as a rendezvous point.

## RabbitMQ Running

![RabbitMQ dashboard](screenshots/rabbitmq-dashboard.jpeg)

RabbitMQ is running locally via Docker on port 5672 (AMQP) and port 15672
(management UI), using the default `guest` / `guest` credentials.

## Sending and Processing Events

![Publisher console](screenshots/publisher-console.jpeg)
![Subscriber console](screenshots/subscriber-console.jpeg)
![RabbitMQ connections](screenshots/rabbitmq-connections.jpeg)

When `cargo run` is executed in the publisher directory, the publisher process
connects to the RabbitMQ broker, publishes 5 `user_created` events to the
`user_created` queue, and exits. The subscriber, which has been idle in its
`loop {}` waiting for incoming events, picks each one up from the queue and
prints it via the `UserCreatedHandler::handle` method. Producer and consumer
never communicate directly — RabbitMQ is the intermediary, which is why I can
start and stop either side independently without affecting the other.