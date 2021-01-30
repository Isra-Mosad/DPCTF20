<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: I <3 Open Source
### Category: Web
### Difficulty: Hard
<hr>

#### Challenge Description
This challenge will need users to download the source code of the web application through an exposed git repo, crack some hardcoded credentials, and combine an SSRF vulnerability with LFI to upload a webshell to achieve code execution.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-1.png)

#### Solution
The challenge clearly indicates that this is an open source app using the "git" protocol. We can check if there is a `/.git/` folder exposed, which means we can download and retrieve the entire source code of the application.

For that, the player can use any tool such as `git-dumper`:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-2.png)

We can verify that the files were downloaded:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-3.png)

Upon reading all the files, the user will notice that `auth.php` contains some hard-coded credentials:

```php
<?php
    //Check that the auth matches
    if (isset($_GET['username']) && isset($_GET['password'])) {
            if (!(hash('sha256', $_GET['username']) === 'c6c6dc4efdd314700252330e1e36db2ef1b1cc2d703b884168c541963336a0c8') || !(hash('sha256', $_GET['password']) === '1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc946683d7b336b63032')) {
                    echo "Invalid credentials!";
                    die();
            }
    } else {
            echo "Missing credentials!";
            die();
    }
?>
```

Using the internet to crack these hashes shows that the username is `webadmin` and the password is `letmein`. Knowing that there are the credentials, we can know read the other available API endpoints that we have.

Upon inspection of the file `readResponse.php`, the player should note that it looks like this is an SSRF vulnerability as any input that the player passes is run through cURL. Another file (`fetchUserInfo.php`) reveals MySQL credentials.

```php
<!-- readResponse.php -->
<?php
    require_once('auth.php');
    if(isset($_GET['url'])) {
        $ch = curl_init();
                curl_setopt($ch, CURLOPT_URL, $_GET['url']);
                curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
                curl_setopt($ch, CURLOPT_HEADER, 0);
                $result = curl_exec($ch);
        curl_close($ch);
                echo '<pre>' . htmlspecialchars($result) . '</pre>';
    }
?>
```

```php
<!-- fetchUserInfo.php -->
<?php
    $conn = new mysqli('127.0.0.1', 'webapp', '');
    //I really hope this error doesn't fire
    if ($conn->connect_error) {
            die("Connection failed: " . $conn->connect_error);
    }

    //TODO: Add functionality here! I hope my boss pays me here...
?>
```

Additionally, inspecting `fetchFile.php` shows a basic LFI vulnerability:

```php
<?php
    require('auth.php');
    if (isset($_GET['file'])) {
            echo '<pre>';
            include($_GET['file']);
            echo '</pre>';
    }
?>
```

The MySQL creds have an empty password, which fulfills the requirement for an SSRF attack on the MySQL database using the `gopher://` protocol. This vulnerability abuses the fact that `cURL` is run on any URL that is given, which means we can make `cURL` run SQL statements on the database using the `gopher://` protocol. Many tools exist for this, but for this writeup I will be using `SSRFMap` (you will need to save a sample request to a file):

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-6.png)

What this payload will do is create a webshell in `/tmp/shell.php` which we can then include using the LFI vulnerability in `fetchFile.php`. Let's send this payload to `readResponse.php`:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-7.png)

You can now see we successfully created the webshell and can now run commands by including the shell using `fetchFile.php`.

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/i_heart_open_source-8.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>