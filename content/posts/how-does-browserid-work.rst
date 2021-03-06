How does BrowserID work?
########################

:date: 2012-05-07 23:01:04
:category: Security
:tags: BrowserID, OpenID, Security

This article will explain how the cryptography behind BrowserID works, and
lightly cover why BrowserID is a better alternative to OpenID.

A website will ask your browser for a BrowserID assertion via JavaScript.
This will either use a native BrowserID API in your browser, or it will use
the JavaScript implementation of BrowserID. This is known as the user agent.

.. code-block:: js

    navigator.id.get(gotAssertion);

For supporting browsers, the user agent will be supplied and the
`navigator.id.get` method will exist. The user agent can be provided by either
the browser, an extension to the browser, or via a JavaScript include on the
website to use a JavaScript implementation.

.. code-block:: html

    <script src="https://browserid.org/include.js" type="text/javascript"></script>

This means that a browser or browser extension can provide their own
implementation, or a JavaScript implementation can be used in unsupported
browsers.

Out of the process, once the browser has an assertion. It will provide the
assertion to the `gotAssertion` callback. The website can then provide this
back to the web server to validate the assertion to confirm the user's identity.

What is an assertion?
=====================

An assertion is a string which holds a JSON structure including an identity
assertion and an identity certificate. The assertion will be generated by the
browser for an identity, the browser will need to get an identity certificate
from a provider to ensure they own that identity. This identity can be used
again and again until it expires.

Identity certificate
--------------------

An identity certificate will look like the following:

.. code-block:: js

    {
        "iss": "kylefuller.co.uk",
        "exp": "1313971280961",
        "public-key": {
            ...
        },
        "principal": {
            "email": "inbox@kylefuller.co.uk"
        }
    }

The "iss" field will include the domain of the issuer. The code checking the
assertion should verify that is either the domain used in the email address, or
a trusted fallback provider, such as `browserid.org`. This means that either
the domain providing your email address or `browserid.org` can provide a valid
identity certificate for your email.

The fallback provider will allow you to use that provider instead of your email
provider in the situation that your email provider does not support BrowserID.

The above identity certificate signed by the issuer, and their certificate
should be available over HTTPS at `https://iss/.well-known/browserid`.

The public key in the identity certificate is the public key of the user. This
key can be generated by the browser so only the user will have control over it.

Identity assertion
------------------

The identity assertion is the final piece in a BrowserID assertion. It will be
signed by the browser's private key, so it will match the public key in the
identity certificate which our provider has confirmed we own.

.. code-block:: js

    {
        "exp": 1320280579437,
        "aud": "https://demand-browserid.herokuapp.com"
    }

The identity assertion provides an expiry and an audience. The identity
assertion allows a website to confirm the assertion is meant for them, the
audience should be the website that we want to sign-in to.

The relying party which the user is identifying to should check both the
identity certificate and the identity assertion to check the certificates
match, and that neither has expired. If these are valid assertions for our
audience, then we can confirm the user owns the email address and in the
identity certificate.

Why use BrowserID, whats wrong with OpenID?
===========================================

There are many benefits of BrowserID over OpenID.

* BrowserID is far simpler to use than OpenID for an average user. It is also
  quicker, since you do not need to type your OpenID into each site.
* When you sign-in to a site using OpenID, your OpenID provider knows you are
  visiting that site. With BrowserID, your BrowserID provider is not aware of
  the site you are visiting since your assertion is created in the browser. You
  have already retrieved your identity certificate from the provider. Your
  privacy is protected with BrowserID.
* BrowserID uses an email address, any existing site will already have your
  email address. This would allow a site to start using BrowserID without
  requiring you to provide any additional information. 

