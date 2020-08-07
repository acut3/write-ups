# XSS #2: reflected DOM-based XSS using /profile/?link=

## PoC

As an authenticated user, visit the following link:

https://lab.takeover.host/profile/?link=user_%3Ciframe%20srcdoc=%22%3Cscript%20nonce=%27EDNnf03nceIOfn39fn3e9h3sdfa%27%3Ealert(document.domain)%3C/script%3E%22%3E%3C/iframe%3E

## Discussion

The javascript in the `/profile` page looks at the `link` query parameter. If it starts with `user_` but the rest of the string doesn't match the id of the current user, an alert box is displayed using `Swal.fire()`. The message is inserted into the DOM as HTML and it includes the URL-supplied string, which allows us to inject arbitrary elements into the DOM.

Unfortunately there are two obstacles that we need to bypass:

1. While `<script>` elements can be inserted into the DOM, they are not executed. This is normal behavior when elements are inserted with `innerHTML()`, which I guess is what `Swal.fire()` uses.
1. The CSP on `/profile` uses a nonce for `script-src`. This means that inline javascript is blocked by the CSP, even though it would normally be executed when the element is inserted into the DOM. So, no `<img src="" onerror="alert(document.domain)">` or `javascript:` URL. It also means we either have to specify the correct nonce in any `<script>` that we inject, or we need to inject inside an existing `<script>`.

Fortunately for us, the nonce used by the CSP is fixed. It's always `EDNnf03nceIOfn39fn3e9h3sdfa`. So we can just specify that in any `<script>` that we inject. The only effect this nonce has is that it blocks inline javascript.

Now how do we get our `<script>` to execute? We can inject an `<iframe>` that will contain our malicious payload. Using the `srcdoc` attribute allows us to preserve the same origin as the parent frame, something that a `data:text/html` URL wouldn't allow.
