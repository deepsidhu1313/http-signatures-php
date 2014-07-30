# HTTP Signatures

PHP implementation of [HTTP Signatures][draft03] draft specification;
cryptographically sign and verify HTTP requests and responses.

See also:

* https://github.com/99designs/http-signatures-ruby


## Usage

Configure a context with your algorithm, keys, headers to sign.
This is best placed in an application startup file.

```rb
use HttpSignatures\Context;

$context = new Context(array(
  'keys' => array('examplekey' => 'secret-key-here'),
  'algorithm' => 'hmac-sha256',
  'headers' => array('(request-target)', 'Date', 'Accept'),
));
```

If there's only one key in the `keys` hash, that will be used for signing.
Otherwise, specify one via `'signingKeyId' => 'examplekey'`.

### Messages

A message is an HTTP request or response. A subset of the interface of
[Symfony\Component\HttpFoundation\Request] is expected; the ability to read
headers via `$message->headers->get($name)` and set them via
`$message->headers->set($name, $value)`, and for signing requests, methods to
read the path, query string and request method.

```rb
use Symfony\Component\HttpFoundation\Request;

$message = Request::create('/path?query=123', 'GET');
$message->headers->replace(array(
  'Date' => 'Wed, 30 Jul 2014 16:40:19 -0700',
  'Accept' => 'llamas',
));
```

### Signing a message

```rb
$context->signer()->sign($message);
```

Now `$message` contains the signature headers:

```rb
$message->headers->get('Signature');
# keyId="examplekey",algorithm="hmac-sha256",headers="...",signature="..."

$message->headers->get('Authorization');
# Signature keyId="examplekey",algorithm="hmac-sha256",headers="...",signature="..."
```

### Verifying a signed message

Message verification is not implemented, but will look like this:

* The key ID, algorithm name, header list and provided signature will be parsed
  from the `Signature` and/or `Authorization` header.
* The signing string will be derived by selecting the listed headers from the
  message.
* The valid signature will be derived by applying the algorithm and secret key.
* The message is valid if the provided signature matches the valid signature.


## Contributing

Pull Requests are welcome.


[draft03]: http://tools.ietf.org/html/draft-cavage-http-signatures-03
[Symfony\Component\HttpFoundation\Request]: https://github.com/symfony/HttpFoundation/blob/master/Request.php
