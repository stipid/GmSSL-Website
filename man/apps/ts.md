# NAME

ts - Time Stamping Authority tool (client/server)

# SYNOPSIS

**gmssl** **ts**
**-query**
\[**-rand** file:file...\]
\[**-config** configfile\]
\[**-data** file\_to\_hash\]
\[**-digest** digest\_bytes\]
\[**-\[digest\]**\]
\[**-tspolicy** object\_id\]
\[**-no\_nonce**\]
\[**-cert**\]
\[**-in** request.tsq\]
\[**-out** request.tsq\]
\[**-text**\]

**gmssl** **ts**
**-reply**
\[**-config** configfile\]
\[**-section** tsa\_section\]
\[**-queryfile** request.tsq\]
\[**-passin** password\_src\]
\[**-signer** tsa\_cert.pem\]
\[**-inkey** private.pem\]
\[**-sha1|-sha224|-sha256|-sha384|-sha512**\]
\[**-chain** certs\_file.pem\]
\[**-tspolicy** object\_id\]
\[**-in** response.tsr\]
\[**-token\_in**\]
\[**-out** response.tsr\]
\[**-token\_out**\]
\[**-text**\]
\[**-engine** id\]

**gmssl** **ts**
**-verify**
\[**-data** file\_to\_hash\]
\[**-digest** digest\_bytes\]
\[**-queryfile** request.tsq\]
\[**-in** response.tsr\]
\[**-token\_in**\]
\[**-CApath** trusted\_cert\_path\]
\[**-CAfile** trusted\_certs.pem\]
\[**-untrusted** cert\_file.pem\]
\[_verify options_\]

_verify options:_
\[-attime timestamp\]
\[-check\_ss\_sig\]
\[-crl\_check\]
\[-crl\_check\_all\]
\[-explicit\_policy\]
\[-extended\_crl\]
\[-ignore\_critical\]
\[-inhibit\_any\]
\[-inhibit\_map\]
\[-issuer\_checks\]
\[-no\_alt\_chains\]
\[-no\_check\_time\]
\[-partial\_chain\]
\[-policy arg\]
\[-policy\_check\]
\[-policy\_print\]
\[-purpose purpose\]
\[-suiteB\_128\]
\[-suiteB\_128\_only\]
\[-suiteB\_192\]
\[-trusted\_first\]
\[-use\_deltas\]
\[-auth\_level num\]
\[-verify\_depth num\]
\[-verify\_email email\]
\[-verify\_hostname hostname\]
\[-verify\_ip ip\]
\[-verify\_name name\]
\[-x509\_strict\]

# DESCRIPTION

The **ts** command is a basic Time Stamping Authority (TSA) client and server
application as specified in RFC 3161 (Time-Stamp Protocol, TSP). A
TSA can be part of a PKI deployment and its role is to provide long
term proof of the existence of a certain datum before a particular
time. Here is a brief description of the protocol:

1. The TSA client computes a one-way hash value for a data file and sends
the hash to the TSA.
2. The TSA attaches the current date and time to the received hash value,
signs them and sends the time stamp token back to the client. By
creating this token the TSA certifies the existence of the original
data file at the time of response generation.
3. The TSA client receives the time stamp token and verifies the
signature on it. It also checks if the token contains the same hash
value that it had sent to the TSA.

There is one DER encoded protocol data unit defined for transporting a time
stamp request to the TSA and one for sending the time stamp response
back to the client. The **ts** command has three main functions:
creating a time stamp request based on a data file,
creating a time stamp response based on a request, verifying if a
response corresponds to a particular request or a data file.

There is no support for sending the requests/responses automatically
over HTTP or TCP yet as suggested in RFC 3161. The users must send the
requests either by ftp or e-mail.

# OPTIONS

## Time Stamp Request generation

The **-query** switch can be used for creating and printing a time stamp
request with the following options:

- **-rand** file:file...

    The files containing random data for seeding the random number
    generator. Multiple files can be specified, the separator is **;** for
    MS-Windows, **,** for VMS and **:** for all other platforms. (Optional)

- **-config** configfile

    The configuration file to use, this option overrides the
    **OPENSSL\_CONF** environment variable. Only the OID section
    of the config file is used with the **-query** command. (Optional)

