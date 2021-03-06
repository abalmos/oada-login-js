oadaIDClient
============

JavaScript client library for OADA identity.
Can be used both in NodeJS and in the browser.

Getting Started
---------------

### Installation ###
The library is not yet published with `npm`,
but it can be installed from the GitHub repo.
```sh
$ npm install OADA/oada-id-client-js
```

### Browser Code Generation ###
Grunt is needed to create the browser version of the code.
If you do not already have it, Grunt can be installed like so:
```sh
$ npm install -g grunt-cli
```

The code to use in the browser can be generated with the following command:
```sh
$ grunt build
```
This will create the file `dist/main.browserify.min.js`,
which is a UMD bundle of the browser version of the library functions.
It will work with most browser module systems,
or expose a global variable `oadaIdClient` if no module system is present.

Examples
--------
* [On Server Example][]
* [In Browser Example][]

### Running the examples ###
Run the following command to start an express server on `localhost:3000`
which server both examples:
```sh
$ grunt demo
```

Connect Style "Middleware" Wrapper Usage
----------------------------------------
Version of the library functions which wrap the core functionality
for use as connect style "middleware".
This can be used in a NodeJS server using a compatible web development
framework, such as express.

For a working example of using this wrapper, see the [on server example][].


### getIDToken(domain, options) ###
Middleware for generating an ID token request against an OADA identity
provider.

#### Parameters ####
`domain` string of domain with which to log in the user.
The value passed to the function can be overriden by a query or form
parameter with a name of `domain`.

`options` object containings at least the following properties:
* [`client_id`][] string containing your client's ID
  (received when the client was registered with an OADA developer discovery
  provider).
* [`redirect_uri`][] string containing an asbolute URI
  to which the user's browser will be redirected when the token request
  finishes.
* `privateKey`
  * `pem` string or buffer containing your client's PEM encoded private RSA
    key.
  * [`kid`][] string containing the key ID paramter,
    for finding the corresponding public key where your client is registered.

[Optional OpenID Connect parameters][oidparams] placed in options as
string properties will be used (e.g. `display`, `prompt`, `login_hint`).

#### Usage Example ####
```javascript
var options = {
    client_id: 'your_clients_client_id',
    redirect_uri: 'https://some/url/that/calls/handleRedirect()',
    privateKey: {
        pem: fs.readFileSync('/path/to/key.pem'),
        kid: 'key_id_corresponding_to_pem',
    },
};

app.use('/getIdToken',
    oadaIdClient.getIDToken('some.oada-identity-provider.com', options));
```

### getAccessToken(domain, options) ###
Middleware for generating an access token request against an OADA compliant
API.

#### Parameters ####
`domain` string of domain from which to get an OADA API access token.
The value passed to the function can be overriden by a query or form
parameter with a name of `domain`.

`options` object containings at least the following properties:
* [`client_id`][] string containing your client's ID
  (received when the client was registered with an OADA developer discovery
  provider).
* [`redirect_uri`][] string containing an asbolute URI
  to which the user's browser will be redirected when the token request
  finishes.
* [`scope`][] space separated string of OAuth scopes for the request access
  token to have.
* `privateKey`
  * `pem` string or buffer containing your client's PEM encoded private RSA
    key.
  * [`kid`][] string containing the key ID paramter,
    for finding the corresponding public key where your client is registered.

[Optional OpenID Connect parameters][oidparams] placed in options as
string properties will be used (e.g. `display`, `prompt`, `login_hint`).

#### Usage Example ####
```javascript
var options = {
    client_id: 'your_clients_client_id',
    redirect_uri: 'https://some/url/that/calls/handleRedirect()',
    privateKey: {
        pem: fs.readFileSync('/path/to/key.pem'),
        kid: 'key_id_corresponding_to_pem',
    },
    scope: 'some.oada.defined.scope',
};

app.use('/getAccessToken',
    oadaIdClient.getAccessToken('some.oada-cloud-provider.com', options));
```

### handleRedirect() ###
Middleware for handling redirects from `getIDToken` or `getAccessToken`
middlewares.
In most case you will apply this middleware in two locations,
one to receieve `getIDToken` redirects and
another to recieve `getAccessToken` redirects.

