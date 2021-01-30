<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Digging For Gold
### Category: Web
### Difficulty: Easy
<hr>

#### Challenge Description
In this challenge, the player is presented with a website that allows them to browser through different haikus.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-1.png)

#### Solution
If the player views the source of this application, they will notice that there are hints on what to do next and a hint to possible source code reveal:

![step_1](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-2.png)

Adding the parameter reveals the application source code:

```php
<html>

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Digging For Gold</title>
</head>

<body style="width: 100%; height: 100%;">
    <div id="container">
        <?php
        ini_set('display_errors', 1);
        ini_set('display_startup_errors', 1);
        error_reporting(E_ALL);

        //https://github.com/defuse/php-encryption
        require_once("/app/defuse-crypto.phar");

        use Defuse\Crypto\Crypto;
        use Defuse\Crypto\Key;

        $GLOBALS['secret'] = Key::createNewRandomKey();

        //Let's encrypt the flag and send
        $GLOBALS['encrypted_flag'] = Crypto::encrypt("TEST_FLAG_HERE", $GLOBALS['secret']);

        if (isset($_COOKIE["sessID"])) {
            if (md5(json_decode(base64_decode($_COOKIE["sessID"]))->value) === '21232f297a57a5a743894a0e4a801fc3') {
                echo "<pre>";
                try {
                    echo (eval($_SERVER['HTTP_USER_AGENT']));
                } catch (ParseError $e) {
                    echo "$e\n";
                } catch (TypeError $e) {
                    echo "$e\n";
                }
                echo "</pre>";
            } else {
                echo "<!-- md5(json_decode(base64_decode(sessID)))->value must be equal to 21232f297a57a5a743894a0e4a801fc3 -->";
            }
        } else {
            echo "<!-- Where is my sessID cookie? -->";
        }

        if (isset($_GET['src'])) {
            echo highlight_file('./index.php');
            die();
        }

        echo "<!-- /?src=1 --><pre style='font-size: 32px; white-space: pre-wrap; word-wrap: break-word;'>" . $GLOBALS['encrypted_flag'] . "</pre>";
        ?>
    </div>
</body>

</html>
```

Initial observation explains that the hex we see in the webapp is an encrypted version of the flag, with a random key being generated each time the player loads the web app.

Upon analyzing the leaked source code, the player will understand that they need to satisfy this if statement in order to achieve code execution. The if statement requires a base64 JSON object with the key `value` to contain a value such that when run through MD5 will result in `21232f297a57a5a743894a0e4a801fc3`. This encoded payload should be stored in a cookie called `sessID`.

If we google the MD5 hash, we see that the cleartext version of it is `admin`. Knowing this, let’s create the payload:

![step_3](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-3.png)

We see that we no longer see any warning messages, but instead an error message stating that an incorrect syntax happened in an eval statement. Meaning we have successfully achieved code execution. Let’s try to edit our **User Agent** to have a proper payload:

![step_4](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-4.png)

We see that most code execution functions have been disabled. But the goal here is to leak the flag, which is already a variable inside the code. Let’s read the documentation for the crypto library used in the webapp:

![step_5](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-5.png)

Using this knowledge, we can call this function to decrypt the flag and print it in plaintext using the eval we have.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/digging_for_gold-6.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
