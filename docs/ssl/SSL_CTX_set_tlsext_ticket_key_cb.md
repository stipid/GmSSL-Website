# NAME

SSL\_CTX\_set\_tlsext\_ticket\_key\_cb - set a callback for session ticket processing

# SYNOPSIS

    #include <openssl/tls1.h>

    long SSL_CTX_set_tlsext_ticket_key_cb(SSL_CTX sslctx,
           int (*cb)(SSL *s, unsigned char key_name[16],
                     unsigned char iv[EVP_MAX_IV_LENGTH],
                     EVP_CIPHER_CTX *ctx, HMAC_CTX *hctx, int enc));

# DESCRIPTION

SSL\_CTX\_set\_tlsext\_ticket\_key\_cb() sets a callback function _cb_ for handling
session tickets for the ssl context _sslctx_. Session tickets, defined in
RFC5077 provide an enhanced session resumption capability where the server
implementation is not required to maintain per session state. It only applies
to TLS and there is no SSLv3 implementation.

The callback function _cb_ will be called for every client instigated TLS
session when session ticket extension is presented in the TLS hello
message. It is the responsibility of this function to create or retrieve the
cryptographic parameters and to maintain their state.

The OpenSSL library uses your callback function to help implement a common TLS
ticket construction state according to RFC5077 Section 4 such that per session
state is unnecessary and a small set of cryptographic variables needs to be
maintained by the callback function implementation.

In order to reuse a session, a TLS client must send the a session ticket
extension to the server. The client can only send exactly one session ticket.
The server, through the callback function, either agrees to reuse the session
ticket information or it starts a full TLS handshake to create a new session
ticket.

Before the callback function is started _ctx_ and _hctx_ have been
initialised with EVP\_CIPHER\_CTX\_init and HMAC\_CTX\_init respectively.

For new sessions tickets, when the client doesn't present a session ticket, or
an attempted retrieval of the ticket failed, or a renew option was indicated,
the callback function will be called with _enc_ equal to 1. The OpenSSL
library expects that the function will set an arbitrary _name_, initialize
_iv_, and set the cipher context _ctx_ and the hash context _hctx_.

The _name_ is 16 characters long and is used as a key identifier.

The _iv_ length is the length of the IV of the corresponding cipher. The
maximum IV length is **EVP\_MAX\_IV\_LENGTH** bytes defined in **evp.h**.

