Copyright (c) 2013 Michael Mol <mikemol@gmail.com>

(insert BSD 3-clause license here)

msgpass
=======

msgpass: A message format targeted at secure(ish) asynchronous emails.

Preamble
--------

Email is inherently insecure. Tools such as GPG and S?MIME are nice, but they
don't secure certain pieces of metadata. Namely, they don't protect the source
address and the subject. Further, the transport process of email results in
routing information added to the message, baking additional information
subject to analysis.

With the existence of malicious nation-states and other entities which perform
wholesale capture and analysis of network traffic, it becomes critical for
purposes of individual freedom to maintain the ability to communicate despite
these entities, and it becomes further critical for these communications to be
as private as possible.

Finally, with the recent shuttering of privacy-minded hosted email services,
and with the revelation of fundamental flaws in the privacy and security of
email, individuals are left with a glaring hole in the suite of standard
communication habits; while it's easy to use protocols such as TLS and OTR to
establish secure synchronous communication, there remain no easy and common
means to have a more critical service: asynchronous communication.

msgpass is intended to help fill this gap.

Introduction
------------

msgpass tries to tackle the security and privacy aspects in a few basic ways.

1. It uses existing PGP/GPG tools as a means to both protect message
content and to identify source and destination identities.

2. It does not expose source identity information in the clear; only the
destination identity is shown.

3. In order to improve on delivery guarantees, it allows for multiple return
paths to be specified in the encrypted portion of message headers.

4. In order to resist attacks based on known (or guessed, such as is used in
the CRIME and BEAST attacks against SSL/TLS) cyphertext, nonces are specified
at the beginning of the message body, and the message version number is kept
in the clear.

5. In order to ease uptake, the system is explicitly intended to be able to
work with existing content distribution systems such as pastebins and NNTP
servers.

Message Format
==============

The explicit encoding is not yet known. The encoded content, however, is described below. The body immediately follows the header.

Basic types
-----------

Several types are reused frequently, such as Blobs and Counts. Blobs are
themselves compound types, but are used so frequently that I feel they
warrant up-front explanation.

First, the Count data type. The purpose of this type is, simply, to provide
a count of values. In the message format, this is often used as a prelude to
an array of like objects. 

A Count type is always unsigned, and will have the specified number of bits
where it's referenced elsewhere in the spec.

Next, the Blob type. The Blob type is intended for data packing. It consists
of a Count value followed by as many octets as are specified by that Count.
Where the Blob type is referenced, the number of bits for the corresponding
Count field will also be specified.

Finally, the String type. Strings are Blob types where the content of the
blob is specified as using UTF-8 as its encoding.

Unencrypted Header
------------------

<table>
  <tr>
    <th>Field</th><th>Size</th><th>Description</th>
  </tr>
  <tr>
    <td>Magic Number</td>
    <td>32 bits</td>
    <td>The magic number useful for identifying this file type from others.</td>
  </tr>
  <tr>
    <td>Version Number</td>
    <td>16 bits</td>
    <td>Version of the message format. This spec describes version 1.</td>
  </tr>
  <tr>
    <td>Recipient GPG Key ID</td>
    <td>32 bits</td>
    <td>The ID of the public key of the recipient.</td>
  </tr>
</table>

Everything following this header is encrypted using the public key associated
with the Recipient GPG Key ID.

Encrypted Body
--------------

<table>
  <tr>
    <th>Field</th><th>Size</th><th>Description</th>
  </tr>
  <tr>
    <td>Nonce</td>
    <td>64 bits</td>
    <td>High-entropy random data</td>
  </tr>
  <tr>
    <td>Message ID</td>
    <td>128 bits</td>
    <td>Unique identifier for message</td>
  </tr>
  <tr>
    <td>Sender Public GPG Key</td>
    <td>Variable. Blob with 16-bit count</td>
    <td>GPG Public Key of sender. See String type.</td>
  </tr>
  <tr>
    <td>Return Paths</td>
    <td>Variable. Blob with 32-bit count.</td>
    <td>Depositories for replies to this message</td>
  </tr>
  <tr>
    <td>Payload</td>
    <td>Variable. Blob with 64-bit count.</td>
    <td>Content of message body</td>
  </tr>
  <tr>
    <td>Signature</td>
    <td>Variable</td>
    <td>GPG signature, using sender's key, of entire body of message</td>
  </tr>
</table>

Return Paths
------------

<table>
  <tr>
    <th>Field</th><th>Size</th><th>Description</th>
  </tr>
  <tr>
    <td>Count</td>
    <td>8 bits</td>
    <td>How many return paths are listed. See Count type.</td>
  </tr>
  <tr>
    <td>Path 1</td>
    <td>Variable. Blob with 32-bit count.</td>
    <td>Return Path</td>
  </tr>
  <tr>
    <td>Path 2</td>
    <td>Variable. Blob with 32-bit count.</td>
    <td>Return Path</td>
  </tr>
  <tr>
    <td>Path N...</td>
    <td>Variable. Blob with 32-bit count.</td>
    <td>Return Path</td>
  </tr>
