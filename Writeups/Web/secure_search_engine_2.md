<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Secure Search Engine 2
### Category: Web
### Difficulty: Hard
<hr>

#### Challenge Description
In this challenge, the player is presented with an updated version of the previous challenge with extra security in place. The goal is to achieve SQLi to dump the flag.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-1.png)

#### Solution
Just like the previous challenge, we can test for SQLi by inserting a `'`:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-2.png)

We can see that the website is still directly injectable and requires no obfuscation upfront. We also find the same HTML comment that allows us to view the SQL statements that are being sent to the database with `/debug?=1`. However, there is a new limitation. Only up to 10 characters are allowed in the *first* query. Let's test this by searching for 10 A's and 2 B's at the end (`AAAAAAAAAABB`):

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-3.png)

We can see that indeed all characters after the 10th are removed from the query. Additionally, the backend still strips all spaces. However, think about the sentence given to us, "for the first query". Does this suggest we can have multiple queries in the same request? Let's first confirm injection by attempting to inject an `OR 1=1--`:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-4.png)

Aha, we tested successful injection and even got some hints from the database, saying that we can infact search for multiple users by including multiple instances of the `q` HTTP parameter. This means that for us to search for posts left by both the admin user and the hacker user, we can send `q=admin&q=hacker`. Let's test this and see how the backend shapes the query:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-5.png)

It seems that the backend will append an `OR` in the middle of both queries. This is unsafe, and we can get rid of this `OR` to continue injection by ending the first query with `/*` and beginning the second query with `*/`, thus allowing us to control the query. Here is an example with the HTTP payload `q=admin/*&q=*/hacker&debug=1`:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-6.png)

We can see that no results are returned because the query becomes `SELECT * FROM app WHERE user='adminhacker';`. This means that we can use this technique to get around the 10 character limit for the first query and continue injecting our queries. This tecnique is known as **HTTP Parameter Pollution, or HPP**.

**Note: This is also possible to inject with SQLMap with the `--hpp` option, but I will do it manually.**

Let's directly run the query to fetch the flag from last challenge:

```sql
UNION/**/SELECT/**/flag,2,3/**/FROM/**/secret/**/--
```

But it will be sent like this:

```http
q='/*&q=*/UNION/**/SELECT/**/flag,2,3/**/FROM/**/secret/**/--&debug=1
```

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_2-7.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>