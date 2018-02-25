# Kvetch.js

I really like using the fetch API packaged in evergreen browsers, but I was getting annoyed with having to set the credentials, redirect, and method options all the time, plus it always takes a bit of code to format the querystring correctly, plus it's annoying to set headers and stringify JSON objects that I want to PUT to my server. So I wrote Kvetch. First argument is URL, which gets passed directly to fetch. Second argument is an optional query object. This can have as many key value pairs as you want, it just gets URIComponent encoded, joined with &s and =s, and appended to the URL after a '?' (so don't put the ? in yourself). Third argument is the Body a.k.a. Request Payload. It can be an object, a string, an ArrayBuffer (ie Binary data) or FormData.

A lot of people use axios, request, or other libraries on npm, but I didn't want to add extra features and a bunch of dependencies, I just wanted to prevent repeating myself using the native API.

If you need it to work on browsers without fetch, just bring in some fetch polyfill can define that first, kvetch will use window.fetch just fine.

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

Uses Proxy to pass on the http method to the options object, whole thing weights in at 2KB:
```js
/* - Public Domain software by Colten Jackson
 * - make a global method to prefer over 'fetch' api 
 * - pass an object to convert key:value pairs to a properly encoded querystring (k:v, hence, kvetch)
 * - use a Proxy with a getter so you can call 'kvetch.get()','kvetch.put()','kvetch.delete()' and so on with a single function
 * * */
(function(){
	let kv2query = kv => Object.keys(kv || {}).map(key => {
			var value = kv[key]
			return encodeURIComponent(key) + '=' + encodeURIComponent(value)
		}).join('&')

	let bodyBuilder = body => {
		let bodyType = body && body.constructor
		/* if bodyType if falsey, assign an empty object, no change */
		if(!bodyType) return {}
		/* if bodyType is formdata, hand the body back and let fetch assign content header automatically */
		else if(bodyType == FormData) return {body}
		/* otherwise, stringify objects and set content type to go with it */
		else return {
			body: bodyType == Object ? JSON.stringify(body) : body,
			headers: {
				'Content-Type': bodyType == ArrayBuffer ? 'application/octet-stream' :
								bodyType == Object ? 'application/json' : 'text/plain'
			}
		}
	}

	window.kvetch = new Proxy({}, {
		get: (target, name) => {
			return (url, queryObject = {}, optionalBody) => {
				/* can't do anything with an url */
				if(!url || url.constructor != String) throw new Error("first argument URL must be a string.")
				/* it's okay if queryObject is left blank, but if its truthy it better be an object */
				if(queryObject && queryObject.constructor != Object) throw new Error("second argument QueryObject must be an object or nothing at all.")
				/* optionalBody will have to switch between content types for string, object -> JSON, or raw byte array */

				options = Object.assign({
					method: name.toUpperCase(),
					credentials: 'same-origin',
					redirect: 'error'
				}, bodyBuilder(optionalBody))

				return fetch(url + '?' + kv2query(queryObject), options)                
			}
		}
	})
})()
```
You can even copy paste that into your console to take it for a spin.

