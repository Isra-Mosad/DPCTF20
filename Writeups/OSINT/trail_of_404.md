<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Trail of 404s
### Category: OSINT
### Difficulty: Easy
<hr>

#### Challenge Description
This challenge will need users to identify to investigate a Twitter account for clues and find the flag.

#### Solution
The challenge first tells us to look at a twitter account, [@dead_packets](https://twitter.com/dead_packets). Running this account through the Wayback Machine reveals a deleted tweet:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-1.png)

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-2.png)

Accessing the pastebin link shows that it has been deleted. Going through the wayback machine we can recover the contents of the paste:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-3.png)

We get information about a user `ze_epic_haxx0r` who seems to have an account on the site with an alien mascot. The site that should come to mind is [Reddit](https://reddit.com), and their mascot is shown below:

![chal](https://qph.fs.quoracdn.net/main-qimg-7d2f5552ca8fdefdd548c26eba761f78.webp)

Going to `ze_epic_haxx0r’s` [reddit account](https://reddit.com/u/ze_epic_haxx0r) shows one post:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-4.png)

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-5.png)

This tells us to head to `DeadPackets’s` [Github page](https://github.com/DeadPackets), but there used to be something there that has been removed. Using wayback, we find the flag.

#### Flag Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/trail_of_404-6.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
