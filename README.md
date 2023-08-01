# OpenSSL

- [OpenSSL](#openssl)
  - [Section 1](#section-1)
  - [Section 2](#section-2)
    - [What is SSL/TLS? What is HTTPS?](#what-is-ssltls-what-is-https)
    - [How do SSL/TLS Protect Your Data?](#how-do-ssltls-protect-your-data)
    - [Anti-Replay and Non-Repudiation](#anti-replay-and-non-repudiation)
    - [Key Players](#key-players)
    - [Hashing](#hashing)
    - [Data Integrity](#data-integrity)
    - [Encryption](#encryption)
    - [Public and Private Keys](#public-and-private-keys)
    - [How TLS/SSL Use Cryptography](#how-tlsssl-use-cryptography)
    - [PKI](#pki)
    - [RSA (Rivest Shamir and Edelman)](#rsa-rivest-shamir-and-edelman)
    - [Diffie Helman (DH)](#diffie-helman-dh)
    - [Digital Signature Algorithm (DSA)](#digital-signature-algorithm-dsa)
  - [Section 3](#section-3)
    - [Generating and Inspecting RSA Keys](#generating-and-inspecting-rsa-keys)
    - [Generating and Inspecting DSA Keys](#generating-and-inspecting-dsa-keys)
    - [Generating and Inspecting Elliptical Curve Keys](#generating-and-inspecting-elliptical-curve-keys)
  - [Section 4](#section-4)
    - [Adding and Removing Encryption to RSA Keys](#adding-and-removing-encryption-to-rsa-keys)
    - [OpenSSL PKey Utility](#openssl-pkey-utility)
    - [Matching Private Keys to Certificates and CSRs](#matching-private-keys-to-certificates-and-csrs)
  - [Section 5](#section-5)
    - [Generating Certificate Signing Requests (CSRs) and Self-Signed Certificates](#generating-certificate-signing-requests-csrs-and-self-signed-certificates)
  - [Section 6](#section-6)
    - [Extracting Specific Information From Certificates and CSRs](#extracting-specific-information-from-certificates-and-csrs)
  - [Section 7](#section-7)

## Section 1

## Section 2

### What is SSL/TLS? What is HTTPS?

The internet is many routers owned by many different ISPs.

-   On either end: you and who you are speaking to

Most common data transferred on the internet is websites,

-   Written in HTML (Hyper Text Markup Language)
-   Transferred with: HTTP (Hyper Text Transfer Protocol)

SSL / TLS create a secure, protected tunnel across the internet - HTTP - HTML transferred with HTTP protected by SSL.

-   SSL / TLS can also protect other data transfer
    -   SSL VPN
-   SSL - Secure Sockets Layer
    -   created by Netscape
-   TLS - Transport Layer Security
    -   IETF renamed protocol to TLS

### How do SSL/TLS Protect Your Data?

Data sent across a wire can be captured by anyone in the middle. The purpose of SSL/TLS is to protect the data in three ways:

-   **Confidentiality** - data is only accessible by client and server -> Encryption
-   **Integrity** - data is not modified between the client and server -> Hashing
-   **Authentication**- client/server are indeed who they are they are -> PKI

### Anti-Replay and Non-Repudiation

Anti-Replay

-   Provided with built-in sequence numbers
-   Built into integrity + authentication mechanism

Non-Repudiation

-   Repudiate - to refuse to have anything to do with
-   Sender cannot later deny sending a message
-   Byproduct of integrity + authentication

### Key Players

**Client**

-   Entity initiating the TLS handshake
-   Web browser
    -   Phone, apps, smart toaster, IOT...
-   Optionally authenticated (rare)

**Server**

-   Entity receiving the TLS handshake
    -   Web server
        -   Apache, IIS, NGINX, etc...
        -   Load balancer or SSL Accelerator
-   Always authenticated

**Certificate Authority**

-   Governing entity that issues certificates
-   Trusted by client and server
-   Provides trust anchor
    -   If we trust the CA, we trust what the CA trusts
-   Five organizations secure 98% of the internet:
    -   IdenTrust (51.9%)
        -   Let's Encrypt
    -   DigiCert (19.4%)
        -   Verisign
    -   Sectigo (17.5%)
    -   GoDaddy (6.9%)
    -   GlobalSign (2.9%)

### Hashing

Algorithm which takes as input a message of arbitrary length and produces as output a "fingerprint" fof the original message.

-   `"hello" -> 8 + 5 + 12 + 12 + 15`
-   Hashing is often referred to one-way encryption
-   Result of the hashing algorithm is called a digest
    -   Also called: checksum, fingerprint, hash, CRC, etc...
-   Real world hashing algorithms must satisfy four requirements:
    -   Infeasible to produce given digest
    -   Impossible to extract original message
    -   Slight changes to produce drastic differences
    -   Resulting digest is fixed width (length)

**Example:**

```sh
echo "hello" | md5sum
```

Collisions

-   Two messages result in identical digests
-   Cannot be avoided - It is a byproduct of "fixed width digests"

Common hashing algorithms:

-   MD5 - 128 bits
-   SHA/SHA1 - 160 bits
-   SHA2 family:
    -   SHA-224 - 224 bits
    -   SHA-256 - 256 bits
    -   SHA-384 - 384 bits
    -   SHA-512 - 512 bits

### Data Integrity

Hashing is used to provide integrity.

-   Sender calculates digest from the message
-   Sender sends message digest
-   Receiver calculates digest from the received message
-   Receiver compares both digests
    -   If the digests are identical, the message was not modified in transit
    -   However, simply hashing the message is not enough. The MIM can intercept message, modify the message, recalculate digest and then send both to receiver.
-   Both parties establish a mutual secret key
-   Sender combines messages + secret key to create digest
-   Receiver verifies by calculating hash of message + secret key
-   Message Authentication Code (MAC)
    -   Concept of combining message + secret key when calculating digest
    -   Provides integrity and authentication for bulk data transfer
    -   Message + secret key must be combined in the same way
    -   Industry standard implementation of MAC: HMAC
        -   Hash Based Message Authentication Code (RFC 2104)

### Encryption

Encryption if used to provide confidentiality.

-   Plain text: data before encryption
-   Cipher text: data while encryption

Simple encryption: transforms plain text into cipher text.

Key Based Encryption:

-   Combines industry vetted algorithm with a secret key
    -   Algorithms is created by experts
    -   Secret keys can be randomly generated

Two types of Key Based Encryption:

-   Symmetric Encryption
    -   Encrypt and decrypt using the same keys
    -   Ideal for bulk data
    -   Strength:
        -   Faster. Lower CPU cost.
        -   Cipher text is same size as plain text
    -   Weakness:
        -   Secret key must be shared - less secure
    -   DES, RC4, 3DES, AES, ChaCha20
-   Asymmetrical Encryption
    -   Encrypt and decrypt using different keys
    -   These two keys are mathematically related
    -   What one key encrypts, only the other can decrypt
    -   Restricted to limited data
    -   One key will be made public, other key will be kept private
    -   Weaknesses:
        -   Slower. Requires much larger key sizes.
        -   Cipher text expansion
    -   Strengths:
        -   Private key is never shared - more secure
    -   DSA, RSA, Diffie-Hellman, ECDSA, ECDH

### Public and Private Keys

**Example 1:**

-   Jim wants to send a secret message to Pam
-   Jim uses Pam's public key to encrypt the message
-   Only the correlating private key can decrypt

**Example 2:**

-   Pam wants to prove that she sent the message
-   Pam uses her private key to encrypt the message
-   If Jim can decrypt Pam's public key:

    -   Jim knows Pam must have sent the message
    -   Jim knows message was not modified in transit

-   Asymmetric Key Pairs can provide encryption and signatures
-   But what if we used asymmetric keys to share symmetric keys?

**Example 3:**

-   Pam randomly generates a symmetric secret key
-   Pam encrypts symmetric key with Jim's public key
-   Jim decrypts symmetric key with Jim's private key
-   Bulk data can now be symmetrically encrypted
    -   This can be done in either direction, for arbitrary amount of data
-   Hybrid Encryption - concept of using both asymmetric and symmetric encryption
    -   Asymmetric encryption to facilitate key exchange
    -   Secret key used with symmetrical encryption for bulk data

**Example 4:**

-   Process for using an asymmetrical key pair for signatures:
-   Pam calculates a hash of a message
-   Pam encrypts a hash of a message
-   Jim decrypts the signature with Pam's public key
-   Jim calculates hash of received message
-   If both digests, match, this proves two things:
    -   Message hasn't changed since Pam signed it
    -   Only Pam could have created the signatures

### How TLS/SSL Use Cryptography

CA is trusted by client.
CA will generate a certificate.
Certificate links an asymmetric key pair to a specific identity.
Certificate is signed by the CA (provides authentication).

### PKI

Three entities form a PKI: client, server, CA.
The client is the entity that needs to connect securely to something or to verify a particular identity.
The server is the entity that needs to prove its identity.
The Certificate Authority is the governing entity which verifies identities and generates certificates.
There could also be a corporate PKI.

### RSA (Rivest Shamir and Edelman)

RSA is by far the most popular asymmetric encryption algorithm in use today.
RSA creates a pair of commutative keys:

-   i.e. encrypt with one of the keys and decrypt with the other; it doesn't matter which order you go in.

DH/DSA work differently:

-   Semi-prime numbers are numbers that are divisible by prime numbers
-   RSA Example:
    -   Generating keys:
        -   Select two prime numbers (P, Q) - 7, 19
        -   Calculate product (P \* Q) - 133
        -   Calculate totient (P - 1) \* (Q - 1) - 108
        -   Select public key (E) - 29
            -   Must be prime
            -   Must be less than totient
            -   Must NOT be a factor of the totient
        -   Select private key (D) - 41
            -   Product of D and E, divided by T must result in a remainder of 1
            -   `(D * E) % T = 1`
    -   Encryption and Decryption
        -   Encryption:
            -   message ^E^ % N = cipher text
        -   Decryption:
            -   cipher ^D^ % N = message
    -   Encrypt with public key, decrypt with private key (works both ways)
        -   60^29^ % 133 = 86
        -   8g^41^ % 133 = 60
        -   60^41^ % 133 = 72
        -   72^29^ % 133 = 60
    -   How secure is RSA?
        -   Security lies in difficulty of factoring prime numbers
        -   If given 133, could you extract 7 and 19?
        -   How about 1909?
        -   In 1991, RSA Labs created the RSA Challenge. They released 54 different semi-private numbers of various sizes and offered cash prizes for anybody that could drive the prime factors.
            -   The competition ended in 2007 and only 12 of the original 54 had been solved
            -   As of 2020, another 11 were solved
            -   The biggest semi prime that was factored was 829 bits
            -   In the almost 30 years since these numbers were originally published, the 1024 bit number that they released has never been factored
            -   2048 bit RSA keys is the standard since 2015

### Diffie Helman (DH)

Allows two parties to establish a shared secret over an unsecured medium.
Shared secrets is never transmitted.
Security of DH is dependent on discrete logarithm problem.

### Digital Signature Algorithm (DSA)

DSA is a asymmetric encryption algorithm.
DSA simply creates and validates signatures.

-   No encryption, no key exchange

DSA simply has two operations:

-   Signature generation
    -   Input: message, private key, random #, dsa params
    -   Output: signature
-   Signature verification:
    -   Input: message, public key, signature, dsa params
    -   Output: true or false
-   Random numbers is very important
    -   Must be unique for each message or DSA fails catastrophically
    -   If random numbers is ever re-used, private key can be extracted
    -   RFC 6969 - generate random number deterministically based on message

## Section 3

### Generating and Inspecting RSA Keys

Generate 1024 bit RSA key:

-   `openssl genrsa -out RSA-key1.pem 1024`

Inspect RSA key:

-   `openssl rsa -in RSA-key1.pem -text`

Inspect RSA key without pem format:

-   `openssl rsa -in RSA-key1.pem -text - noout`

Generate 4096 bit RSA private key, encrypted with AES128:

-   safer if transferring it
-   `openssl genrsa -out RSA-key2.pem -aes128 4096`
-   `des` is insecure by today's standards

List all algorithms:

-   `openssl list -cipher-algorithms`

### Generating and Inspecting DSA Keys

Generate DSA parameters file

-   `openssl dsaparam -out DSA-PARAM.pem 1024`

Generate DSA keys file with parameters file

-   `openssl gendsa -out DSA-key.pem DSA-PARAM.pem`

Generate DSA Parameters and Keys in one File

-   `openssl dsaparam -genkey -out DSA-PARAM-KEY.pem 2048`

Inspecting DSA Parameters file

-   `openssl dsaparam -in DSA-PARAM.pem -text -noout`

Inspecting DSA Private Key file

-   `openssl dsa -in DSA-KEY.pem -text -noout`

### Generating and Inspecting Elliptical Curve Keys

Generate EC Parameters file

-   `openssl genpkey -genparam -algorithm EC -pkeyopt ec_paramgen_curve:secp384r1 -out EC-PARAM.pem`

Generate EC Keys from Parameters file

-   `openssl genpkey -paramfile EC-PARAM.pem -out EC-KEY.pem`

Generate EC Keys directly

-   `openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-384 -out EC-KEY.pem`

View supported Elliptic Curves

-   `openssl ecparam -list_curves`
-   Recommended Curves: prime256v1, secp384r1, secp521r1 (identical to P-256, P-384, P-521)

Inspecting Elliptic Curve (EC) Parameters file

-   `openssl ecparam -in EC-PARAM.pem -text -noout`

Inspecting Elliptic Curve (EC) Private Key file

-   `openssl ec -in EC-KEY.pem -text -noout`

## Section 4

### Adding and Removing Encryption to RSA Keys

Removing encryption from an RSA key file

-   `openssl rsa -in ENCRYPTED-KEY.pem -out KEY.pem`

Encrypting an RSA Key File

-   `openssl rsa -in KEY.pem -aes128 -out ENCRYPTED-KEY.pem``

Hash a RSA key

-   `sha1sum RSA-key1.pem`

### OpenSSL PKey Utility

Converting any Private Key file into text (RSA, DSA, or EC)

-   `openssl pkey -in KEY.pem -noout -text`

Extracting only Public Key as text from any Key file

-   `openssl pkey -in KEY.pem -noout -text_pub`

Extracting only Public Key in PEM format

-   `openssl pkey -in KEY.pem -pubout`

pkey expects a Private Key file. Public Key file can be read with -pubin

### Matching Private Keys to Certificates and CSRs

Check if RSA Key matches a CSR or Cert

-   Compare Modulus values to see if files match each other
    -   `openssl req -in CSR.pem -noout -modulus`
    -   `openssl x509 -in CERT.pem -noout -modulus`
    -   `openssl rsa -in KEY.pem -noout -modulus`

Check if EC Key matches a CSR or Cert

-   Compare Public Key values to see if files match each other
    -   `openssl req -in EC-CSR.pem -noout -pubkey`
    -   `openssl x509 -in EC-CERT.pem -noout -pubkey`
    -   `openssl ec -in EC-KEY.pem -pubout`

## Section 5

### Generating Certificate Signing Requests (CSRs) and Self-Signed Certificates

Generating CSRs:

-   Generate CSR with existing Private Key file
    -   `openssl req -new -key KEY.pem -out CSR.pem`
-   Generate CSR and new Private Key file
    -   `openssl req -new -newkey <alg:opt> -nodes -out CSR.pem`

Generating Self-Signed Certificates:

-   Generate Certificate with existing Private Key file
    -   `openssl req -x509 -key KEY.pem -out CERT.pem`
-   Generate Certificate and new Private Key file
    -   `openssl req -x509 -newkey <alg:opt> -nodes -out CERT.pem`

Self-signed certificates will have the same subject and issuer.

## Section 6

### Extracting Specific Information From Certificates and CSRs

Viewing x509 Certificate as human readable Text

-   `openssl x509  -in CERT.pem  -noout -text`

Viewing Certificate Signing Request (CSR) contents as Text:

-   `openssl req  -in CSR.pem  -noout -text`

Extract specific pieces of information from x509 Certificates

-   `openssl x509  -in CERT.pem  -noout  -dates`
-   `openssl x509  -in CERT.pem  -noout  -issuer –subject`

Other items you can extract:

```
-modulus -pubkey -ocsp_uri -ocspid
-serial -startdate -enddate
```

## Section 7

To check if file is PEM format

-   `openssl x509 -in FILE`

To check if file is DER format

-   openssl x509 -in FILE -inform DER`

To check if file is PFX format

-   `openssl pkcs12 -in FILE -nodes`

To check, or convert, PEM or DER Key Files use openssl pkey instead of openssl x509 and same command arguments.

Convert PEM Certificate file to DER

-   `openssl x509 -in CERT.pem -outform DER -out CERT.der`

Convert DER Certificate file to PEM

-   `openssl x509 -in CERT.der -inform der -out CERT.pem`

Convert PEM Certificate(s) to PFX

-   `openssl pkcs12 -in CERTS.pem -nokeys -export -out CERTS.pfx`

To include a key in PFX file use -inkey KEY.pem instead of -nokeys.

To extract everything within a PFX file as a PEM file:

-   `openssl pkcs12 -in FILE.pfx -out EVERYTHING.pem -nodes`

To extract only the Private Key from a PFX file as PEM:

-   `openssl pkcs12 -in FILE.pfx -out KEY.pem -nodes -nocerts`

PFX files can contain Certificate(s), or Certificate(s) + one matching Key

-   `-clcerts` - extract only end-entity certificate (client certificate)
-   `-cacerts` - extract all but end-entity certificate
-   `-nokeys` - extract only certficiates