The initialization vector _iv_ should be a random value. The cipher context
_ctx_ should use the initialisation vector _iv_. The cipher context can be
set using [EVP\_EncryptInit\_ex(3)](http://man.he.net/man3/EVP_EncryptInit_ex). The hmac context can be set using
[HMAC\_Init\_ex(3)](http://man.he.net/man3/HMAC_Init_ex).

When the client presents a session ticket, the callback function with be called
with _enc_ set to 0 indicating that the _cb_ function should retrieve a set
of parameters. In this case _name_ and _iv_ have already been parsed out of
the session ticket. The OpenSSL library expects that the _name_ will be used
to retrieve a cryptographic parameters and that the cryptographic context
_ctx_ will be set with the retrieved parameters and the initialization vector
_iv_. using a function like [EVP\_DecryptInit\_ex(3)](http://man.he.net/man3/EVP_DecryptInit_ex). The _hctx_ needs to be
set using [HMAC\_Init\_ex(3)](http://man.he.net/man3/HMAC_Init_ex).

If the _name_ is still valid but a renewal of the ticket is required the
callback function should return 2. The library will call the callback again
with an argument of enc equal to 1 to set the new ticket.

The return value of the _cb_ function is used by OpenSSL to determine what
further processing will occur. The following return values have meaning:

- 2

    This indicates that the _ctx_ and _hctx_ have been set and the session can
    continue on those parameters. Additionally it indicates that the session
    ticket is in a renewal period and should be replaced. The OpenSSL library will
    call _cb_ again with an enc argument of 1 to set the new ticket (see RFC5077
    3.3 paragraph 2).

- 1

    This indicates that the _ctx_ and _hctx_ have been set and the session can
    continue on those parameters.

- 0

    This indicates that it was not possible to set/retrieve a session ticket and
    the SSL/TLS session will continue by negotiating a set of cryptographic
    parameters or using the alternate SSL/TLS resumption mechanism, session ids.

    If called with enc equal to 0 the library will call the _cb_ again to get
    a new set of parameters.

- less than 0

    This indicates an error.

# NOTES

Session resumption shortcuts the TLS so that the client certificate
negotiation don't occur. It makes up for this by storing client certificate
an all other negotiated state information encrypted within the ticket. In a
resumed session the applications will have all this state information available
exactly as if a full negotiation had occurred.

If an attacker can obtain the key used to encrypt a session ticket, they can
obtain the master secret for any ticket using that key and decrypt any traffic
using that session: even if the ciphersuite supports forward secrecy. As
a result applications may wish to use multiple keys and avoid using long term
keys stored in files.

Applications can use longer keys to maintain a consistent level of security.
For example if a ciphersuite uses 256 bit ciphers but only a 128 bit ticket key
the overall security is only 128 bits because breaking the ticket key will
enable an attacker to obtain the session keys.

# EXAMPLES

Reference Implementation:
  SSL\_CTX\_set\_tlsext\_ticket\_key\_cb(SSL, ssl\_tlsext\_ticket\_key\_cb);
  ....

    static int ssl_tlsext_ticket_key_cb(SSL *s, unsigned char key_name[16], unsigned char *iv, EVP_CIPHER_CTX *ctx, HMAC_CTX *hctx, int enc)
    {
        if (enc) { /* create new session */
            if (RAND_bytes(iv, EVP_MAX_IV_LENGTH) ) {
                return -1; /* insufficient random */
            }

            key = currentkey(); /* something that you need to implement */
            if ( !key ) {
                /* current key doesn't exist or isn't valid */
                key = createkey(); /* something that you need to implement.
                                     * createkey needs to initialise, a name,
                                     * an aes_key, a hmac_key and optionally
                                     * an expire time. */
                if ( !key ) { /* key couldn't be created */
                    return 0;
                }
            }
            memcpy(key_name, key->name, 16);

            EVP_EncryptInit_ex(&ctx, EVP_aes_128_cbc(), NULL, key->aes_key, iv);
            HMAC_Init_ex(&hctx, key->hmac_key, 16, EVP_sha256(), NULL);

            return 1;

        } else { /* retrieve session */
            key = findkey(name);

            if  (!key || key->expire < now() ) {
                return 0;
            }

            HMAC_Init_ex(&hctx, key->hmac_key, 16, EVP_sha256(), NULL);
            EVP_DecryptInit_ex(&ctx, EVP_aes_128_cbc(), NULL, key->aes_key, iv );

            if (key->expire < ( now() - RENEW_TIME ) ) {
                /* return 2 - this session will get a new ticket even though the current is still valid */
                return 2;
            }
            return 1;

        }
    }

# RETURN VALUES

returns 0 to indicate the callback function was set.

# SEE ALSO

[ssl(3)](http://man.he.net/man3/ssl), [SSL\_set\_session(3)](http://man.he.net/man3/SSL_set_session),
[SSL\_session\_reused(3)](http://man.he.net/man3/SSL_session_reused),
[SSL\_CTX\_add\_session(3)](http://man.he.net/man3/SSL_CTX_add_session),
[SSL\_CTX\_sess\_number(3)](http://man.he.net/man3/SSL_CTX_sess_number),
[SSL\_CTX\_sess\_set\_get\_cb(3)](http://man.he.net/man3/SSL_CTX_sess_set_get_cb),
[SSL\_CTX\_set\_session\_id\_context(3)](http://man.he.net/man3/SSL_CTX_set_session_id_context),

# COPYRIGHT

Copyright 2014-2016 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use
this file except in compliance with the License.  You can obtain a copy
in the file LICENSE in the source distribution or at
[https://www.openssl.org/source/license.html](https://www.openssl.org/source/license.html).