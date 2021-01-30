<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Lazy Developer
### Category: Web
### Difficulty: Medium
<hr>

#### Challenge Description
This challenge will need users to perform recon to find more information about a webapp, analyze the leaked source code and satisfy the requirements to achieve insecure deserialization and get code execution.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-1.png)

#### Solution
This challenge allows directory brute forcing, so the player will find the directory `/src/`:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-2.png)

After visiting this directory, the player will see leaked source code of the application:

```php
<?php
    //Source code for the API incase I forget lol

    //Query class
    class Query {
        private $hook;
        function __wakeup() {
            if (isset($this->hook)) eval($this->hook);
        }
    }

    if ($_SERVER['REQUEST_METHOD'] === 'POST') {

        //Epic security mechanism! Hackers give up!
        if ((sha1($_COOKIE['auth']) == "0e666")) {
            if ($_POST['query']) {
                $param = unserialize(base64_decode($_POST['query']));
                echo "Well, something is supposed to happen. But I'm too lazy to finish this part.\n";
            } else {
                echo "Invalid parameters.\n";
            }
        } else {
            echo "Invalid authorization cookie.\n";
        }

    } else {
        echo "<h1>This is an API...but I am too lazy to write documentation. Have a good day!</h1>\n";
    }
?>
```

Upon analyzing the leaked code, the player should understand that if they provide a cookie called `auth` with a specific value, they can execute any command in the `query` parameter. This comparison uses `==` on a value that begins with `0e` which indicates this is a type juggling attack.

Googling popular values for SHA1 that will create a hash starting with `0e` gives the input `10932435112`. Setting our cookie to this value results in the following message:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-3.png)

We see that this cookie is correct, and we have now proceeded into the deserialization part of the code. We see that a PHP class called `Query` is defined in the application, and has the magic method `__wakeup()` that can be abused by the player. The player can now craft a class that will abuse this method to craft a payload, like this:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-4.png)

Sending the output string to the application results in:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-5.png)

We see that we have successfully achieved code execution. Now we can get the flag (which was told to us to be `hidden.txt`).

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/lazy_developer-6.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
