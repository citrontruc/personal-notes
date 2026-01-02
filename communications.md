# Communications

## Table of Content

- [Communications](#communications)
  - [Table of Content](#table-of-content)
  - [Protocols](#protocols)
  - [Short polling](#short-polling)
  - [Long polling](#long-polling)
  - [Server-sent events](#server-sent-events)
  - [Websockets](#websockets)

## Protocols

TCP. HTTP builds on TCP and HTTPS builds on HTTP.

UDP. Lightweight, used for fast transmission with possible loss.

Sockets.

SMTP (Simple Mail Transfer Protocol). Used for email delivery. SFTP variant for files.

SSH.

## Short polling

The client periodically asks for an update to the server. If nothing happens, we get an empty response.

**Problem**: lots of messages. Can overflow server.

## Long polling

Server only responds when there are updates.

Client has to send question and wait indefinitely until there is a response.

**Problem**: needs an existing connection. Can be long.

## Server-sent events

Client creates a connection but server handles everything (one sided). Server communicates when things happen.

## Websockets

Works in both directions.

Handshake protocol to open. Data is transmitted through frames that can contain parts of the reponse. Closing a websocket through closing handshake.
