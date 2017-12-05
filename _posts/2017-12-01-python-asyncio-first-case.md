---
layout: post
title: Asyncio
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [python, asyncio]
---

Python中异步的使用
[原文](https://www.youtube.com/watch?v=WSq0S7UvI8E)

Python实现一个简单的echo服务器。

第一个版本

```Python
# echo_server.py

from socket import *


def echo_server(address):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(1)
    while True:
        client, addr = sock.accept()
        print('Connection from', addr)
        echo_handler(client)


def echo_handler(client):
    while True:
        data = client.recv(10000)
        if not data:
            break
        client.sendall(b'Got: ' + data)
    print('Connection closed')
    client.close()


if __name__ == '__main__':
    echo_server(('', 25000))

```

启动

```bash
python thread_echoserver.py
```

打开多个终端，执行 ```nc localhost 25000```, 随便输入几个单词

```bash
nc localhost 25000
hello
Got hello
world
Got world
```

但是仅有第一个终端会有反应，由于echo服务器仅能接受一个连接，所以其他的连接被阻塞了，需要关闭第一个终端，其他的终端才能看到相同的效果。

改进的方法之一，使用多线程

```Python
# thread_echoserver.py

from socket import *
import threading


def echo_server(address):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(1)
    while True:
        client, addr = sock.accept()
        print('Connection from', addr)
        t = threading.Thread(target=echo_handler,
                             args=(client,))
        t.start()


def echo_handler(client):
    while True:
        data = client.recv(10000)
        if not data:
            break
        client.sendall(b'Got: ' + data)
    print('Connection closed')
    client.close()


if __name__ == '__main__':
    echo_server(('', 25000))

```

启动

```bash
python thread_echoserver.py
```

打开多个终端，执行 ```nc localhost 25000```, 随便输入几个单词

```bash
# bash 1
nc localhost 25000
hello
Got hello
world
Got world
```

```bash
# bash n
nc localhost 25000
hello
Got hello
world
Got world
```

上面，用多线程的方法解决了多个客户端同时访问一个服务器端时连接阻塞的问题，(但是由于线程本身的开销，当连接数量很多的时候，阻塞或是变慢的问题便会存在)有没有更好的方法呢？

使用Asyncio

```Python
# async_echo_server.py

from socket import *
import asyncio


async def echo_server(address, loop):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(1)
    sock.setblocking(False)
    while True:
        client, addr = await loop.sock_accept(sock)
        print('Connection from', addr)
        loop.create_task(echo_handler(client, loop))


async def echo_handler(client, loop):
    while True:
        data = await loop.sock_recv(client, 10000)
        if not data:
            break
        await loop.sock_sendall(client, b'Got: ' + data)
    print('Connection closed')
    client.close()


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(echo_server(('', 25000), loop))

```

启动server

```bash
python3 async_echo_server.py
```

启动多个连接

```bash
# bash 1

nc localhost 25000
hello
Got hello
world
Got world
...

```

```bash
# bash n

nc localhost 25000
hello
Got hello
world
Got world
...

```

类比上面两种实现方式，代码仅有很小的变动，而且可以接受更多的连接。

> windows 下使用 nc [netcat-for-windows](https://joncraton.org/blog/46/netcat-for-windows/)