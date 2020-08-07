# XSS #1: reflected DOM-based XSS using /logout?next=javascript:

## PoC

Visit the following URL:
https://lab.takeover.host/logout?next=javascript://%250aalert(document.domain)

## Discussion

At the bottom of the `/logout` page there is a script that looks at the next query parameter and, if it matches `/^(\w)*(:\/\/)(.)*(\.)(.)*$/g`, redirects to this location by setting `window.location.href`.

Setting `window.location.href` to a `javascript:` URL would get us an XSS since there is no CSP protection on `/logout`. The only challenge is to bypass the regexp. The URL needs to start with `javascript://`, but obviously the double slash starts a comment that extends to the end of the line. Luckily, double-URL-encoding a newline character (newline -> `%0a` -> `%250a`, thanks 0xatul) does the trick and ends the comment, allowing us to execute anything that follows. The dot in `document.domain` matches the rest of the regexp, and if it didn't we could add one anywhere.
