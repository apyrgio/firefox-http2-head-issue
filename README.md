# Firefox HTTP/2 HEAD Issue

This repo contains an easy way to reproduce a Firefox issue regarding HTTP/2
HEAD responses that contain very long headers.

## How to reproduce

Simply `git clone` this repo, `cd` to it and then serve it with an [HTTP/2
server] *(note that you will have to accept the server's self-signed TLS
certificate)*:

```shell
git clone git@github.com:apyrgio/firefox-http2-head-issue.git
cd firefox-http2-head-issue
sudo docker run --rm --name http2 -d -p 5000:5000 -v $PWD:/data surma/simplehttp2server -config config.json
```

Then, visit https://localhost:5000/pi100k, which returns the first 100,000
digits of Pi as an HTTP header. The page should load properly. However, if you
open Firefox's console and run the following command:

```javascript
fetch("/pi100k", {method: "HEAD"}).then(console.log)
```

you should see the following:

1. The request takes a long time to complete (~1 minute).
2. The request ultimately fails with the following error:

   > "TypeError: NetworkError when attempting to fetch resource."

3. The status code in Firefox's Network tab is 200.

## Cleanup

When you're done, you can simply stop the HTTP/2 server, and it will be deleted:

```shell
sudo docker stop http2
```

## Isn't this expected?

Well, kind of. It's true that each browser has an [internal upper limit] on the
header size they accept. However, when that limit is reached, browsers should
fail gracefully, which does not happen in this case. For instance, if you visit
https://localhost:5000/pi1m, which returns the first 1,000,000 digits of Pi, the
page immediately fails to load and the request does not show up as succeeded.
So, something is different in this case.

Finally, note that when HTTP/1.0 is used, Firefox handles headers with 100,000
characters gracefully.

[HTTP/2 server]: https://github.com/GoogleChromeLabs/simplehttp2server
[internal upper limit]: https://stackoverflow.com/a/3436155
