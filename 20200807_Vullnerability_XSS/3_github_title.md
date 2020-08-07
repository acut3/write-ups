# XSS #3: stored XSS using title from Github

## PoC

In this example the attacker is controlling the https://github.com/acut3/vulln/ repository.
1. Locally create a file named (for example) `title.html` with the following content:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="utf-8">
   <title><script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">alert(document.domain)</script></title>
   </head>
   <body></body>
   </html>
   ```
2. Create a new release on your repository, attach the `title.html` file to this release and publish the release. This makes the file downloadable from the https://github.com/acut3/vulln/releases/download/v0.0.2/title.html URL (assuming you used v0.0.2 as the tag for your release)
3. Enter `acut3/vulln/releases/download/v0.0.2/title.html` as your Github profile on https://lab.takeover.host/apply
4. Click "My Profile" to view the new profile and trigger the XSS


## Discussion

When a user gives his Github profile on the app, the server fetches the `https://github.com/<profile_name>` URL and extracts whatever is between the `<title>` tags on that page. The extracted string is inserted into the DOM as an anchor. This is done in `api.js`:

```js
$(".github_profile").html("<a title='"
                          + response.ch_github_title
                          + "' class='github' href='https://github.com/"+encodeURI(response.ch_github)
                          + "' target='_blank'>"
                          + response.ch_github_title
                          + "</a>");
```

We can see that jQuery's `.html()` method is used, which means that not only the data is inserted as HTML, but it's also evaluated (something that wouldn't happen if we were simply using `innerHTML`). If only we could control the title of our Github page, we'd have a stored XSS.

I immediately noticed that we could use complete paths as our Github handle (e.g. `acut3/vulln`), and the app would grab the title of that page in my repository. However when specifying something like `https://something.com/evil.html` as my handle the server would simply try to grab `https://github.com/https://something.com/evil.html`. We are truly stuck to pages on `github.com`, and none of them let you use `<` or `>` characters in their title. It's only after a full day of searching for ways of hosting files on `github.com` that I learned about releases and realized they let you do just that.
