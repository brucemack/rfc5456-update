**Proposal to Augment RFC5456 (IAX2) with a New Authentication Method**

The purpose of this proposal is to define a new authentication method for use in the IAX2 protocol
defined by [RFC5456](https://datatracker.ietf.org/doc/html/rfc5456). 

# 1. Background/Informative 

## 1.1 Goals

The Inter-Asterisk eXchange protocol (IAX2) is used to implement digital telephony
over Internet Protocol (IP) networks. The IAX2 protocol provides security mechanisms
at a few points in the call initiation workflow. These mechanisms generally follow a
"challenge response" handshake. As currently defined, implementers can choose between 
an MD5 or a Rivest–Shamir–Adleman (RSA) challenge format. While these mechanisms
provide good protection from 
casual attackers, both have demonstrated cryptographic weaknesses. The goal of this 
proposal is to expand the list of authentication methods to include the Edwards-Curve
Digital Signature Algorithm which is a well-established digital signature algorithm 
described in [RFC 8032](https://datatracker.ietf.org/doc/html/rfc8032).

## 1.2 Challenge/Response Flows

There are two points in the IAX2 protocol flow where this issue is relevant:

1. The (optional) peer registration process is used to allow an IAX2 peer to advertise
its presence on the network and to provide the information needed to be reachable by
other peers. This process involves a handshake with a central directory service.
This handshake is defined in section 6.1.2 of the IAX2 RFC. Given the obvious security
implications of a central directory, the handshake allows the registration service
to challenge the identity of a potential peer. 
2. When a call is being established between two peers it will generally be necessary
for the called peer to authenticate the calling peer. This handshake is described 
section 6.2.6 of the IAX2 RFC. Given the obvious security
implications of a peer-to-peer network, the handshake allows one peer to 
challenge the identity of another.

In both cases the idea is the same: the server/peer that is the recipient of a connection
can challenge the originator of the connection to determine whether it possesses a required
secret. To avoid exposing the secret during the handshake, a cryptographically-robust digest 
is provided by the connection originator as a proxy for the actual secret.

## 1.3 Existing Digest Formats Used in IAX2

The existing protocol allows two digest methods for this challenge/response mechanisms. Some
background information is provided here to support the proposal. However, it should be noted 
that the existing RFC is somewhat vague when it comes to the description of the nature/format
of the challenge token or digest formation process.

### 1.3.1 The MD5 Digest 

The MD5 message digest is defined in RFC 1321.

In this mechanism a challenge string is provided to a connection originator by the connection
recipient using the CHALLENGE information element (0x0f). The RFC does not define the format/length 
of this challenge other than to say that the challenge is in UTF-8 encoded format. Per the RFC (section 
8.6.15) the resulting digest "is computed by taking the MD5 digest of the challenge string and the 
password string." This digest is returned in the MD5 RESULT (0x10) information element. Again, the RFC is 
silent on the exact format of this element, other than to say that it is "UTF-8 encoded." 

Although not explicitly stated, the author has observed that the MD5 RESULT element contains the 
UTF-8 encoded representation of the 32-character hexadecimal representation of the 16-byte (128 bit)
standard MD5 digest.

What is also unstated is the method of joining the original challenge string and password that forms the 
input of the MD5 digest. From experimentation is appears that the current implementations concatenate
the challenge and password directly without delimiter.

## 1.3.2 The RSA Digest

The RFC provides a bit more specificity in this case. The UTF-8 encoded challenge string from the CHALLENGE
information element (0x0f) provided by the connection recipient is SHA1-hashed (RFC3174) and the
result is RSA-signed using a private key by the connection originator. The PKCS #1 v2.0 method is used to
address padding requirements. Finally, the RSA digest is validated by the connector recipient using a public key.

What is not explicitly stated in the RFC is that the result of the RSA signing computed by the connection
originator is Base-64 encoded (RFC4648) before being sent back to the recipient for authentication.

## 1.4 New Format: Edwards-Curve Digital Signature Algorithm

The proposal is to add a third digest option. Specifically the ED25519 variant of the
Edwards-Curve Digital Signature Algorithm described in RFC8032. Using this method the recipient of
a connection would generate a challenge string, this challenge would be signed by the connection originator
using a private key, and the resulting digest would be validated by the connection recipient using a
public key. The important parameters of this digest:

* The challenge in the ED25519 methodology is 128 bits. This challenge is generally joined to a 128 bit
public key to form the input of the digest calculation.
* The digest that is formed by the ED25519 methodology is 512 bits.

These sizes are relevant because of the 255-byte size limit for Information Elements in the IAX2 protocol.
UTF-8 encoded hex representations of the challenge and digests will both fit.

Although several sections of the RFC need to be changed, the proposal is actually quite simple: **take
anything that is currently possible using the RSA digest method and allow the ED25519 digest method as well.**

# 2. Proposed Additions

## Section 6.1.1 - Registration Overview

Section 6.1.1 should be enhanced to include mention of a new method:

        IAX allows user authentication via multiple methods.  MD5 Message-
        Digest authentication [RFC1321] uses an MD5 sum arrangement, but
        still requires that both ends have plaintext access to the secret.
        (See Section 8.6.15.)  Rivest, Shamir, and Adleman's (RSA) algorithm
        [RFC3447] and the Edwards-Curve Digital Signature Algorithm (ED25519) 
        [RFC8032] both allow unidirectional secret knowledge through public/
        private key pairs.  The IAX Private keys SHOULD always be Triple Data
        Encryption Standard (3DES) encrypted [RFC1851].  (See Section 8.6.16.)

## Section 6.1.2 - New Information Element for REGREQ

The table describing the permitted IEs in the REGREQ message should be expanded
to mention the new result type:

        +----------------+----------------+-------------+-------------+
        | IE             | Section        | Status      | Comments    |
        +----------------+----------------+-------------+-------------+
        | Username       | Section 8.6.6  | Required    |             |
        |                |                |             |             |
        | MD5 Result     | Section 8.6.15 | Conditional | per REGAUTH |
        |                |                |             |             |
        | RSA Result     | Section 8.6.16 | Conditional | per REGAUTH |
        |                |                |             |             |
        | ED25519 Result | Section 8.6.xx | Conditional | per REGAUTH |
        |                |                |             |             |
        | Refresh        | Section 8.6.18 | Optional    |             |
        +----------------+----------------+-------------+-------------+

## Section 6.1.3 - New Option for REGAUTH

The table in section 6.1.3 should be elaborated to include the new possible format
for the challenge element:

        +--------------+----------------+-------------+-------------------------+
        | IE           | Section        | Status      | Comments                |
        +--------------+----------------+-------------+-------------------------+
        | Username     | Section 8.6.6  | Required    |                         |
        |              |                |             |                         |
        | Auth Methods | Section 8.6.13 | Required    |                         |
        |              |                |             |                         |
        | Challenge    | Section 8.6.14 | Conditional | If RSA, MD5, or ED25519 |
        +--------------+----------------+-------------+-------------------------+

## Section 6.1.6 - New Information Element for REGREL 

The table describing the permitted IEs in the REGREL message should be expanded
to mention the new result type:

        +----------+----------------+-------------+----------------------------+
        | IE       | Section        | Status      | Comments                   |
        +----------+----------------+-------------+----------------------------+
        | Username | Section 8.6.6  | Required    |                            |
        |          |                |             |                            |
        | MD5      | Section 8.6.15 | Conditional | MD5, RSA or ED25519 Result |
        | Result   |                |             | is required                |
        |          |                |             |                            |
        | RSA      | Section 8.6.16 | Conditional |                            |
        | Result   |                |             |                            |
        |          |                |             |                            |
        | ED25519  | Section 8.6.xx | Conditional |                            |
        | Result   |                |             |                            |
        |          |                |             |                            |
        | Cause    | Section 8.6.21 | Optional    |                            |
        |          |                |             |                            |
        | Cause    | Section 8.6.33 | Optional    |                            |
        | Code     |                |             |                            |
        +----------+----------------+-------------+----------------------------+

## Section 6.1.6 - New Information Element for NEW

The table describing the permitted IEs in the NEW message should be expanded
to mention the new result type:

        +------------------+----------------+-------------+---------------------+
        | IE               | Section        | Status      | Comments            |
        +------------------+----------------+-------------+---------------------+
        | Version          | Section 8.6.10 | Required    |                     |
        |                  |                |             |                     |
        | Called           | Section 8.6.1  | Required    |                     |
        | Number           |                |             |                     |
        |                  |                |             |                     |
        | Auto Answer      | Section 8.6.24 | Optional    |                     |
        |                  |                |             |                     |
        | Codecs Prefs     | Section 8.6.35 | Required    |                     |
        |                  |                |             |                     |
        | Calling          | Section 8.6.29 | Required    |                     |
        | Presentation     |                |             |                     |
        |                  |                |             |                     |
        | Calling          | Section 8.6.2  | Optional    |                     |
        | Number           |                |             |                     |
        |                  |                |             |                     |
        | Calling TON      | Section 8.6.30 | Required    |                     |
        |                  |                |             |                     |
        | Calling TNS      | Section 8.6.31 | Required    |                     |
        |                  |                |             |                     |
        | Calling Name     | Section 8.6.4  | Optional    |                     |
        |                  |                |             |                     |
        | ANI              | Section 8.6.3  | Optional    |                     |
        |                  |                |             |                     |
        | Language         | Section 8.6.9  | Optional    |                     |
        |                  |                |             |                     |
        | DNID             | Section 8.6.12 | Optional    |                     |
        |                  |                |             |                     |
        | Called           | Section 8.6.5  | Conditional | 'Default' assumed   |
        | Context          |                |             | if IE excluded      |
        |                  |                |             |                     |
        | Username         | Section 8.6.6  | Optional    |                     |
        |                  |                |             |                     |
        | RSA Result       | Section 8.6.16 | Conditional | If challenged with  |
        |                  |                |             | RSA                 |
        |                  |                |             |                     |
        | MD5 Result       | Section 8.6.15 | Conditional | If challenged with  |
        |                  |                |             | MD5                 |
        |                  |                |             |                     |
        | ED25519 Result   | Section 8.6.xx | Conditional | If challenged with  |
        |                  |                |             | ED25519             |
        |                  |                |             |                     |
        | Format           | Section 8.6.8  | Required    |                     |
        |                  |                |             |                     |
        | Capability       | Section 8.6.7  | Conditional |                     |
        |                  |                |             |                     |
        | ADSICPE          | Section 8.6.11 | Optional    |                     |
        |                  |                |             |                     |
        | Date Time        | Section 8.6.28 | Optional    | Suggested           |
        |                  |                |             |                     |
        | Encryption       | Section 8.6.34 | Optional    |                     |
        |                  |                |             |                     |
        | OSP Token        | Section 8.6.42 | Optional    |                     |
        +------------------+----------------+-------------+---------------------+

## Section 6.2.6 - New Information Element for AUTHREP

The table describing the permitted IEs in the AUTHREP message should be expanded
to mention the new result type:

        +----------------+----------------+-------------+------------+
        | IE             | Section        | Status      | Comments   |
        +----------------+----------------+-------------+------------+
        | RSA Result     | Section 8.6.16 | Conditional | If RSA     |
        |                |                |             |            |
        | MD5 Result     | Section 8.6.15 | Conditional | If MD5     |
        |                |                |             |            |
        | ED25519 Result | Section 8.6.xx | Conditional | If ED25519 |
        +----------------+----------------+-------------+------------+

## Section 6.2.7 - Clarification of AUTHREQ Message

The text that describes the challenge methods should be expanded:

        The AUTHREQ message is sent in response to a NEW message if
        authentication is required for the call to be accepted.  It MUST
        include the 'authentication methods' and 'username' IEs, and the
        'challenge' IE if MD5, RSA, or ED25519 authentication is specified.

## Section 8.6 - Addition of a New Information Element to Support ED25519

Here the lengthy table that lists the supported Information Elements should enhanced with a
new row, making use of one of the reserved codepoints. We propose the use of 0x20.  

* The HEX column should contain: 0x20
* The NAME column should contain: "ED25519 RESULT"
* The DESCRIPTION column should contain: "ES25519 challenge result"

## Section 8.6.13 - Expansion of the AUTHMETHODS

Section 8.6.13 of the RFC defines the AUTHMETODS information element. This element is used to indicate
the authentication methods that a peer accepts. The proposal is to extend the list. Specifically, the 
table in section 8.6.13 should be extended to add a new codepoint for the ED25519 digest method. The
new table should look like this:

        +--------+--------------------------+
        | METHOD | DESCRIPTION              |
        +--------+--------------------------+
        | 0x0001 | Reserved (was Plaintext) |
        |        |                          |
        | 0x0002 | MD5                      |
        |        |                          |
        | 0x0004 | RSA                      |
        |        |                          |
        | 0x0008 | ED25519                  |
        +--------+--------------------------+

## Section 8.6.14 - Expansion of CHALLENGE Documentation

This section should be expanded to include the new method:

        The purpose of the CHALLENGE information element is to offer the MD5,
        RSA, or ED25519 challenge to be used for authentication.  It carries the
        actual UTF-8-encoded challenge data.

Furthermore, an elaboration:

        The challenge size for the ED25519 authentication method MUST be challenger-defined
        128-bit (16-byte value), represented as 32 character hexadecimal value, encoded in 
        UTF-8 format.

## New Section 8.6.xx - Addition of ED25519 RESULT Information Element

The following text is proposed. This largely parallel the RSA RESULT
element:

        The purpose of the ED25519 RESULT information element is to offer an ED25519
        response to an authentication CHALLENGE.  It carries the UTF-8-encoded challenge 
        result.  The result is computed as follows: converting the 32-hex character 
        ED25519 challenge string to a 16-byte value, concatenate the the 16-bytes into a 
        single 128-bit value, and sign the ED25519 using the private key. The ED25519 keys 
        are stored locally.

        Upon receiving an ED25519 RESULT information element, its value must be
        verified with the sender's public key to match the original challenge string.

        The ES25519 RESULT information element MAY be sent with IAX AUTHREP and
        REGREQ messages if an AUTHREQ or REGAUTH and appropriate CHALLENGE
        have been received.  This information element MUST NOT be sent except
        in response to a CHALLENGE.

                            1
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |      0x20     |  Data Length  |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |                               |
        : UTF-8-encoded ED25519 Result  :
        |                               |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    
    ## Section 10 - Elaboration of Security Considerations

    This section provides some general context around the security of IAX2 channels.
    Everything that is described regarding MD5 and RSA applies equally (if not more)
    to the ED25519 encryption technology.

    The proposal is to replace: "MD5 and RSA" with "MD5, RSA, and ED25519 throughout
    this section.

    