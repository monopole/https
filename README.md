# HyperText Transfer Protocol Secure

In the old days, when we watered our lawns and nobody
thought that ice cream was bad for you, the World Wide
Web was a village square for the exchange of
unencrypted high energy physics data and cat photos.

Then came commerce, private data, and the need for
authentication and authorization. The free-wheeling
days of unencrypted HTTP ended. Web browsers now want
to speak _HTTPS_ exclusively.

## Public Key Cryptography

Using the Diffie-Hellman algorithm, Alice creates a
pair of entangled keys - one public, the other private.
Each of the two keys is a byte array that can be base64
encoded as a printable string.  The bigger the array,
the more secure the key.

Alice prints her _public_ key on paper and hands it out
to whomever asks, and keeps the corresponding private
key secret.

Bob obtains Alice's public key directly from Alice's hand.

Whenever he wants, Bob can use that key to create an
encrypted version of a message.  Bob can send this
encrypted text over any public channel, e.g. printed on
a post card.

Only Alice can decode this message, using her private
key.

So far we're in a simple world where _the only way for
a public key recipient to know that a key is associated
with Alice is to accept the paper from her hand while
looking at her face_.

At the core of every authentication system is the idea
that a specific human knows and trusts another specific
human, who knows and trusts another, etc.

## Cryptographic Signing

Alice wants to let people know that she approves of
some text blob - a poem, a means to cure cancer, a
base64-encoded cat photo, etc.

One way Alice could do this would be to print the text,
and physically hand it to people while saying _I
approve this._

But say she wants to do this less synchronously, e.g.
by mailing the text to Bob, who happens to be off
trekking in Tibet with nothing but a HAM radio.

So Alice runs the text through a publically known
hashing algorithm (e.g. SHA-256) to generate a
relatively short string - a hash. The algoritm is such
that any change in the text would create a completly
different hash.

Alice then uses Diffie-Hellman to _encrypt_ the hash
using her _private_ key. The encrypted hash is called a
_signature_. The signature can be converted to a
printable string (base64 again) and sent via email,
morse code, etc.

Bob obtained Alice's public key from her hand before
leaving on his trek, and he knows the hashing
algorithm.  Bob receives the blob of text and the
signature via morse code over HAM radio.

Bob hashes the text, and descrypts the signature using
Alice's public key.  If the two results match, Bob
knows that he has the exact text that Alice signed.

## Trust Chains

Introduce Cindy.

For Cindy to send encrypted messages to Alice, or to
verify that Alice signed something, Cindy must, just
once, physically visit Alice and get her public key
from her hand.

Such a visit might be inconvenient.

Assume that Cindy knows and trusts Bob, and at some
point has obtained Bob's public key directly from Bob.

Cindy can get Alice's public key directly from Bob's
hand, with Bob's verbal assurance that he obtained the
key directly from Alice.

If Cindy isn't near Bob, Cindy can ask Bob to send
Alice's public key via email.

Bob, wanting to make sure that Cindy gets the correct
key with proper context, makes a text blob with
multiple key-value pairs; Alice's name, Alice's public
key, the date the blob was created, etc.

Bob also adds a field _for his own name_ so that anyone
reading the text blob can know who created it.

Since Bob is going to send this over a public channel
(email), he wants to give Cindy a means to verify that
it hasn't been altered.  So Bob generates a signature
for the text blob using his own private key.

Now Bob needs to send the blob and its distinct
signature together to Cindy.

There are many ways to do this. For example, Bob can
compress and encode the blob to a single printable
string, and likewise encode the signature to a single
string. Bob can then create a single printable file
with two named fields, the _blob_ and the blob's
_signature_.

Bob triumphantly calls this file a _certificate_. Bob
emails the certificate to Cindy.  Cindy knows how to
unpack all this, and uses Bob's public key (which she
already has via other means) to verify the signature.

Either way, manually or via email, Cindy has received
Alice's public key from Bob, and can now send encrypted
messages to Alice without Bob's involvement.

### Betrayal

If Bob wants to secretly decrypt a message that Cindy
sends to Alice, Bob must do two things.

 * Bob must at the outset secretly generate a fresh
   public/private pair, hand the public key to Cindy,
   and falsely claim that it came from Alice.

 * Bob has to intercept and stop all messages Cindy
   sends to Alice.

Bob can decrypt Cindy's messages, read them, then
re-encrypt them using Alice's true public key, then
send them on to Alice, concealing the interception.

If Bob fails to intercept, Alice will get a message
from Cindy that she cannot decrypt, which will arouse
suspicion.

Further, Bob is obligated to do all this for all
messages coming from anyone that Cindy passed the
fake Alice key to. This seems daunting, but might be
eased in certain corporate situations with standardized
communication mechanisms.  Predictability eases
espionage.

### Chains of trust

In the above, Bob mentions himself (Bob) in the text
blob as the certificate issuer (signer). Cindy, who has
Bob's public key but not Alice's, verifies that this
certificate hasn't been tampered with, and obtains
Alices public key from the certificate so that she can
encrypt messages directly to Alice.

Now consider Don, Dan or Dave. Assume that they have
neither Alice's nor Bob's public keys, but they do know
Cindy.

Even though these keys are _public_, one cannot be sure
one has the _true_ key unless the key is obtained from
someone directly trusted.

