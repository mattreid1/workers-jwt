# workers-jwt

[`@mattreid.dev/workers-jwt`](https://www.npmjs.com/package/@mattreid.dev/workers-jwt) helps you
generate a `JWT` on Cloudflare Workers with the WebCrypto API. Helper function for GCP Service Accounts included.

Fork of [@sagi.io/workers-jwt](https://github.com/sagi/workers-jwt) This fork does not work with Node.js but does with Cloudflare Workers.

## Installation

~~~
$ npm i @mattreid.dev/workers-jwt
~~~

## API

We currently expose two methods: `getToken` for general purpose `JWT` generation
and `getTokenFromGCPServiceAccount` for `JWT` generation using a `GCP` service account.

### **`getToken({ ... })`**

Function definition:

```js
const getToken = async ({
  privateKeyPEM,
  payload,
  alg = 'RS256',
  headerAdditions = {},
}) => { ... }
```

Where:

  - **`privateKeyPEM`** is the private key `string` in `PEM` format.
  - **`payload`** is the `JSON` payload to be signed, i.e. the `{ aud, iat, exp, iss, sub, scope, ... }`.
  - **`alg`** is the signing algorithm as defined in [`RFC7518`](https://tools.ietf.org/html/rfc7518#section-3.1), currently only `RS256` and `ES256` are supported.
  - **`headerAdditions`** is an object with keys and string values to be added to the header of the `JWT`.

### **`getTokenFromGCPServiceAccount({ ... })`**

Function definition:

```js
const getTokenFromGCPServiceAccount = async ({
  serviceAccountJSON,
  aud,
  alg = 'RS256',
  expiredAfter = 3600,
  headerAdditions = {},
  payloadAdditions = {}
}) => { ... }
```

Where:

  - **`serviceAccountJSON`** is the service account `JSON` object .
  - **`aud`** is the audience field in the `JWT`'s payload. e.g. `https://www.googleapis.com/oauth2/v4/token`'.
  - **`expiredAfter`** - the duration of the token's validity. Defaults to 1 hour - 3600 seconds.
  - **`payloadAdditions`** is an object with keys and string values to be added to the payload of the `JWT`. Example - `{ scope: 'https://www.googleapis.com/auth/chat.bot' }`.
  - **`alg`**, **`headerAdditions`** are defined as above.


## Example

Suppose you'd like to use `Firestore`'s REST API. The first step is to generate
a service account with the "Cloud Datastore User" role. Please download the
service account and store its contents in the `SERVICE_ACCOUNT_JSON_STR` environment
variable.

The `aud` is defined by GCP's [service definitions](https://github.com/googleapis/googleapis/tree/master/google)
and is simply the following concatenated string: `'https://' + SERVICE_NAME + '/' + API__NAME`.
More info [here](https://developers.google.com/identity/protocols/OAuth2ServiceAccount#jwt-auth).

For `Firestore` the `aud` is `https://firestore.googleapis.com/google.firestore.v1.Firestore`.

## Cloudflare Workers Usage

Cloudflare Workers expose the `crypto` global for the `Web Crypto API`.

~~~js
const { getTokenFromGCPServiceAccount } = require('@mattreid.dev/workers-jwt')

const serviceAccountJSON = await ENVIRONMENT.get('SERVICE_ACCOUNT_JSON','json')
const aud = `https://firestore.googleapis.com/google.firestore.v1.Firestore`

const token = await getTokenFromGCPServiceAccount({ serviceAccountJSON, aud} )

const headers = { Authorization: `Bearer ${token}` }

const projectId = 'example-project'
const collection = 'exampleCol'
const document = 'exampleDoc'

const docUrl =
  `https://firestore.googleapis.com/v1/projects/${projectId}/databases/(default)/documents`
  + `/${collection}/${document}`

const response = await fetch(docUrl, { headers })

const documentObj =  await response.json()
~~~