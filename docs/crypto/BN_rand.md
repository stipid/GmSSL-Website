# NAME

BN\_rand, BN\_pseudo\_rand, BN\_rand\_range, BN\_pseudo\_rand\_range - generate pseudo-random number

# SYNOPSIS

    #include <openssl/bn.h>

    int BN_rand(BIGNUM *rnd, int bits, int top, int bottom);

    int BN_pseudo_rand(BIGNUM *rnd, int bits, int top, int bottom);

    int BN_rand_range(BIGNUM *rnd, BIGNUM *range);

    int BN_pseudo_rand_range(BIGNUM *rnd, BIGNUM *range);

# DESCRIPTION

BN\_rand() generates a cryptographically strong pseudo-random number of
**bits** in length and stores it in **rnd**.
If **bits** is less than zero, or too small to
accomodate the requirements specified by the **top** and **bottom**
parameters, an error is returned.
The **top** parameters specifies
requirements on the most significant bit of the generated number.
If it is **BN\_RAND\_TOP\_ANY**, there is no constraint.
If it is **BN\_RAND\_TOP\_ONE**, the top bit must be one.
If it is **BN\_RAND\_TOP\_TWO**, the two most significant bits of
the number will be set to 1, so that the product of two such random
numbers will always have 2\***bits** length.
If **bottom** is **BN\_RAND\_BOTTOM\_ODD**, the number will be odd; if it
is **BN\_RAND\_BOTTOM\_ANY** it can be odd or even.
If **bits** is 1 then **top** cannot also be **BN\_RAND\_FLG\_TOPTWO**.

BN\_pseudo\_rand() does the same, but pseudo-random numbers generated by
this function are not necessarily unpredictable. They can be used for
non-cryptographic purposes and for certain purposes in cryptographic
protocols, but usually not for key generation etc.

BN\_rand\_range() generates a cryptographically strong pseudo-random
number **rnd** in the range 0 <= **rnd** < **range**.
BN\_pseudo\_rand\_range() does the same, but is based on BN\_pseudo\_rand(),
and hence numbers generated by it are not necessarily unpredictable.

The PRNG must be seeded prior to calling BN\_rand() or BN\_rand\_range().

# RETURN VALUES

The functions return 1 on success, 0 on error.
The error codes can be obtained by [ERR\_get\_error(3)](http://man.he.net/man3/ERR_get_error).

# SEE ALSO

[bn(3)](http://man.he.net/man3/bn), [ERR\_get\_error(3)](http://man.he.net/man3/ERR_get_error), [rand(3)](http://man.he.net/man3/rand),
[RAND\_add(3)](http://man.he.net/man3/RAND_add), [RAND\_bytes(3)](http://man.he.net/man3/RAND_bytes)

# COPYRIGHT

Copyright 2000-2016 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use
this file except in compliance with the License.  You can obtain a copy
in the file LICENSE in the source distribution or at
[https://www.openssl.org/source/license.html](https://www.openssl.org/source/license.html).