+++
title = "Architecture Overview"
description = "An overview of the various Tokio coponents"
menu = "going_deeper"
weight = 90
+++

Most networking applications are structured in a layered fashion.

- **Byte streams** are at the lowest layer. This is usually provided by TCP or
  UDP sockets. At this layer, operations are made against byte arrays and
  usually done with buffers. Besides directly manipulating the socket, this is
  also where functionality like
  [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) would reside.

* **Framing** is taking a raw stream of bytes and breaking it up into meaningful
  units. For example, an HTTP protocol could be framed into frames consisting of
  request head, response head, or body chunk. A line based protocol consists of
  `String` frames that are delineated by new line tokens. At this point, instead
  of dealing with a stream of raw bytes, we are dealing with a stream of frame
  values. In Tokio, we sometimes refer to a full duplex stream of frames as a
  **Transport**.

* A **request / response exchange** generally is where application logic starts
  appearing. At this layer, a request is issued and a response for the request
  is returned. When the request is issued, it is turned into one or more frames
  and written to a transport. Then, at some point in the future, a response to
  the request will be read from the transport, and matched with the original
  request.

* At the **application** layer, the details of how requests and responses are
  mapped onto a transport doesn't matter. A single application may be receiving
  and issuing requests for many different protocols. An HTTP server application
  will be receiving HTTP requests, and then in turn, issuing database requests
  or other HTTP requests.

Each of these layers tend to be implemented in different libraries, and the end
application will pull in the protocol implementations and just interact with
them at the request / response exchange layer.

Tokio's abstractions map on to these different layers.

## [Byte streams](#byte-streams) {#byte-streams}

[tokio-core](http://github.com/tokio-rs/tokio-core) provides the lowest level
building blocks for writing asynchronous I/O code: an [event loop](/docs/getting-started/reactor/) and the [concrete I/O types](/docs/getting-started/core/#concrete-io), such as TCP and UDP sockets. These primitives work on the byte level, just like the `std::io` types, except the Tokio types are non-blocking.

## [Framing](#framing) {#framing}

Framing is done with Tokio by first defining a frame type, usually an `enum`,
then implementing a  transport as a [`Stream + Sink`](/docs/getting-started/streams-and-sinks)
that yields the frame. The transport will handle encoding and decoding the frame
value to the raw stream of bytes. This can either be done [manually](TODO) or
using a helper like [`Codec`](/docs/getting-started/core/#io-codecs).

## [Request / Response](#request-response) {#request-response}

The request / response exchange layer is handled by Tokio's [`Service`](TODO)
trait. The `Service` trait is a simplified interface making it easy to write
network applications in a modular and reusable way, decoupled from the
underlying protocol. It is one of Tokio's fundamental abstractions. It is a
similar abstraction to Finagle's `Service`, Ruby's Rack, or Java's servlet;
however, Tokio's `Service` trait is abstract over the underlying protocol.

There are generally two ways to map request / responses to a stream of frames:
[pipelining](TODO) or [multiplexing](TODO). [tokio-proto](TODO)'s goal is to
take a transport and handle the required logic to map that to an implementation
of `Service`.

A big advantage of having a standardized `Service` interface is that it is
possible to write reusable [middleware](TODO) components that add useful
functionality.

## [Application](#application) {#application}

Generally, all the previously listed layers will be implemented in libraries.
For example, an HTTP server implementation would implement an HTTP transport,
then use [tokio-proto](TODO) to map that to a `Service`. The `Service` is what
the HTTP library would expose.

An application would depend on many different libraries, providing various
protocol implementations exposed as services, using the [futures](TODO) library
to hook everything together.