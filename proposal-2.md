**Proposal to Augment RFC5456 (IAX2) with a New Authentication Method**

The purpose of this proposal is to define a new authentication method for use in the IAX2 protocol
defined by [RFC5456](https://datatracker.ietf.org/doc/html/rfc5456). 

# 1. Background/Informative 

## 1.1 Goals

The Inter-Asterisk eXchange protocol (IAX2) is used to implement digital telephony
over Internet Protocol (IP) networks. The IAX2 protocol provides security mechanisms
at a few points in the call initiation workflow. These mechanisms generally follow a
"challenge response" protocol. As currently defined, implementers can choose between 
an MD5 or a Rivest–Shamir–Adleman (RSA) challenge format. While these mechanisms
provide good protection from 
casual attackers, both have demonstrated cryptographic weaknesses. The goal of this 
proposal us to expand the list of authentication methods to include the Edwards-Curve
Digital Signature Algorithm which is a well-established digital signature algorithm 
described in [RFC 8032](https://datatracker.ietf.org/doc/html/rfc8032).

## 1.2 Challenge/Response Flows

There are two points in the IAX2 protocol flow where this issue is relevant:

1. The (optional) peer registration process is used to allow an IAX2 peer to advertise
its presence on the network and to provide the information needed to be reachable by
other peers. This process involves a handshake with a central directory service.
This handshake is defined in section 6.1.2 of the IAX2 RFC. Given the obvious security
implications of a central directory, the handshake allows the registration service
to challenge the identity a potential peer. 
2. When a call is being established between two peers it will generally be necessary
for the called peer to authenticate the calling peer. This handshake is described 
section 6.2.6 of the IAX2 RFC. Given the obvious security
implications of a peer-to-peer network, the handshake allows one peer to 
challenge the identity of another.

In both cases the idea is the same: the server/peer that is the recipient of a connection
can challenge the originator of the connection to determine whether it possesses a required
secret. To avoid exposing the secret during the handshake, a cryptographically-robust digest 
is provided by the caller as a proxy for the actual secret.

## 1.3 Existing Formats

The existing protocol allows two signature methods for this challenge/response mechanisms. Some
background information is provided here to support the proposal. However, it should be noted 
that very little information about the exact nature/format of the challenge token or digest formation
process is provided in the existing RFC document.

### 1.3.1 The MD5 Digest 

The MD5 message digest is defined in RFC 1321.

When using this mechanism a challenge string is provided to a connection originator by the connection
recipient using the CHALLENGE information element (0x0f). The RFC does not define the format/length 
of this challenge, other than to say that the challenge is in UTF-8 encoded format. Per the RFC (section 
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

* The challenge in the ED25519 methodology is 128 bits. This is generally joined to a 128 bit
public key to form the input of the digest calculation.
* The digest that is formed by the ED25519 methodology is 512 bits.

# 2. Proposed Additions

## 2.1 An Expansion of the AUTHMETHODS