- **-data** file\_to\_hash

    The data file for which the time stamp request needs to be
    created. stdin is the default if neither the **-data** nor the **-digest**
    parameter is specified. (Optional)

- **-digest** digest\_bytes

    It is possible to specify the message imprint explicitly without the data
    file. The imprint must be specified in a hexadecimal format, two characters
    per byte, the bytes optionally separated by colons (e.g. 1A:F6:01:... or
    1AF601...). The number of bytes must match the message digest algorithm
    in use. (Optional)

- **-\[digest\]**

    The message digest to apply to the data file.
    Any digest supported by the GmSSL **dgst** command can be used.
    The default is SHA-1. (Optional)

- **-tspolicy** object\_id

    The policy that the client expects the TSA to use for creating the
    time stamp token. Either the dotted OID notation or OID names defined
    in the config file can be used. If no policy is requested the TSA will
    use its own default policy. (Optional)

- **-no\_nonce**

    No nonce is specified in the request if this option is
    given. Otherwise a 64 bit long pseudo-random none is
    included in the request. It is recommended to use nonce to
    protect against replay-attacks. (Optional)

- **-cert**

    The TSA is expected to include its signing certificate in the
    response. (Optional)

- **-in** request.tsq

    This option specifies a previously created time stamp request in DER
    format that will be printed into the output file. Useful when you need
    to examine the content of a request in human-readable

    format. (Optional)

- **-out** request.tsq

    Name of the output file to which the request will be written. Default
    is stdout. (Optional)

- **-text**

    If this option is specified the output is human-readable text format
    instead of DER. (Optional)

## Time Stamp Response generation

A time stamp response (TimeStampResp) consists of a response status
and the time stamp token itself (ContentInfo), if the token generation was
successful. The **-reply** command is for creating a time stamp
response or time stamp token based on a request and printing the
response/token in human-readable format. If **-token\_out** is not
specified the output is always a time stamp response (TimeStampResp),
otherwise it is a time stamp token (ContentInfo).

- **-config** configfile

    The configuration file to use, this option overrides the
    **OPENSSL\_CONF** environment variable. See **CONFIGURATION FILE
    OPTIONS** for configurable variables. (Optional)

- **-section** tsa\_section

    The name of the config file section containing the settings for the
    response generation. If not specified the default TSA section is
    used, see **CONFIGURATION FILE OPTIONS** for details. (Optional)

- **-queryfile** request.tsq

    The name of the file containing a DER encoded time stamp request. (Optional)

