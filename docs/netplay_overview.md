# Netplay Architecture Overview

This document provides a high-level overview of the netplay architecture in the `Card-Forge/forge` repository.

## Core Technologies

The netplay architecture is built on top of the [Netty](https://netty.io/) framework, a non-blocking I/O client-server framework for the development of Java network applications.

## Key Components

The core of the netplay architecture is located in the `forge-gui/src/main/java/forge/gamemodes/net` directory. The key components are:

### Server-Side

*   **`FServerManager`**: This is the main class for the server. It is a singleton that is responsible for:
    *   Starting and stopping the Netty server.
    *   Managing connected clients.
    *   Broadcasting events to clients.
    *   Handling the game lobby.

### Client-Side

*   **`FGameClient`**: This is the main class for the client. It is responsible for:
    *   Connecting to the server.
    *   Sending events to the server.
    *   Receiving events from the server.
    *   Managing the client-side game lobby.

### Communication and Event Handling

*   **`NetEvent`**: This is the base class for all network events. All events that are sent between the client and server must extend this class.
*   **`GameProtocolHandler`**: This is a Netty `ChannelInboundHandlerAdapter` that is responsible for dispatching `NetEvent` objects to the correct methods. It uses reflection to invoke the appropriate method on the `FServerManager` or `FGameClient`, based on the type of the event.
*   **`CompatibleObjectEncoder` and `CompatibleObjectDecoder`**: These are Netty codecs that are responsible for serializing and deserializing `NetEvent` objects. They use Java serialization, with the addition of LZ4 compression to reduce the size of the serialized data.

## Connection and Event Flow

1.  The client initiates a connection to the server by creating an `FGameClient` object and calling the `connect()` method.
2.  The server accepts the connection in the `FServerManager` and creates a `RemoteClient` object to represent the client.
3.  The client and server can now send `NetEvent` objects to each other.
4.  When a `NetEvent` is sent, it is first encoded by the `CompatibleObjectEncoder`, which serializes the object to a byte array and compresses it with LZ4.
5.  The receiving end decodes the byte array with the `CompatibleObjectDecoder`, which decompresses the data and deserializes it back into a `NetEvent` object.
6.  The `GameProtocolHandler` then receives the `NetEvent` object and uses reflection to invoke the appropriate method on the `FServerManager` or `FGameClient`.

## Pain Points and Areas for Improvement

*   **Java Serialization**: The use of Java serialization is a known performance bottleneck. It is also not very secure, as it can be used to execute arbitrary code.
*   **Reflection**: The use of reflection to invoke methods is also a performance bottleneck.
*   **Lack of Versioning**: The current serialization mechanism does not have any versioning, which can lead to issues when the client and server are running different versions of the code.
*   **Disconnection Handling**: The current disconnection handling is not very robust. When a client disconnects, the game is simply ended.