</table>

TODO: Specify return paths more clearly. Include sample path types.

Payload
_______

<table>
  <tr>
    <th>Field</th><th>Size</th><th>Description</th>
  </tr>
  <tr>
    <td>Part count</td>
    <td>8 bits</td>
    <td>How many payload parts to look for</td>
  </tr>
  <tr>
    <td>Part 1</td>
    <td>Variable. Blob with 64-bit count.</td>
    <td>Payload part.</td>
  </tr>
  <tr>
    <td>Part 2</td>
    <td>Variable. Blob with 64-bit count.</td>
    <td>Payload part.</td>
  </tr>
  <tr>
    <td>Part N...</td>
    <td>Variable. Blob with 64-bit count./td>
    <td>Further payload parts as necessary</td>
  </tr>
</table>

Payload Part
____________

<table>
  <tr>
    <th>Field</th><th>Size</th><th>Description</th>
  </tr>
  <tr>
    <td>Payload Part ID</td>
    <td>16 bits</td>
    <td>ID of payload part</td>
  </tr>
  <tr>
    <td>Payload Part Name</td>
    <td>Variable. String with 16-bit count.</td>
    <td>Name of Payload</td>
  <tr>
  <tr>
    <td>Payload Part MIME Type</td>
    <td>Variable. String with 8-bit count.</td>
    <td>MIME type of payload body</td>
  </tr>
  <tr>
    <td>Payload Part Body</td>
    <td>Variable. Blob with 64-bit count.</td>
    <td>Body of the Payload Part</td>
  </tr>
</table>

Field Descriptions
-------------------

__Magic Number__

In order to identify the data on the wire or on disk, a magic number is used.
This is fairly common practice. The magic number for this data is yet to be
determined.

This field is not encrypted.

__Version Number__

In order to allow for changes in the format on the wire, a version number is
necessary. Because the version number defines how to parse the message, it
must come early in the byte stream. Because the version number is unlikely to
change often, I don't think it likely to reveal much identifying information.
Further, because the version number isn't likely to change much, it represents
a piece ofknown data, and thus, were it included near the beginning of the
encrypted portion of the stream, would improve the efficacy of
known-cyphertext attacks against the message body.

In consideration all of these factors, the version number is kept in the
clear.

This field is not encrypted.

__Recipient GPG Key ID__

The public key ID of the intended recipient. This is the only intended
mechanism by which the message's destination should be known. Knowledge of the
recipient's GPG key should inform the sender of the appropriate cyphers to
use, etc. The message body is encrypted using the public key identified by
this key ID. Only the message recipient should be able to decode the message.

This field is not encrypted. All subsequent fields are encrypted as a
monolithic block.

__Nonce__

Random, high-entropy data. This data has no semantic meaning. Its sole purpose
is to help scramble the state of the cypher engine, and to provide uniqueness
to multiple copies of the same message sent to a given recipient using varying
return paths. As such, if a message is placed in multiple depositories, the
nonce *should* be unique for each placement.

__Message ID__

Value uniquely identifying the message. This value should be able to be
compared with the same field from another message in order to detect duplicate
messages without any further decryption of the message body.

If the same message is placed in multiple depositories, they MUST have common
message IDs, regardless of whether or not they have distinct nonces. There is
no other defined way to detect identical messages.

__Sender Public GPG Key__

This is the sender's public key. Or, at least, the public key used to sign the
message. The recipient may, of course, use whatever means they wish to verify
the key; without out some out-of-band means to verify the key, there's no way
for the recipient to validate the authenticity of the message short of only
distributing their own public key to specific parties.

__Return Paths__

In the event the recipient wishes to reply, this field lists recommended
depositories which the sender may check for messages. Examples of depositories
might include:

* Pastebins
* Newsgroups
* File servers
* Forums

A return path might also indicate a different public key which should be used
encrypt messages sent via that path. As a consequence, it might also function
as a CC/BCC target list for replies, resulting in a default of a "reply-to-
all" semantic. It might also serve as a mechanism by which the original
sender may seek to encode further information which might trigger decisions
upon receipt of reply.

Before this spec is finalized, a proper syntax should be defined.

__Payload__

The message content. Consists of multiple Payload Parts.

__Payload Part__

Each Payload Part consists of an ID, a Payload Part Name, a MIME type for the
Payload Part Body, and the Payload Part Body itself, which contains the actual
data.

__Payload Part ID__

Identifies the Payload Part, so that other Payload Parts which may wish to
transclude it may do so. An example of this principle is seen in HTML emails
with inline-attached images.

Payload Parts with the same ID, but different MIME types, should be considered
different means of representing the same content. In this way, plaintext, HTML
and audio forms of the same message might be simultaneously offered.

__Payload Part Name__

A human-readable description of the Payload Part. If Payload Parts are broken
out into separate files, this would be considered the part's filename.

__Payload Part MIME Type__

A hint as to how the content of the Payload Part Body should be used.

__Payload Part Body__

The raw meat of a Payload Part.

__Signature__

GPG signature of entire encrypted body of message, using the sender's private GPG key.


