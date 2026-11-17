# LANvas

LANvas is a multi-user collaborative whiteboard for a local network. One computer runs the server and
participants run the desktop client. The server and clients communicate over TCP; no internet connection
is needed while using the app. The server uses only Python's standard library, and the PyQt6 client
provides the graphical interface.

## What it includes

- A shared board with freehand drawing, shapes, text, sticky notes, images, fill, and two eraser modes.
- Pages, layers, links between pages, presentation mode, zoom and pan, and PNG, JPG, PDF, and SVG export.
- Live stroke previews, participant cursors, chat and private messages.
- Rooms with unique codes, optional passwords, public or private listings, approval for entry, and host,
  admin, editor, and viewer roles.
- Per-user undo and redo, named board versions, autosave, room recovery, and automatic client reconnect.

## How the code is organized

| File | Purpose |
| --- | --- |
| `server.py` | Async TCP server. Owns rooms and validates and orders board changes, manages access and chat, and saves room state. |
| `client.py` | PyQt6 desktop app. Draws the board, handles user input, and runs a network thread that reconnects after a dropped connection. |
| `common.py` | Shared message framing, password proof, board operation reducer, and eraser geometry. |
| `test_lanvas.py` | Protocol tests that start a server and communicate with it over sockets. |
| `../LANvas_Technical_Documentation.pdf` | Detailed architecture, protocol, recovery behavior, controls, and user guide. |

The server is the source of truth for each room. It processes accepted edits in a single asyncio event
loop and attaches a sequence number to every operation. Clients apply the same operations locally for a
responsive canvas; sequence checks and snapshots bring clients back in sync if they miss an update.
Messages are JSON frames with a four-byte length prefix over TCP. Password rooms use a challenge and
proof flow so the password is not sent when a participant joins.

Room data is stored under `data/` by default. Changed rooms are autosaved, and saved rooms are loaded
when the server starts again. The server listens on TCP port `47801` by default.

## Requirements

- Python 3.9 or newer on the server and each client computer.
- All participating computers connected to the same LAN.
- PyQt6 on computers running the graphical client. It is installed as a project dependency below.

Allow inbound TCP port `47801` through the server computer's firewall. The app does not use TLS, so
traffic is unencrypted on the LAN. Clients enter the server address manually; the app does not discover
servers using UDP.

## Install with uv (recommended)

Install [uv](https://docs.astral.sh/uv/) and Python 3.9+, then from this directory run:

```sh
uv sync
```

This uses `uv.lock` to create and sync the project's `.venv`. To run the server on the host computer:

```sh
uv run python server.py
```

The server prints its LAN address and port. On each computer that will use the whiteboard, including the
host if it will also draw, run the client from this directory:

```sh
uv run python client.py
```

To install the optional test dependency as well:

```sh
uv sync --extra test
```

## Install with pip

With Python 3.9+ and pip installed, from this directory install the client dependency:

```sh
python -m pip install -r requirement.txt
```

Then start the server on the host computer and the client on every participant's computer:

```sh
python server.py
python client.py
```

Alternatively, install the project and its dependencies from `pyproject.toml`:

```sh
python -m pip install .
```

For tests, install the optional test dependency and run pytest:

```sh
python -m pip install -e '.[test]'
python -m pytest -q
```

## Create and join a room

In the client, enter the server address and your name. To host, choose **Create a room**, set any desired
password and entry options, then connect. LANvas assigns a room code; share the code and server address
with participants. To join, select a listed public room or enter its code and password. The host or an
admin can manage participants and room permissions.

For server options such as `--host`, `--port`, `--data`, `--autosave`, `--grace`, and `--log-level`, and
the full controls and protocol description, see [the technical documentation](../LANvas_Technical_Documentation.pdf).
