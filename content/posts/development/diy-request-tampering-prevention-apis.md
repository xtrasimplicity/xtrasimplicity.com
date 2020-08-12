---
title: "DIY Request Tampering Prevention in APIs"
date: 2020-08-08T13:14:04+10:00
draft: false
tags: ['api', 'request tampering', 'security']
categories: ['Development', 'APIs']
---

In API development, end-to-end encryption with TLS is a very important, fundamental protection mechanism used to prevent the request/response from being tampered with (and even inspected), in-transit.

Sometimes, however, you might also want to add API-level validation checks to ensure that the request has not been tampered with. This is useful in scenarios where transport-layer security is not used for whatever reason (and the request payload does not contain sensitive content), but in-transit manipulation of the request payload could result in unwanted changes - such as the deletion of a different record than what was intended.

This can be easily mitigated by signing the request payload, API endpoint, and HTTP method used for the request, on the API client side, and verifying this on the API server.

As an example, consider the following HTTP request which is used to delete a user with an ID of `123`.
```
POST /api/v1/users/delete
Content-Type: application/json

{
	"id": "123"
}
```

If an attacker is able to manipulate this request in-transit, they could change the `id` property of the JSON payload to a different value, resulting in a different user being deleted. e.g.
```
POST /api/v1/users/delete
Content-Type: application/json

{
	"id": "456"
}
```

To prevent this, we can use pre-shared keys to sign the request (and even response) payloads. This will require a change to both the API server and the client.

### Step 1: Generate pre-shared keys
To start with, we'll need to generate a unique `public key` and `secret key` for each API client. It is best to do this using a secure, random generator, such as Ruby's `SecureRandom`.

```ruby
public_key = SecureRandom.hex(16)
secret_key = SecureRandom.hex(64)
```

### Step 2: Ensure that the API client signs the request
Now, we'll need to make sure that the API client signs the request with the pre-shared keys, and sends the `public` key alongside the request so that the server knows how to verify the request's validity.

One way to do this is to generate a `Hash-based Message Authentication Code` (HMAC), using the pre-shared keys and request payload.
e.g.

```ruby
require 'openssl'

public_key = 'myPublicKey'
secret_key = 'MyUltraSecretKey'

api_entrypoint = '/api/v1/users/delete'
api_method = 'POST'
request_payload = {
  id: '123'
}.to_json

digest = OpenSSL::Digest.new('sha256')
data = [public_key, api_entrypoint, api_method, request_payload].join('|')

signature = OpenSSL::HMAC.hexdigest(digest, secret_key, data)
```

And then send this signature to the API server, along with the request. e.g.
```
POST /api/v1/users/delete
Content-Type: application/json
X-Client-Id: 'myPublicKey'
X-Request-Signature: <GENERATED_SIGNATURE>

{
	"id": "123"
}
```

### Step 3: Verify that the request signature is valid
On the API server, we now need to ensure that the request was valid before we do anything with the request.

To do this, we can parse the `X-Client-Id` header to determine the API client's public key, and then generate a signature in the same way that the client should have.
e.g.

```ruby
require 'openssl'

client_public_key = request.headers['X-Client-Id']
client_secret_key = ... # Check the database for the secret key associated with the client's public key we retrieved from the request headers.

digest = OpenSSL::Digest.new('sha256')
data = [client_public_key, request.location, request.method, request.body].join('|')

expected_signature = OpenSSL::HMAC.hexdigest(digest, client_secret_key, data)
actual_signature = request.headers['X-Request-Signature']

if actual_signature == expected_signature
   # Request hasn't been tampered with
else
   # Request has been tampered with
end
```

Once we've done this, we can simply compare the two signatures - the one we expected to have been generated (given the request method, API entrypoint, body and API client secret), and the one supplied by the API client. If these signatures do not match, or if the public key provided is invalid (or missing completely), we can assume that:

1. The request has been manipulated in-transit, or
2. The signature has not been generated in the same way on both the client and server, or
3. There is a configuration issue on either the API client or server side.

If the two signatures do match, we can assume that the request has not been tampered with and that we can safely continue processing the request.






