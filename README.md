# FT_IRC

A lightweight IRC server written in **C++98** using non-blocking sockets and `poll(2)`.

This project implements a subset of the IRC protocol with user registration, channel management, messaging, user/channel modes, and operator-only actions.

## What I've gained from this project

- Building an event-driven TCP server with `socket`, `bind`, `listen`, `accept`, and `poll`.
- Parsing line-based protocols (`\r\n`) and handling partial reads with per-client buffering.
- Designing modular command dispatchers and IRC-style numeric replies.
- Managing stateful entities (users/channels) with access control via user and channel modes.
- Implementing protocol constraints and edge-case handling for multi-target commands.

## Prerequisites

- Linux (or Linux-compatible environment)
- `c++` compiler with C++98 support
- `make`

## Build

```bash
make
```

Binary produced: `./ircserv`

### Useful build flags

```bash
make D=1   # AddressSanitizer
make D=2   # AddressSanitizer + DEBUG logs
make D=3   # DEBUG logs
```

## Usage

```bash
./ircserv <port> <password>
```

Example:

```bash
./ircserv 6667 supersecret
```

If arguments are missing, the program prints:

```text
./ircserv <port> <password>
```

## Connection and registration flow

A client must authenticate and register before using most commands:

1. `PASS <server_password>`
2. `NICK <nickname>`
3. `USER <username> 0 * :<realname>`

After successful registration, the server sends welcome numerics (`001` to `004`).

## Implemented commands

### Core/session

- `CAP` (accepted/no-op)
- `PASS`
- `NICK`
- `USER`
- `PING`
- `QUIT`

### Messaging and discovery

- `PRIVMSG`
- `NOTICE` (handled by the same logic as `PRIVMSG`)
- `WHOIS`
- `LIST`
- `NAMES`

### Channel management

- `JOIN`
- `PART`
- `TOPIC`
- `INVITE`
- `KICK`
- `MODE` (user and channel modes)

### Operator/admin

- `OPER`
- `KILL` (dispatch key is currently lowercase `kill` in command map)

## Modes supported

- **User modes:** `i`, `r`, `o`, `O`
- **Channel modes (available set):** `O o v a i m n q p s r t k l b e I`

(Behavior is implemented for common operational modes such as invite-only, key, limit, ban/exception/invite lists, topic protection, and moderated channels.)

## High-level architecture

- `srcs/socket.cpp`: main event loop (`poll`) and per-client buffering/dispatch.
- `srcs/Server/*`: server state, socket/user/channel containers, command dispatch, send helpers.
- `srcs/Commands/*`: one file per IRC command.
- `srcs/User.cpp`: user state, registration, user-mode updates, channel membership metadata.
- `srcs/Channel.cpp`: channel state, join/part policy checks, mode lists.

## Quick manual test (netcat)

In terminal A:

```bash
./ircserv 6667 supersecret
```

In terminal B:

```bash
nc 127.0.0.1 6667
```

Then send:

```text
PASS supersecret
NICK alice
USER alice 0 * :Alice Example
JOIN #test
PRIVMSG #test :hello world
PING 123
```

## Notes / caveats

- This is a pedagogical IRC server and not a full RFC-complete implementation.
- Some command checks are permissive/strict depending on the specific command path.
- Default operator credential entries are hardcoded in the server source for testing.

