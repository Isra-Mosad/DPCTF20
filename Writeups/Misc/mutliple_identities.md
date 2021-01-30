<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Multiple Identities
### Category: Misc
### Difficulty: Hard
<hr>

#### Challenge Description
In this challenge, the player is asked to generate a file that is multiple valid file formats at the same time, otherwise known as a file polygot.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/multiple_identities-1.png)

#### Solution
The challenge asks us to upload a file that is both a valid linux ELF executable that will output `HELLO_FROM_ELF` and a valid ZIP file containing a PNG called `qrcode.png` which is a QR code of the phrase `HELLO_FROM_PNG`. This type of file that is two file formats at the same time is called a **File Polygot**.

First, let's create the ELF file. Here is sample C code:
```cpp
#include <stdio.h>
int main()
{
    puts("HELLO_FROM_ELF");
    return 0;
}
```

Compile it and run shows that it gives us the output we desire:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/multiple_identities-2.png)

We can generate the QR code using any tool of our desire, I will generate one online:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/multiple_identities-3.png)

After putting the QR code in a zip file, we need to create our file polygot. For this writeup, I will be using the tool[Mitra](https://github.com/corkami/mitra). The command that will be used is `python3 mitra.py elf file.zip` where `file.zip` is our ZIP file containing `qrcode.png`. This will create a file called `file.elf.zip`.

Once the file polygot is generated, we base64 it and send it to the service.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/multiple_identities-4.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>