- **-passin** password\_src

    Specifies the password source for the private key of the TSA. See
    **PASS PHRASE ARGUMENTS** in [gmssl(1)](http://man.he.net/man1/gmssl). (Optional)

- **-signer** tsa\_cert.pem

    The signer certificate of the TSA in PEM format. The TSA signing
    certificate must have exactly one extended key usage assigned to it:
    timeStamping. The extended key usage must also be critical, otherwise
    the certificate is going to be refused. Overrides the **signer\_cert**
    variable of the config file. (Optional)

- **-inkey** private.pem

    The signer private key of the TSA in PEM format. Overrides the
    **signer\_key** config file option. (Optional)

- **-sha1|-sha224|-sha256|-sha384|-sha512**

    Signing digest to use. Overrides the **signer\_digest** config file
    option. (Optional)

- **-chain** certs\_file.pem

    The collection of certificates in PEM format that will all
    be included in the response in addition to the signer certificate if
    the **-cert** option was used for the request. This file is supposed to
    contain the certificate chain for the signer certificate from its
    issuer upwards. The **-reply** command does not build a certificate
    chain automatically. (Optional)

- **-tspolicy** object\_id

    The default policy to use for the response unless the client
    explicitly requires a particular TSA policy. The OID can be specified
    either in dotted notation or with its name. Overrides the
    **default\_policy** config file option. (Optional)

- **-in** response.tsr

    Specifies a previously created time stamp response or time stamp token
    (if **-token\_in** is also specified) in DER format that will be written
    to the output file. This option does not require a request, it is
    useful e.g. when you need to examine the content of a response or
    token or you want to extract the time stamp token from a response. If
    the input is a token and the output is a time stamp response a default
    'granted' status info is added to the token. (Optional)

- **-token\_in**

    This flag can be used together with the **-in** option and indicates
    that the input is a DER encoded time stamp token (ContentInfo) instead
    of a time stamp response (TimeStampResp). (Optional)

- **-out** response.tsr

    The response is written to this file. The format and content of the
    file depends on other options (see **-text**, **-token\_out**). The default is
    stdout. (Optional)

- **-token\_out**

    The output is a time stamp token (ContentInfo) instead of time stamp
    response (TimeStampResp). (Optional)

- **-text**

    If this option is specified the output is human-readable text format
    instead of DER. (Optional)

- **-engine** id

    Specifying an engine (by its unique **id** string) will cause **ts**
    to attempt to obtain a functional reference to the specified engine,
    thus initialising it if needed. The engine will then be set as the default
    for all available algorithms. Default is builtin. (Optional)

## Time Stamp Response verification

The **-verify** command is for verifying if a time stamp response or time
stamp token is valid and matches a particular time stamp request or
data file. The **-verify** command does not use the configuration file.

- **-data** file\_to\_hash

    The response or token must be verified against file\_to\_hash. The file
    is hashed with the message digest algorithm specified in the token.
    The **-digest** and **-queryfile** options must not be specified with this one.
    (Optional)

- **-digest** digest\_bytes

    The response or token must be verified against the message digest specified
    with this option. The number of bytes must match the message digest algorithm
    specified in the token. The **-data** and **-queryfile** options must not be
    specified with this one. (Optional)

- **-queryfile** request.tsq

    The original time stamp request in DER format. The **-data** and **-digest**
    options must not be specified with this one. (Optional)

- **-in** response.tsr

    The time stamp response that needs to be verified in DER format. (Mandatory)

- **-token\_in**

    This flag can be used together with the **-in** option and indicates
    that the input is a DER encoded time stamp token (ContentInfo) instead
    of a time stamp response (TimeStampResp). (Optional)

- **-CApath** trusted\_cert\_path

    The name of the directory containing the trusted CA certificates of the
    client. See the similar option of [verify(1)](http://man.he.net/man1/verify) for additional
    details. Either this option or **-CAfile** must be specified. (Optional)

- **-CAfile** trusted\_certs.pem

    The name of the file containing a set of trusted self-signed CA
    certificates in PEM format. See the similar option of
    [verify(1)](http://man.he.net/man1/verify) for additional details. Either this option
    or **-CApath** must be specified.
    (Optional)

- **-untrusted** cert\_file.pem

    Set of additional untrusted certificates in PEM format which may be
    needed when building the certificate chain for the TSA's signing
    certificate. This file must contain the TSA signing certificate and
    all intermediate CA certificates unless the response includes them.
    (Optional)

- _verify options_

    The options **-attime timestamp**, **-check\_ss\_sig**, **-crl\_check**,
    **-crl\_check\_all**, **-explicit\_policy**, **-extended\_crl**, **-ignore\_critical**,
    **-inhibit\_any**, **-inhibit\_map**, **-issuer\_checks**, **-no\_alt\_chains**,
    **-no\_check\_time**, **-partial\_chain**, **-policy**, **-policy\_check**,
    **-policy\_print**, **-purpose**, **-suiteB\_128**, **-suiteB\_128\_only**,
    **-suiteB\_192**, **-trusted\_first**, **-use\_deltas**, **-auth\_level**,
    **-verify\_depth**, **-verify\_email**, **-verify\_hostname**, **-verify\_ip**,
    **-verify\_name**, and **-x509\_strict** can be used to control timestamp
    verification.  See [verify(1)](http://man.he.net/man1/verify).

# CONFIGURATION FILE OPTIONS

The **-query** and **-reply** commands make use of a configuration file
defined by the **OPENSSL\_CONF** environment variable. See [config(5)](http://man.he.net/man5/config)
for a general description of the syntax of the config file. The
**-query** command uses only the symbolic OID names section
and it can work without it. However, the **-reply** command needs the
config file for its operation.

When there is a command line switch equivalent of a variable the
switch always overrides the settings in the config file.

- **tsa** section, **default\_tsa**

    This is the main section and it specifies the name of another section
    that contains all the options for the **-reply** command. This default
    section can be overridden with the **-section** command line switch. (Optional)

- **oid\_file**

    See [ca(1)](http://man.he.net/man1/ca) for description. (Optional)

- **oid\_section**

    See [ca(1)](http://man.he.net/man1/ca) for description. (Optional)

- **RANDFILE**

    See [ca(1)](http://man.he.net/man1/ca) for description. (Optional)

- **serial**

    The name of the file containing the hexadecimal serial number of the
    last time stamp response created. This number is incremented by 1 for
    each response. If the file does not exist at the time of response
    generation a new file is created with serial number 1. (Mandatory)

- **crypto\_device**

    Specifies the GmSSL engine that will be set as the default for
    all available algorithms. The default value is builtin, you can specify
    any other engines supported by GmSSL (e.g. use chil for the NCipher HSM).
    (Optional)

- **signer\_cert**

    TSA signing certificate in PEM format. The same as the **-signer**
    command line option. (Optional)

- **certs**

    A file containing a set of PEM encoded certificates that need to be
    included in the response. The same as the **-chain** command line
    option. (Optional)

- **signer\_key**

    The private key of the TSA in PEM format. The same as the **-inkey**
    command line option. (Optional)

- **signer\_digest**

    Signing digest to use. The same as the
    **-sha1|-sha224|-sha256|-sha384|-sha512** command line option. (Optional)

- **default\_policy**

    The default policy to use when the request does not mandate any
    policy. The same as the **-tspolicy** command line option. (Optional)

- **other\_policies**

    Comma separated list of policies that are also acceptable by the TSA
    and used only if the request explicitly specifies one of them. (Optional)

- **digests**

    The list of message digest algorithms that the TSA accepts. At least
    one algorithm must be specified. (Mandatory)

- **accuracy**

    The accuracy of the time source of the TSA in seconds, milliseconds
    and microseconds. E.g. secs:1, millisecs:500, microsecs:100. If any of
    the components is missing zero is assumed for that field. (Optional)

- **clock\_precision\_digits**

    Specifies the maximum number of digits, which represent the fraction of
    seconds, that  need to be included in the time field. The trailing zeroes
    must be removed from the time, so there might actually be fewer digits,
    or no fraction of seconds at all. Supported only on UNIX platforms.
    The maximum value is 6, default is 0.
    (Optional)

- **ordering**

    If this option is yes the responses generated by this TSA can always
    be ordered, even if the time difference between two responses is less
    than the sum of their accuracies. Default is no. (Optional)

- **tsa\_name**

    Set this option to yes if the subject name of the TSA must be included in
    the TSA name field of the response. Default is no. (Optional)

- **ess\_cert\_id\_chain**

    The SignedData objects created by the TSA always contain the
    certificate identifier of the signing certificate in a signed
    attribute (see RFC 2634, Enhanced Security Services). If this option
    is set to yes and either the **certs** variable or the **-chain** option
    is specified then the certificate identifiers of the chain will also
    be included in the SigningCertificate signed attribute. If this
    variable is set to no, only the signing certificate identifier is
    included. Default is no. (Optional)

# ENVIRONMENT VARIABLES

**OPENSSL\_CONF** contains the path of the configuration file and can be
overridden by the **-config** command line option.

# EXAMPLES

All the examples below presume that **OPENSSL\_CONF** is set to a proper
configuration file, e.g. the example configuration file
gmssl/apps/openssl.cnf will do.

## Time Stamp Request

To create a time stamp request for design1.txt with SHA-1
without nonce and policy and no certificate is required in the response:

    gmssl ts -query -data design1.txt -no_nonce \
          -out design1.tsq

To create a similar time stamp request with specifying the message imprint
explicitly:

    gmssl ts -query -digest b7e5d3f93198b38379852f2c04e78d73abdd0f4b \
           -no_nonce -out design1.tsq

To print the content of the previous request in human readable format:

    gmssl ts -query -in design1.tsq -text

To create a time stamp request which includes the MD-5 digest
of design2.txt, requests the signer certificate and nonce,
specifies a policy id (assuming the tsa\_policy1 name is defined in the
OID section of the config file):

    gmssl ts -query -data design2.txt -md5 \
          -tspolicy tsa_policy1 -cert -out design2.tsq

## Time Stamp Response

Before generating a response a signing certificate must be created for
the TSA that contains the **timeStamping** critical extended key usage extension
without any other key usage extensions. You can add the
'extendedKeyUsage = critical,timeStamping' line to the user certificate section
of the config file to generate a proper certificate. See [req(1)](http://man.he.net/man1/req),
[ca(1)](http://man.he.net/man1/ca), [x509(1)](http://man.he.net/man1/x509) for instructions. The examples
below assume that cacert.pem contains the certificate of the CA,
tsacert.pem is the signing certificate issued by cacert.pem and
tsakey.pem is the private key of the TSA.

To create a time stamp response for a request:

    gmssl ts -reply -queryfile design1.tsq -inkey tsakey.pem \
          -signer tsacert.pem -out design1.tsr

If you want to use the settings in the config file you could just write:

    gmssl ts -reply -queryfile design1.tsq -out design1.tsr

To print a time stamp reply to stdout in human readable format:

    gmssl ts -reply -in design1.tsr -text

To create a time stamp token instead of time stamp response:

    gmssl ts -reply -queryfile design1.tsq -out design1_token.der -token_out

To print a time stamp token to stdout in human readable format:

    gmssl ts -reply -in design1_token.der -token_in -text -token_out

To extract the time stamp token from a response:

    gmssl ts -reply -in design1.tsr -out design1_token.der -token_out

To add 'granted' status info to a time stamp token thereby creating a
valid response:

    gmssl ts -reply -in design1_token.der -token_in -out design1.tsr

## Time Stamp Verification

To verify a time stamp reply against a request:

    gmssl ts -verify -queryfile design1.tsq -in design1.tsr \
          -CAfile cacert.pem -untrusted tsacert.pem

To verify a time stamp reply that includes the certificate chain:

    gmssl ts -verify -queryfile design2.tsq -in design2.tsr \
          -CAfile cacert.pem

To verify a time stamp token against the original data file:
  gmssl ts -verify -data design2.txt -in design2.tsr \\
        -CAfile cacert.pem

To verify a time stamp token against a message imprint:
  gmssl ts -verify -digest b7e5d3f93198b38379852f2c04e78d73abdd0f4b \\
         -in design2.tsr -CAfile cacert.pem

You could also look at the 'test' directory for more examples.

# BUGS

If you find any bugs or you have suggestions please write to
Zoltan Glozik <zglozik@opentsa.org>. Known issues:

- No support for time stamps over SMTP, though it is quite easy
to implement an automatic e-mail based TSA with [procmail(1)](http://man.he.net/man1/procmail)
and [perl(1)](http://man.he.net/man1/perl). HTTP server support is provided in the form of
a separate apache module. HTTP client support is provided by
[tsget(1)](http://man.he.net/man1/tsget). Pure TCP/IP protocol is not supported.
- The file containing the last serial number of the TSA is not
locked when being read or written. This is a problem if more than one
instance of [gmssl(1)](http://man.he.net/man1/gmssl) is trying to create a time stamp
response at the same time. This is not an issue when using the apache
server module, it does proper locking.
- Look for the FIXME word in the source files.
- The source code should really be reviewed by somebody else, too.
- More testing is needed, I have done only some basic tests (see
test/testtsa).

# SEE ALSO

[tsget(1)](http://man.he.net/man1/tsget), [gmssl(1)](http://man.he.net/man1/gmssl), [req(1)](http://man.he.net/man1/req),
[x509(1)](http://man.he.net/man1/x509), [ca(1)](http://man.he.net/man1/ca), [genrsa(1)](http://man.he.net/man1/genrsa),
[config(5)](http://man.he.net/man5/config)

# COPYRIGHT

Copyright 2006-2016 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the GmSSL license (the "License").  You may not use
this file except in compliance with the License.  You can obtain a copy
in the file LICENSE in the source distribution or at
[https://www.openssl.org/source/license.html](https://www.openssl.org/source/license.html).
