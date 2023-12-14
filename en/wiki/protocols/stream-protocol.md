# Stream protocol

The stream protocol is built on top of the unified socket abstraction, and is used to run reliable, TCP-like communication.

Overall, we steal the same construction as the streams used in `sosistab2`, except without any multiplexing on top of the same fingerprint-dock-fingerprint-dock 4-tuple.

## Taking ownership of sockets

A socket can be passed into either a listener (for the server) or connected to a server (for the client).

## Opening streams

While connecting to the server, the client sends a SYN message to the server endpoint.

The server responds with SYN-ACK, and the connection is now open.

A random stream-ID must be chosen.

## Sending data

Data is sent as usual. The stream-ID must stay the same.

## Closing the connection

When the connection is closed, FIN or RST is sent.

The underlying socket (or forwarding table entry) of each connection must be kept alive for at least 60 seconds, even after the connection is closed. A loop then responds to every incoming packet that is not itself an RST with an RST. This is so that the dock number is kept occupied, and so that the other side realizes that the connection is closed.
