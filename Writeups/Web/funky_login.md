<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Funky Login
### Category: Web
### Difficulty: Beginner
<hr>

#### Challenge Description
In this challenge, the player is presented with a login page and is asked to find the credentials in order to find the flag.

#### Challenge Screenshot

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/funky_login-1.png)

```js
$("form").on("submit",function(a){const t=$("#username").val(),c=$("#password").val();t===atob("ZWxpdGVfYWRtaW4=")&&c===atob("c2VjdXJlX3VuaGFja2FibGVfcGFzc3dvcmQ=")||(alert("Incorrect credentials!"),a.preventDefault())});
```
#### Solution
Reversing the two base64 strings found in the JS reveals that the username is `elite_admin` and the password is `secure_unhackable_password`. Logging in with these credentials reveals the flag.

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
