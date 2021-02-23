# Socket Programming HOWTO
https://docs.python.org/3/howto/sockets.html#socket-howto

*Writers notes are in itallics*

- Sockets are used nearly everywhere, but they're one of the most misunderstood technologies around
- This is a broad overview of sockets, specifically TCP (STREAM) - IPv4 (INET) sockets

# Sockets
- As discussed before, this is an overview of STREAM sockets
- STREAM sockets are a descendant od INET sockets
- You'll get better behaviour using this type of sockets *Unless you're a galaxy brain and you really now what you're doing*
- The definition of a what a socket is depends on the context
- Sockets are the most popular form of interprocess communication. Although they aren't the fastest, they're very useful for cross-platform communication
- Sockets were invented at Berkeley as part of the BSD fork of Unix and they spread like wildfire *🔥🔥*

## Different types of sockets
- **Client Socket**: And end-point of a conversation. Client applications (web browsers) use these types of sockets exclusively
- **Server Socket**: Switchboard operator. Server applications use the type of sockets, along with client sockets *Recieves data sent by the client and transfers it to whoever is listening at the other end*
- **Blocking Socket**:
- **Non-blocking Socket**:

# How do sockets work?

## Client side
When you click on a link, your browser does something very similar to this *Although I don't think it'd be doing it in Python :X*

```python
# Create an INET, STREAMing socket
s = socket.socket(socket.AF_INET, socket.SOCL_STREAM)
# Connect to the webserver on port 80
s.connect(("www.python.org"), 80)
```

After `connect` completes *and thus, you're connected to the server*, the socket `s` can be used to send a request to the server *(normally the requested item is a webpage)*. The same socket will read the reply sent by the socket and then **it'll be destroyed** *(pulped, dead, that object it doesn't exist anymore!)*. **Client sockets are normally only used for one exchange**.

## Server side
What happens server side is a little bit more complex. The server first creates a **server socket** *In here, the server will probably be using Python :X*

### Creating the server socket
```python
# Create an INET, STREAMing socket
serversocket = socket.socket(socket.AD_INET, socket.SOCK_STREAM)
# Bind the socket to a host and a port
serversocket = socket.bind((socket.gethostname(), 80))
# Start listening for requests. When a socket begins listening it becomes a server socket
serversocket.listen(5)
```
There're a couple of things to note about the code block above:
- `socket.gethostname()`is used instead of `s.bind(('localhost', 80))` so that the socket is visible to the outside world. If `s.bind(('', 80))`this would make the socket reachable by any address the machine happened to have. Alo note how the hostname/port definition is inside a tuple
- The argument to `s.listen()` tells the socket that we want to queue up as many as 5 connect requests before we start refusing connections

### Server mainloop
After we create the socket, and it starts listening on port 80, we enter the mainloop of the server:
```python
while True:
    # Accept connections from outside
    (clientsocket, address) = serversocket.accept()
    
    # Now do something with the clientsocket
    # in this case, we'll pretend this is a threaded server
    ct = client_thread(clientsocket)
    ct.run()
```

There're actually three ways this loop can work:
1. Dispatch a thread that handles `clientsocket`
1. Create a new process that handle `clientsocket`
1. Restructure the app to use non-blocking sockets