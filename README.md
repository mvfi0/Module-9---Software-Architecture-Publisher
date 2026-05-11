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