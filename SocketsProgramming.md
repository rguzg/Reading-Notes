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
- **Server Socket**: Switchboard operator. Server applications use this type of sockets, along with client sockets *Recieves a request sent in by a client and connect the client socket with another client socket*
- **Blocking Socket**
- **Non-blocking Socket**

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
1. Create a new process that handles `clientsocket`
1. Restructure the app to use non-blocking sockets

The important thing to understand here is that, **server sockets only produce client sockets**, they don't recieve or send any data. Client Sockets are created as a response to another socket doing a `connect()`to the host and port we are bound to. As soon as we create the client socket the server goes back to listening for more connections. The two clients are free to exchange whatever information they might want to exchange.

### A quick word from our sponsors: Interprocess Communication
If you need fast IPC between two processes on one machine, you should look into **pipes** and **shared memory**. If you decide to use AF_INET sockets, bind the server to `localhost`. This will take a shortcut around some couple of layers of network code and be a little bit faster

## Using a socket
- The communication between a web browser's client socket and a web server's client socket is a peer to peer conversation
- It is the job of the designer to decide the rules for the conversation
- A common design rule is that the connecting socket *(the socket that's using `s.connect`)* starts the conversation by sending a request.

### Sharing data between sockets
There're two sets of verbs used for communication:

#### Using send and recv
- You can use `send` and `recv`:
    - `send` and `recv` operate on the network buffers, which means that they may not send all the bytes you hand them or recieve all bytes you expect from them.
    - These methods return after their associated buffers have been filled (`send`) or emptied (`recv`). They then tell you how many bytes they handled. It's the developer's responsability to call them again until the message has been completely dealt with.
    
    - When `recv` returns 0 bytes that means that the other side has closed the connection. **You will not recieve more data from this connection**, but you **may be able to send data successfully**
    - A protocol like HTTP uses a socket for only one transfer, the client sends a requests, then reads the reply and then the socket is discarded. This means that the client can easily detect the end of the reply by receiving 0 bytes.
    - If you plan on reusing a socket for further transfers you need to take into account that there's no End of Transfer indicator on a socket. 
    - Not taking all of this into account may lead to a part of the connection waiting on a `recv` forever because the socket will not tell you that there's nothing more to read.

    - These past points indicate that there're several ways we can handle sockets: using fixed length messages, delimiting messages, indicating how long the messages are or end by shutting down the connection.
    
    Example of handling sockets using fixed length messages: 
    In here `MSGLEN` represents an already known length of a message    

    ```python
    class MySocket:
        def __init__(self, sock = None):
            if sock is None:
                self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            else:
                self.sock = sock

        def connect(self, host, port):
            self.sock.connect((host,port))

        def mysend(self, msg):
            totalsent = 0

            while totalsent < MSGLEN:
                # Send from totalsent until the end of the message. If the message is not sent in its entirety
                # totalsent will be used to determine from where should send start sending bytes
                sent = self.sock.send(msg[totalsent:])

                if sent == 0:
                    raise RuntimeError('Socket connection broken')

                totalsent = totalsent + sent

        def myrecieve(self):
            chunks = []
            bytes_recieved = 0

            while bytes_recieved < MSGLEN:
                # Recieve the lowest number: 2048 bytes or MSGLEN - bytes_recieved. This is used incase the bytes left to read in the message is less than 2048
                chunk = self.sock.recv(min(MSGLEN - bytes_recieved, 2048))

                if chunk == b'':
                    raise RuntimeError('Socket connection broken')

                chunks.append(chunk)
                bytes_recieved = bytes_recieved + len(chunk)
            
            return b''.join(chunks)
    ```

    The sending code can be used for almost any messaging scheme. In Python **you send strings** and you can use `len()` to determine its length. It's mostly the receiving code that gets more complex.

    The easiest enhancement is to make the first character an indicator of message type, and have the type determine its length. Now you have two `recvs`, the first one to get the first character and determine the message length, and the second one in a loop to recieve the rest.

    If you decide to go the delimited route, you'll be receiving an arbitrary chunk size *(this should be defined beforehand in the conversation rules and it must be the same in the client and server side)* (4096 or 8192 are good network buffer sizes).

    One complication to be aware of: If your conversation protocol allows multiple messages to be sent back to back (without some kind of reply), and you pass `recv`and arbitrary chunk size, you may end up reading the start of a following message.

    Prefixing the message with its length gets more complex, because if you're just reading whatever information is in the network buffer you may note recieve the full representation of the file's length or you might not write the full representation of the file's length to the network buffer.  This can be solved by using two `recv` loops, one to recieve the file's length and another one to recieve the rest of the message *But that's a nasty solution*.
    
#### Using read and write
- You can transform the client socket into a file-like object and use `read` and `write`:
    - When using these socket verbs it's important to remember to `flush` the sockets. As we're dealing with buffered "files" here, it's a common mistake to `write` something and then `read` for a reply. Without a `flush` call there, you may wait forever for a reply because the stuff you wrote is still in your output buffer

### Sending binary data
It's possible to send binary data over a socket, but keep in mind that not all machines use the same formats for binary data! *that pesky endianness smh*

Sending the ascii representation of binary data might actually be smaller than sending the binary representation, for example, in a 32-bit machine the binary representation of 0 is `0x0000` and the ASCII character for `'0'` is `0x30`

This doesn't fit well with fixed-length messages, though. *I don't want to sound like captain obvious, but changing the length of a charcter from 4 bits to 2 bits, after sending how long the message is, is really not a good idea*

### Disconnecting
Technically, you are supposed to use `shutdown` on a socket before you `close` it. `shutdown`is an advisory to the socket at the other end and depending on the argument you pass it, it can mean "I'm not sending anymore, but I'm still listening!" or "I'm not listening anymore, Bye!". Most programmers negelct this piece of ettiquete so socket libraries normally implement the `shutdown()` call inside the `close()` method.

An example of using `shutdown` is an HTTP-like exchange. The client sends a request and then does a `shutdown(1)` this tells the server that the client is done sending, but can still receive. The server can detect when it has received the complete request when it receives zero bytes. The server sends a reply. If `send `completes successfully then the client was indeed still receiving.

Python automatically closes a socket when it's garbage collected. *But please for the love of everything that's holy, don't rely on this. If the socket disappears without a proper close, it can leave the other end hanging indefinitely*

### When Sockets Die 💀
- A socket dies when one of the other sides comes down hard (without doing a `close`)
- When this happens, the socket will probably hang. TCP will wait for a long time for a response that will never arrive *😢*
- If you're using threads, the thread that contained the socket is pretty much dead
- If the socket dies, do not try to kill the thread because you'll probably screw up your entire process

# Non-blocking sockets
- Non-blocking sockets are pretty much the same as regular sockets *(you'll use the same function calls, for example)*
- You turn a socket into a non-blocking socket by using `socket.setblocking(False)`
- The main difference between non-blocking and blocking sockets is that in the former, `send`, `recv` and `connect` can return without having done anything. You can check return codes and error codes to check if these methods have done anything but **that's a really bad idea**. A better way of checking is the methods have done anything is by using `select`:

```python
ready_to_read, ready_to_write, in_error = select.select(potential_readers, potential_writers, potential_errs, timeout)
```

- You pass `select` three lists: One for the sockets that you might want to try reading, the second one for sockets you might want to try writing to and finally those that you want to check for errors. 
- A socket can go into more than one list
- The `select` call is blocking, but you can give it a timeout
- `select` returns three lists. They contain the sockets that are actually readable, writable and in error

-  If a socket is in the output readable list, you can be pretty certain that a `recv` will return something. If a socket is in the output writable list, a `send` will work.
- If a server socket comes out in the readable list, an `accept` will work
- If a socket that has tried to connect to someone else comes out as writable, that means is connected to the other end

- `select` is also useful when handling blocking sockets. It's one way of determining wheter you'll block i.e. the socket returns readable when there's something in the buffer. This method doesn't tell you if a socket is done sending its data or if it's busy doing something else.