#### Usage Example ####
```javascript
// Handle ID token redirects
app.use('/url/referenced/by/getIDToken/redirect_uri',
    oadaIdClient.handleRedirect());
app.use('/url/referenced/by/getIDToken/redirect_uri',
    function (req, res, next) {
        // ID token is in req.token
        console.dir(req.token);
    });

// Handle access token redirects
app.use('/url/referenced/by/getAccessToken/redirect_uri',
    oadaIdClient.handleRedirect());
app.use('/url/referenced/by/getAccessToken/redirect_uri',
    function (req, res, next) {
        // Access token is in req.token
        console.dir(req.token);
    });
```

Browser Wrapper Usage
---------------------
Version of the library functions which wrap the core functionality
for easy use in the browser.

For a working example of using this wrapper, see the [in browser example][].

### getIDToken(domain, options, callback) ###
Asynchronous funtion for generating an ID token request against
an OADA identity provider.

#### Parameters ####
`domain` string of domain with which to log in the user.

`options` object containings at least the following properties:
* [`client_id`][] string containing your client's ID
  (received when the client was registered with an OADA developer discovery
  provider).
* [`redirect_uri`][] string containing an asbolute URI
  to which the user's browser will be redirected when the token request
  finishes.

[Optional OpenID Connect parameters][oidparams] placed in options as
string properties will be used (e.g. `display`, `prompt`, `login_hint`).

`callback` function of the form `function(err, idToken)`.

#### Usage Example ####
```javascript
var options = {
    client_id: 'your_clients_client_id',
    redirect_uri: 'https://some/url/that/calls/handleRedirect()',
};

var domain; // Set domain based on text box, dropdown, etc.

oadaIdClient.getIDToken(domain, options, function(err, idToken) {
    if (err) { return console.dir(err); } // Soemthing went wrong

    console.dir(idToken);
});
```

### getAccessToken(domain, options, callback) ###
Asynchronous funtion for generating an access token request against an
OADA compliant API.

#### Parameters ####
`domain` string of domain from which to get an OADA API access token.
The value passed to the function can be overriden by a query or form
parameter with a name of `domain`.

`options` object containings at least the following properties:
* [`client_id`][] string containing your client's ID
  (received when the client was registered with an OADA developer discovery
  provider).
* [`redirect_uri`][] string containing an asbolute URI
  to which the user's browser will be redirected when the token request
  finishes.
* [`scope`][] space separated string of OAuth scopes for the request access
  token to have.

[Optional OpenID Connect parameters][oidparams] placed in options as
string properties will be used (e.g. `display`, `prompt`, `login_hint`).

`callback` function of the form `function(err, accessToken)`.

#### Usage Example ####
```javascript
var options = {
    client_id: 'your_clients_client_id',
    redirect_uri: 'https://some/url/that/calls/handleRedirect()',
    scope: 'some.oada.defined.scope',
};

var domain; // Set domain based on text box, dropdown, etc.

oadaIdClient.getAccessToken(domain, options, function(err, accessToken) {
    if (err) { return console.dir(err); } // Soemthing went wrong

    console.dir(accessToken);
});
```

### handleRedirect() ###
Function for handling redirects generated by
`getIDToken` or `getAccessToken` function.
Simply needs to be called by the page served from the URL corresponding to
[`redirect_uri`][].

#### Usage Example ####
```html
<!-- Page served at redirect_uri for getIDToken and/or getAccessToken -->
<html>
<head>
    <script src="path/to/library/browser/code.js"></script>
    <script>oadaIdClient.handleRedirect();</script>
</head>
</html>
```

Base Libary Usage
------------------
Not yet documented.

References
----------
[on server example]: examples/server-client/
[in browser example]: examples/browser-client/
1. [OpenID Connect Core 1.0](http://openid.net/specs/openid-connect-core-1_0.html)
[oidparams]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest "OpenID Connect Core 1.0 Section 3.1.2.1"
2. [OAuth 2.0 Authorization Framework](http://tools.ietf.org/html/rfc6749 "RFC6749")
[`client_id`]: http://tools.ietf.org/html/rfc6749#section-2.2 "RFC6794 Section 2.2"
[`redirect_uri`]: http://tools.ietf.org/html/rfc6749#section-3.1.2 "RFC6794 Section 3.1.2"
[`scope`]: http://tools.ietf.org/html/rfc6749#section-3.3 "RFC6794 Section 3.3"
3. [JSON Web Key (JWK) Draft 31](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-31)
[`kid`]: https://tools.ietf.org/html/draft-ietf-jose-json-web-key-31#section-4.5 "JWK Section 4.5"
