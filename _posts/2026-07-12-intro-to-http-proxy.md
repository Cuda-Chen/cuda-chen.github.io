---
layout: post
title: "Introduction to HTTP Proxy"
category: [networking]
tags: [networking, C, libcurl, programming]
---

## Preface

Recently, I was implementing a C++ network client with
proxy feature with libcurl. Before the implementatiion, I made a research
about the type of proxy, and I think it is worth for a
post explaining how it works.

## What's A Proxy?

A proxy is an intermediate gateway between the source and
the internet. It routes source's traffic so that you can mask
the source IP address, filter content, or balance server loads.
For example, in enterprise network, proxies acts as a gateway
to decide which kind of content can be egressed.

As proxies is just about routing, it can route a variety types
of networking protocols. For libcurl, it supports HTTP, WebSocket, 
and SOCKS proxy. This post will only discuss HTTP proxy (specifically,
HTTP 1.1 proxy).

## An Overview to HTTP(S) Proxy

An HTTP proxy, just like its naming, is a proxy dedicated to acts
as a gateway of the HTTP traffics.

Next, let's take a look how does an HTTP(S) proxy works.

### How An HTTP Proxy Works

Let's look a standard direct request:

```
GET /path/to/resource HTTP/1.1
Host: example.com
```

A proxy request is almost akin to a direct request, but with the difference
of an Absolute-URI of the destination:

```
GET http://example.com/path/to/resource HTTP/1.1
Host: example.com
```

An here are the steps when using HTTP proxy:

1. The source client sends the Absolute-URI request over the TCP connection to the proxy.
2. The proxy parses the request then extracts the destination either the IP
adress or the domain (in above codeblock the proxy extracts `example.com).
3. Optional: The proxy performs a DNS lookup for the destination domain.
4. The proxy opens its own, seperate TCP connection to the destination server.
5. The proxy fetches the resource then stream the HTTP response back to the client
over the original connection.

### How An HTTP Proxy Works on HTTPS Traffic

As the proxy cannot read the HTTPS request because it is encrypted via TLS,
the proxy will blindly forward TCP traffics using the HTTP `CONNECT` method.

1. Over the initial TCP connection to the proxy, the client sends a plain-text
HTTP request, asking the proxy to open a tunnel to a specific host and port:
```
CONNECT example.com:443 HTTP/1.1
Host: example.com:443
```
2. The proxy resolves either IP address or domain (for this example, `example.com`)
then establish a TCP connection to the destination server.
3. Once the outbound TCP connection, sent from the proxy, is successful, the proxy
replies to the client with the following content:
```
HTTP/1.1 200 Connection Established
```
4. The client begins the TLS handshake (sending the `ClientHello`).
5. From this steps, the proxy stops interpreting the traffic as HTTP. It simply
acts as a tunnel to send/receive encrypted bytes between the client and destination server. 

## Conclusion

In this post, I make a brief introduction of the commonly seen proxies
in libcurl. I furthermore dive into how the HTTP(S) works explaining in steps.

Just make your comments so that I can realize that there are anything I should
talk more in details!
