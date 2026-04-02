**Proposal to Augment RFC5456 (IAX2) with a new Media Format**

The purpose of this proposal is to formalize the use of a new media format which is not defined 
in the current version of [RFC5456](https://datatracker.ietf.org/doc/html/rfc5456). Some
clarification of an **existing** media format (16-bit linear, 8kHz sampled audio) is also proposed
for continuity.

# 1. Background/Informative 

## 1.1 Goals

The Inter-Asterisk eXchange protocol (IAX2) is used to implement digital telephony
over Internet Protocol (IP) networks. One important flexibility of this protocol
is the support for a variety of encoding formats used to transfer digital audio 
across the network. For completeness, the protocol also provides support for some 
non-audio formats like JPEG. The RFC uses the term "Media Format" to describe the 
specific method of encoding audio/visual content within the protocol. 

Section 8.7 of the RFC provides a list of allowable media formats. This list is 
repeated in the IANA Registry for the IAX protocol. Importantly, these two documents
assign a 32-bit codepoint to each allowable media format. This codepoint is used within
the IAX2 protocol messages to allow nodes to negotiate the audio format that will be used
during the subsequent telephony session.

The purpose of this discussion is two-fold:
* Clarify the meaning of the **existing** media format "16-bit linear little-endian"
(codepoint 0x00000040) by defining the audio sampling rate that should be used for encoding
audio in this format.
* Add a new media format "16-bit linear (PCM) little-endian 16 kHz sampling rate" (codepoint 0x00008000).

This discussion has been made more complicated by a few factors:
1. The existing RFC document does not define a sampling rate for the "16-bit linear little-endian"
format. In fact, there is little explicit mention of sampling rates throughout the list of media 
formats in section 8.7, although this is usually not an issue since most of the formats are 
described in other related standards. For example: the G.711 mu-law standards document makes
it very clear that audio encoded in this format should be sampled at 8 kHz. As it turns out,
the existing implementations of the IAX2 protocol that the author is aware of have made the 
assumption that audio encoded in the "16-bit linear little-endian" format is sampled at 8 kHz.
2. Although the existing RFC document does not define a 16 kHz variant of the 16-bit linear
media format, the existing implementations of the IAX2 protocol that the author is aware of
have defined a de-facto standard format for this encoding using an **apparently unused** codepoint
of 0x00008000. Although the use of an unregistered format is not desirable, at least the
developers didn't create a conflict with any documented format.
3. To make things even more complicated, it appears that the existing implementations of the
8 kHz variant of the 16-bit linear format (the documented one) contain a formatting error. Audio
samples are encoded using big-endian (network) byte order. **We are not addressing this issue in
this proposal.** The RFC document was clear on the byte-ordering convention and the erroneous 
implementations will be handled separately. 
4. The IAX2 RFC contains an oblique mention to an Information Element (0x29) called SAMPLINGRATE
in section 8.6.32 that suggests a method of eliminating the ambiguity of the existing media
format. However, the
RFC makes no mention of where/how this Information Element should be used. It is likely that 
this issue never came up because the sampling rate is **implicit** in all of the audio-oriented
media formats that are governed by formal standards (i.e. all of the formats
that start with "G" or something similar).

## 1.2 Background on Audio Sampling Rate

One of the key parameters of a digital audio encoding format is the sampling rate. This rate
defines the frequency at which an analog audio waveform is observed and converted into 
digital form. This rate is also used on the other side of the network to convert digital
samples back to an analog form. In order for a digital audio system to be useful there 
must be a clear agreement on the sampling rate assumed by both sides of the session. All of the 
existing audio media formats supported by the IAX2 protocol make some assumption about the 
audio sampling rate.

By way of background, digital audio systems with higher sampling rates generally provide
higher audio fidelity. This is the motivation for proposing a 16 kHz audio format in the IAX2
protocol.

## 1.3 Background on Audio Quantizing Format

The terms "linear" and "pulse-coded modulation" (PCM) are used below. These terms are generally
synonymous and refer to the method of mapping observed analog voltage levels to 
corresponding digital codewords. A PCM encoding system is called "linear" because the analog 
voltage levels are quantized to corresponding digital codewords by a linear function. For example, 
voltage that range from -0.5 volts to +0.5 volts would be mapped onto digital codewords 
that range from -32,768 to 32,767 using a linear factor of ~65,534.

The linear mapping referenced in this proposal is in contrast to several of the other non-linear media formats
supported. G.711 mu-law (codepoint  0x00000004), for example, uses a logarithmic mapping that gives 
more resolution to lower signal values. The G.722 format (codepoint 0x00001000) uses an adaptive
mapping that assigns more/less resolution depending on the evolving shape of the analog signal being
quantized. 

Notice that the preceding paragraph references two audio formats (G.711 and G.722) using designations
assigned by a formal standardization process. It is probably not a coincidence that the audio 
formats that are the subject of this proposal are the only ones that do not reference external standards.
The description "16-bit linear little-endian" leaves some key parameters to the implementer's 
interpretation. To name a few:
* The audio sampling rate to be used.
* The audio block size to be used.
* The convention around signedness to be used (i.e. all positive, signed 2's compliment, etc.)

# 2. Proposed Additions

## 2.1 Clarification of Existing Media Format

The "Media Format Values" table in section 8.7 of the RFC should be changed in the following ways:

* The title at the top of column 3 of the table should be changed from "LENGTH CALCULATIONS" to
"ENCODING NOTES." This will better represent the contents of this column which varies across formats
and doesn't always provide the ability to compute the encoded length.
* The "DESCRIPTION" column for SUBCLASS 0x00000040 should be changed to "16-bit linear (PCM) little-endian 8 kHz" to
eliminate ambiguity.
* The "ENCODING NOTES" column for SUBCLASS 0x00000040 should contain "2 bytes per sample, 160 samples per chunk."

## 2.2 New/Proposed Media Format

The "Media Format Values" table in section 8.7 of the RFC should be augmented to include a new row.
The row should be added in numerical order after AMR.

* The "SUBCLASS" column should contain the value 0x00008000. This specific value is being 
proposed in order to maintain consistency with the existing implementations.
* The "DESCRIPTION" column should contain "16-bit linear (PCM) little-endian 16 kHz."
* The "ENCODING NOTES" column should contain "2 bytes per sample, 320 samples per chunk."

## 2.3 Final Table for Section 8.7 (Proposed)

The proposed table in section 8.7 should look like this:

        +------------+-----------------------+------------------------------------+
        | SUBCLASS   | DESCRIPTION           | ENCODING NOTES                     |
        +------------+-----------------------+------------------------------------+
        | 0x00000001 | G.723.1               | 4-, 20-, and 24-byte frames of 240 |
        |            |                       | samples                            |
        |            |                       |                                    |
        | 0x00000002 | GSM Full Rate         | 33-byte chunks of 160 samples or   |
        |            |                       | 65-byte chunks of 320 samples      |
        |            |                       |                                    |
        | 0x00000004 | G.711 mu-law          | 1 byte per sample                  |
        |            |                       |                                    |
        | 0x00000008 | G.711 a-law           | 1 byte per sample                  |
        |            |                       |                                    |
        | 0x00000010 | G.726                 |                                    |
        |            |                       |                                    |
        | 0x00000020 | IMA ADPCM             | 1 byte per 2 samples               |
        |            |                       |                                    |
        | 0x00000040 | 16-bit linear (PCM)   | 2 bytes per sample, 160 samples    |
        |            | little-endian 8 kHz   | per chunk                          |
        |            |                       |                                    |
        | 0x00000080 | LPC10                 | Variable size frame of 172 samples |
        |            |                       |                                    |
        | 0x00000100 | G.729                 | 20-byte chunks of 172 samples      |
        |            |                       |                                    |
        | 0x00000200 | Speex                 | Variable                           |
        |            |                       |                                    |
        | 0x00000400 | ILBC                  | 50 bytes per 240 samples           |
        |            |                       |                                    |
        | 0x00000800 | G.726 AAL2            |                                    |
        |            |                       |                                    |
        | 0x00001000 | G.722                 | 16 kHz ADPCM                       |
        |            |                       |                                    |
        | 0x00002000 | AMR                   | Variable                           |
        |            |                       |                                    |
        | 0x00008000 | 16-bit linear (PCM)   | 2 bytes per sample, 320 samples    |
        |            | little-endian 16 kHz  | per chunk                          |
        |            |                       |                                    |
        | 0x00010000 | JPEG                  |                                    |
        |            |                       |                                    |
        | 0x00020000 | PNG                   |                                    |
        |            |                       |                                    |
        | 0x00040000 | H.261                 |                                    |
        |            |                       |                                    |
        | 0x00080000 | H.263                 |                                    |
        |            |                       |                                    |
        | 0x00100000 | H.263p                |                                    |
        |            |                       |                                    |
        | 0x00200000 | H.264                 |                                    |
        +------------+-----------------------+------------------------------------+