Cindy can create a new certificate, _embedding Bob's
certificate (containing Alice's key) inside_.  Cindy
can sign this, and send the new certificate and her
signature via email to Don, Dan and Dave.

> ```
>                             ┌──► Don
>                             │
> Alice ───► Bob  ───► Cindy ─┼──► Dan  ───► Emma ───► Frank
>                             │
>                             └──► Dave ───► Erin
> ```

Cindy could just embed Alice's public key directly in
the certificate she creates, but she wants to let Don,
Dan and Dave know that she's an intermediary - that
she's passing along information she got from Bob.

This can continue - Frank can get Alice's public key
from Emma, who got it from Dan, who got it from Cindy,
who got it from Bob, who got it from Alice.

A message from Frank to Alice goes directly between the
two, but it's been encrypted with a public key that
passed through four intermediaries.

## Overview of HTTPS

HTTPS is based on the notion of securely obtaining
public keys via chains of trust.

HTTPS needs one more concept - the notion of
_certificate authority_ that vouches for the identity
claims being made by some third party (a server) to a
large audience (the web). This is detailed below.


### Requesting an HTTPS Session

When you use a web browser to talk to a bank, you want
to know that your account password is going to the
bank, not a bank impersonator.

This is done via public key encryption and,
unavoidably, a certain amount of trust.

An HTTPS session begins as follows:

 * Using a browser, you send a `GET` request to a server
   at some _domain_, e.g. https://example.com.

 * Before serving any HTML (e.g. HTML that presents a
   password form), the server at the `example.com`
   domain sends, in the clear (albeit compressed), a
   _server certificate_ to the browser.

 * The certificate is interpreted by the browser as a
   pair of things:

   - A blob of clear text.

   - A signature attesting to the validity of the text.

   The text is several key-value pairs, including at
   least:

   - The issuee's identifier (to whom the
     certificate was issued).

     In this case, `example.com`. It could also be a
     trademarked name.

   - The issuee's public encryption key.

   - An expiration date for this information.

   - An _embedded_ certificate representing the _issuer_
     of the encapsulating certificate.

     Another term for the issuere is _Certificate
     Authority_ (CA).

### Root Certificates

A server certificate should never have an empty issuer
field, because then nothing vouches for the validity of
the issuee's (server's) public key.

However, such a certificate can and must exist.  An
empty issuer field indicates that the certificate is
self-issued.  This means that the issuee's identifier
and public key (found in the certificate) are in fact
the name and public key of the _issuer_.

Such a certificate is called a _self-signed
certificate_ or _root certificate_.  Any (valid)
certificate chain ends with a root certificate.

### Validating an HTTPS Session

The browser should abort the session if the expiration
date has passed or if the issuee's identifier doesn't
match the domain targetted by the initial `GET`
request.

The issuee's (server's) public key in this bundle is
what the browser needs to start sending encrypted data
to the server.

How can the browser know that this key is truly the
public key associated with `example.com`?

Continuing:

 * The browser pulls the public key from the issuer's
   certificate (embedded in the server certificate),
   and uses it to verify the signature on the server
   certificate.

   The browser then recursively examines the issuer of
   the issuer's certificate, until it finds the end - a
   root certificate.  Any signature validation failure
   in this sequence aborts the HTTPS session.

 * A browser has, in its local store, a list of root
   certificates that the browser simply trusts as
   valid.

 * The browser compares the root certificate from the
   request to the root certificate that the browser
   finds in its local store.

   If they match, the information about the server can
   be trusted, and an encrypted session can be
   established using the server's public key.  A local
   lookup failure or information mismatch aborts the
   HTTPS session.

A certificate is static data that is infrequently
obtained from the CA by the server's operators.  Its
term of validity is typically months or years.


### How was the certificate issued to the server?

The human administrators that run the server must
obtain a certificate from a CA.

The CA asks the administrators to prove that they own
the domain, typically by having the server serve some
randomly generated phrase from that domain within a
specific time period.

If the CA can see this proof, then the CA serves a
certificate, signed by the CA, with the CA's own
certificate embedded as the issuer, to the server
administrators.

The server administrators then make this certificate
available to their server via some storage mechanism.

How can the server adminstrators trust the CA's domain
in order to download certificates?

This is done via HTTPS using root certificates that are
_baked into the browser by the manufacturer_.


### How does the browser get root certificates?

A human at the browser manufacturer must get a
certificate from the root CA via some mechanism other
than HTTPS (obviously).

For example, a human from the browser manufacturing
team can drive over to a building at a street address
known to belong to the CA, and get the certificate on a
thumb drive from some person that claims to work there.
There's always some human to human thing at the bottom.

On top of this, any company that manages a fleet of
browsers has the ability to install root certificates
in its browsers.

This allows the company to act as its own certificate
authority for internal servers.

[chrome://certificate-manager/crscerts]: chrome://certificate-manager/crscerts

In chrome, _root certificates_ can be found at
the psuedo-URL [chrome://certificate-manager/crscerts].



### Upshot

You, as the user of the browser, can be sure you are
talking to the bank's domain if you trust the browser's
manufacturer.

The manufacturer in turn makes sure that the banks
domain is showing the URL bar, and that the cert
supplied by the bank's server is validated against a
trusted root authority.
