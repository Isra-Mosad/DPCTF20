<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Haiku Lover
### Category: Web
### Difficulty: Beginner
<hr>

#### Challenge Description
In this challenge, the player is presented with a website that allows them to browser through different haikus.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/haiku_lover-1.png)

#### Solution
The challenge description tells us that the flag is in `/etc/flag.txt`. Additionally, clicking on any of the links reveals an HTTP parameter `?haiku=` which looks to be vulnerable to LFI.

Simply setting the path to `../../../etc/flag.txt` reveals the flag.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/haiku_lover-2.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
