---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Proposal to Augment IAX2 Protocol with a new Media Format"
abbrev: "IAX2 New Medial Format"
category: info

docname: draft-todo-yourname-protocol-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Bruce MacKinnon
    organization: Independent
    email: bruce@mackinnon.com

normative:

informative:

...

--- abstract

Inter-Asterisk eXchange (IAX2) is an application-layer control
and media protocol used to conduct multimedia sessions (usually VOIP audio calls)
over Internet Protocol (IP) networks. The protocol definition enumerates a
specific set of media formats that can be used by session participants. This document
proposes a new media format to support 16,000 samples/second audio.

--- middle

# Introduction

The Inter-Asterisk eXchange protocol (IAX2) is used to implement digital telephony
over Internet Protocol (IP) networks. One important flexibility of this protocol
is the support for a variety of encoding formats used to transfer digital audio 
across the network. For completeness, the protocol also provides support for some 
non-audio formats like JPEG. The RFC uses the term "Media Format" to describe the 
specific method of encoding audio/visual content within the protocol. 

Section 8.7 of {{!RFC5456}} provides a list of allowable media formats. This list is 
repeated in the IANA Registry for the IAX protocol. Importantly, these two documents
assign a 32-bit codepoint to each allowable media format. This codepoint is used within
the IAX2 protocol messages to allow nodes to negotiate the audio format that will be used
during the subsequent telephony session.

The purpose of this discussion is two-fold:
* Clarify the meaning of the **existing** media format "16-bit linear little-endian"
(codepoint 0x00000040) by defining the audio sampling rate that should be used for encoding
audio in this format.
* Add a new media format "16-bit linear (PCM) little-endian 16 kHz sampling rate" (codepoint 0x00008000).

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

For IAX2 Security Considerations, see section 10 of {{!RFC5456}}.

# IANA Considerations

IANA is requested to make two updates to the IAX Media Formats registry.

## Clarification of Existing Media Format

The "Media Format Values" table in section 8.7 of the RFC should be changed in the following ways:

* The title at the top of column 3 of the table should be changed from "LENGTH CALCULATIONS" to
"ENCODING NOTES." This will better represent the contents of this column which varies across formats
and doesn't always provide the ability to compute the encoded length.
* The "DESCRIPTION" column for SUBCLASS 0x00000040 should be changed to "16-bit linear (PCM) little-endian 8 kHz" to
eliminate ambiguity.
* The "ENCODING NOTES" column for SUBCLASS 0x00000040 should contain "2 bytes per sample, 160 samples per chunk."

## New/Proposed Media Format

The "Media Format Values" table in section 8.7 of the RFC should be augmented to include a new row.
The row should be added in numerical order after AMR.

* The "SUBCLASS" column should contain the value TBD1. This specific value is being 
proposed in order to maintain consistency with the existing implementations.
* The "DESCRIPTION" column should contain "16-bit linear (PCM) little-endian 16 kHz."
* The "ENCODING NOTES" column should contain "2 bytes per sample, 320 samples per chunk."

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
