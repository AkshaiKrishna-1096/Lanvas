# LANvas

**A collaborative whiteboard for your local network.**

LANvas lets a group draw and work together in real time without relying on a cloud service. Run the server on one computer, connect desktop clients over your LAN, and collaborate on shared boards, pages, and presentations.

## Features

- **Rich whiteboard tools:** freehand drawing, lines, shapes, text, sticky notes, images, fills, links, and two eraser modes.
- **Real-time collaboration:** live stroke previews, participant cursors, shared chat, and private messages.
- **Rooms and access control:** room codes, optional passwords, public or private rooms, entry approval, and host, admin, editor, and viewer roles.
- **Board organization:** multiple pages, layers, links between pages, presentation mode, zoom, and pan.
- **Reliable sessions:** per-user undo and redo, named board versions, autosave, room recovery, and automatic client reconnection.
- **Export:** save boards as PNG, JPG, PDF, or SVG.
- **Local-first operation:** server and clients communicate over TCP on your LAN; an internet connection is not needed to use the app.

## Screenshots and project media

- [Demo video](LANvas_Demo_video.mp4)
- [Project presentation](LANvas_Presentation.pptx)
- [Technical documentation and user guide](LANvas_Technical_Documentation.pdf)

Prebuilt archives are also included: [Linux](linux_executable.zip) and [Windows](window_executable.zip).

## Requirements

- Python 3.9 or newer on the server and client computers.
- PyQt6 on computers running the graphical client.
- Participating computers connected to the same local network.

The default server port is **47801/TCP**. Allow inbound connections on that port through the host computer's firewall. Clients connect by entering the server address manually; LANvas does not perform automatic server discovery. Traffic is unencrypted, so use it on a network you trust.

## Quick start

### With uv

Install [uv](https://docs.astral.sh/uv/) and Python 3.9+, then run these commands from the `lanvas/` directory:

```sh
uv sync
uv run python server.py
```

On each computer that will use the whiteboard, start the desktop client from `lanvas/`:

```sh
uv run python client.py
```

### With pip

From `lanvas/`, install the client dependency:

```sh
python -m pip install -r requirement.txt
```

Then run the server on the host computer and the client on each participant's computer:

```sh
python server.py
python client.py
```

The server prints the address and port it is listening on. In the client, enter that address and your name. To start a session, choose **Create a room** and configure its access options. Share the server address and room code with participants so they can join.

## How it works

The server is the source of truth for each room. It validates and orders board operations, then broadcasts them to connected clients. Clients apply updates locally for a responsive canvas and use sequence checks and snapshots to recover synchronization. Messages use JSON with a four-byte length prefix over TCP. Password-protected rooms use a challenge-response login flow.

Room data is stored in a `data/` directory by default. Changed rooms are autosaved and can be restored after a restart.

## Project layout

| File | Description |
| --- | --- |
| `lanvas/server.py` | Async TCP server, room management, access control, operation validation, and persistence. |
| `lanvas/client.py` | PyQt6 desktop application, canvas tools, and client networking. |
| `lanvas/common.py` | Shared message framing, authentication helpers, board operation reducer, and eraser geometry. |
| `lanvas/test_lanvas.py` | Protocol tests for the server and socket communication. |
| `LANvas_Technical_Documentation.pdf` | Architecture, protocol, recovery behavior, controls, and user guide. |

## Tests

Install the optional test dependency from `lanvas/` and run pytest:

```sh
uv sync --extra test
uv run pytest -q
```

Or with pip:

```sh
python -m pip install -e '.[test]'
python -m pytest -q
```

## Server options

The server supports command-line options for its bind address, port, data directory, autosave interval, reconnect grace period, and log level. Run `python server.py --help` from `lanvas/` for the full list.

## License

No license file is currently included. Add a license before redistributing this project under specific terms.
