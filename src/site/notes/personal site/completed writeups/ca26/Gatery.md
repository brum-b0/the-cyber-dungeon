---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/ca26/gatery/","tags":["web","CyberApocalypse-26","session-cookie"],"dg-note-properties":{"tags":["web","CyberApocalypse-26","session-cookie"]}}
---

Presented with an animated castle gate and a login page to get into the gate.
![](/img/user/img/Pasted%20image%2020260909133752.png)

the app checks what the session cookie is to determine whether or not you are inside or not.

```js
.post('/api/flag', ({ cookie, set }) => {
  const session = getSessionValue(cookie)

  if (!session) {
    set.status = 401
    return { ok: false, message: 'Login required' }
  }

  if (session !== 'inside') {
    set.status = 403
    return { ok: false, message: 'Enter the castle first' }
  }

  return { ok: true, flag }
})
```

```js
.post('/api/gate/enter', ({ cookie, set }) => {
  if (!getSessionValue(cookie)) {
    set.status = 401
    return { ok: false, message: 'Login required' }
  }

  if (!isAdminSession(cookie)) {
    set.status = 403
    return { ok: false, message: 'Gate authority required' }
  }

  setSessionCookie(cookie, 'inside')

  return { ok: true, insideGate: true }
})
```

```js
function getSessionValue(cookie: Record<string, { value?: unknown }>) {
  const session = cookie[sessionCookie]?.value

  return typeof session === 'string' && session ? session : null
}

function isAdminSession(cookie: Record<string, { value?: unknown }>) {
  const session = getSessionValue(cookie)

  return session === 'admin' || session === 'inside'
}
```

setting the session cookie to `inside` got me the flag:
![](/img/user/img/Pasted%20image%2020260909134159.png)

> [!NOTE] note
> the post body credentials with admin/admin aren't necessary, and were leftover from other repeater usage playing around on different endpoints. Only the session cookie matters.
