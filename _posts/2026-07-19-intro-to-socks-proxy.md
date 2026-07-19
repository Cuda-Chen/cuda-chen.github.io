---
layout: post
title: "Introduction to SOCKS Proxy"
category: [networking]
tags: [networking, C, libcurl, programming]
---

## Preface

Recently, I was implementing a C++ network client with
proxy feature with libcurl. Just like the [HTTP proxy post]({% post_url
2026-07-12-intro-to-http-proxy %}), I made a research of SOCKS
proxy, and I wirte this post for recording my findings.

## An Overview to SOCKS Proxy

As the name suggests, SOCKS proxy operates on SOCKS protocol,
which works in the fifth layer of OSI model (HTTP works in
the seventh layer). Consequently, SOCKS protocol is an application-
agnostic protocol and has the ability of encapsulating the appliation
layer traffics.

For libcurl, it supports SOCKS 4 and SOCKS 5.

### DNS Resolution in SOCKS Protocol

For SOCKS, the protocol supports the DNS resolution by either
client or proxy explicitly:

- `socks4://` and `socks5://`: local resolution (i.e., resolve
DNS by client)
- `socks4a://` and `socks5h://`: remote resolution (DNS resolution
operated by proxy, in our case).

## How A SOCKS Proxy Works

### SOCKS4

> Reminder: SOCKS4 does not support authentication and IPv6.

1. Client connects to the proxy with **TCP connection**.
2. Client sends a connection request with: `[Version: 4] [Command: 1] [Destination Port] [Destination IP] [User ID string]`.
3. Proxy attempts to connect to the target IP.
4. Proxy responses with a single status packet (`0x5A` for request granted, `0x5B` for rejected/failed).
5. If granted, the tunnel is opened and client sends its application data.

### SOCKS5

1. Client connects to the proxy with **TCP connection**.
2. Client sends a connection request with: `[Version: 5] [Number of Auth Methods] [Auth Methods...]`.
3. Proxy replies the chosen authentication method (e.g., `0x00` for No Auth, `0x02` for Username/Password).
4. Optional: if the authentication is requested, the client sends the credentials in plaintext. Proxy then verifies the credentials and return a success/failure byte.
5. Client requests a connection with: `[Version: 5] [Command: 1 (Connect)] [Reserved: 0] [Address Type] [Destination Address] [Destination Port]`.
6. Proxy responses to client the connection status (e.g., `0x00` for Success, `0x04` for Host Unreachable).
7. Client starts sending the application payload. For proxy, it blindly forwards the raw bytes to the destiation and pipes responses back to the client.

## Conclusion

In this post, I make a recap of proxy. Then I make an introduction
to SOCKS protocol as well as SOCKS proxy. After that, I dive into
how does the SOCKS proxy works.

As there are scarse materials talking about SOCKS proxy, feel free
to make your comments if you think my post can have more information
about the SOCKS proxy!
