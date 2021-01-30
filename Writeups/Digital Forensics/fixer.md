<img src="https://github.com/CTFae/media/blob/main/brand/YT-banner.jpg" />

### Challenge: Fixer
### Category: Digital Forensics
### Difficulty: Easy
<hr>

The description asks where is my data, and you are given one file called commented.jpg, image file.

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-1.png" />

use `file` command to make sure it is jpg

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-2.png" />

You can realize the file name said that the jpg contains a comment but it’s not readable in this command.

Let's try to open the image

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-3.png" />

It can't be opened, there is an invalid or lost jpg marker.

As the file command said this file has a comment but it didn't show valid information about it, let's try to use a tool that can show us the comment and all information about this file, such as `exiftool`.  

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-4.png" />

The comment is not readable here too, but it gives a hint. The comment marker may be replaced with another marker header, because all the information looks valid, no warnings or wrong information, the only thing that looks suspicious is the comment. 

Let's look where is the comment actually replaced in the hex view of the file. First you need to know the marker responsible for commenting.

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-5.png" />

Search for the hex value `0xfffE`

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-6.png" />

It is found only one time. Now try to find the next marker header to know which marker you should change the comment to.

Note: Any jpg marker start with the byte `0xFF` followed by another byte, except `0x00`. You can use the jpg markers list to help you.

After searching, you will find the next marker is `0xFFD9`, the last 2 bytes of the image also among markers represent `ENDOFIMAGE`.

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-7.png" />

So the comment is replaced on the place of the marker before the end marker. To find what it should be, you have to look at the jpg structure.

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-8.png" />

You can notice the section before the end is the data section and it has no marker at the beginning, but it starts after scan section which is 2 bytes for the header and 12 bytes for the body. The Scan section starts with the marker `0xFFDA`. 

So, let's modify `0xFFFE` to `0xFFDA` and save the file. 

Now, you get the fixed image with the flag

<img src="https://github.com/CTFae/media/blob/main/writeups/fixer-9.png" />


<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>






