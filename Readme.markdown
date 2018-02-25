# Kvetch.js

I really like using the fetch API packaged in evergreen browsers, but I was getting annoyed with having to set the credentials, redirect, and method options all the time, plus it always takes a bit of code to format the querystring correctly, plus it's annoying to set headers and stringify JSON objects that I want to PUT to my server. So I wrote Kvetch. First argument is URL, which gets passed directly to fetch. Second argument is an optional query object. This can have as many key value pairs as you want, it just gets URIComponent encoded, joined with &s and =s, and appended to the URL after a '?' (so don't put the ? in yourself). Third argument is the Body a.k.a. Request Payload. It can be an object, a string, an ArrayBuffer (ie Binary data) or FormData.

### `kvetch.get/post/put/delete/options(URL::string[, QueryObject::Object, Body::*)`
You can leave the QueryObject and the Body blank if you don't need them.

You can pass a falsey argument (`null`, `undefined`, etc) as a QueryObject if you only need a Body.

If you give an Object as a Body, it will be JSON stringified and sent with an `application/json` ContentType. If you send FormData (including files), the body is handed directly to fetch and it figures out what to do. If you pass a string, it will be sent untouched with a ContentType of `text/plain`. ArrayBuffers get sent with `application/octet-stream` but I haven't actually tested this and don't know if it's appropriate.

```html
<script src="wherever you cloned this file/kvetch.js">
<script>
    kvetch.get('/', {anything: 'you want'})
    // becomes '/?anything=you%20want'

    kvetch.post('/somedata', null, {
        "key-1" : "some data you want to send as a JSON body",
        "key-b" : "the headers and cookies and everything else are handled for you"
    }).then(res => res.json()).then(data => /* any call to kvetch returns the fetch it invoked */)

    var someSignUpSheet = new FormData()
    someSignUpSheet.append('firstName','your')
    someSignUpSheet.append('middleName','latest')
    someSignUpSheet.append('lastName', 'identity')

    kvetch.put('/signUp', {metadata: 'extra info'}, someSignUpSheet)
          .then(()=> alert("success!"))
</script>
```

Offered as Public Domain, no warranty. Pull Requests Appreciated.
