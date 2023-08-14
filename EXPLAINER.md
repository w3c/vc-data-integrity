# Verifiable Credentials Data Integrity Explainer

## Authors:

- Manu Sporny
- Dave Longley
- Dmitri Zagidulin
- Gregg Kellogg
- Markus Sabadello

## Participate
- [Issue tracker](https://github.com/w3c/vc-data-integrity/)
- [Discussion forum](https://lists.w3.org/Archives/Public/public-vc-wg/)

# Table of Contents

  * [Introduction](#introduction)
  * [Goals](#goals)
  * [Non-goals](#non-goals)
  * [User research](#user-research)
  * [What We're Trying to Accomplish](#what-were-trying-to-accomplish)
  * [How it Works](#how-it-works)
  * [Key scenarios](#key-scenarios)
    + [Signatures in Plain JSON ](#signatures-in-plain-json)
    + [Signatures on Semantics (Verifiable Credentials)](#signatures-on-semantics-verifiable-credentials)
    + [Selective Disclosure](#selective-disclosure)
    + [Unlinkability](#unlinkability)
  * [High-level Design discussion](#high-level-design-discussion)
    + [Canonicalization](#canonicalization)
    + [Use of RDF](#use-of-rdf)
  * [Considered alternatives](#considered-alternatives)
    + [JOSE and JWTs](#jose-and-jwts)
    + [COSE and CWTs](#cose-and-cwts)
    + [SD-JWTs and JWPs](#sd-jwts-and-jwps)
  * [Stakeholder Feedback / Opposition](#feedback)
  * [References & acknowledgements](#acknowledgements)

## Introduction

These family of VC Data Integrity specifications describe mechanisms for
ensuring the authenticity and integrity of Verifiable Credentials and similar
types of constrained  digital documents using cryptography, especially through
the use of digital signatures and related mathematical proofs. The resulting
proofs can be embedded in their native representations (JSON, YAML, etc.)
instead of having to be base64-encoded and made opaque to developers and
database tooling. Advanced features such as selective disclosure and unlinkable
digital signatures are supported through the design of this security system.

## Goals

We are trying to make it easier for developers to take advantage of new forms of
cryptography while keeping the resulting digitally signed messages easy to work
with in their native data representations.

https://www.w3.org/TR/vc-data-integrity/#design-goals-and-rationale

## Non-goals

Compatability with JWTs, JOSE, or COSE. There are other specifications that are
under development in the same WG that do that, such as VC-JWT.

Digitally signing experimental graph formats such as Labeled Property Graphs,
RDF-Star graphs, or other newer graph formats.

## User research

The work on Data Integrity started over a decade ago and has been incubated in
the W3C Credentials Community Group over that time period. We have performed a
number of interoperability plugfests over the years to test whether or not
developers find using the technology acceptable when they have a use case that
requires the features described in Data Integrity. These technologies have also
been deployed at scale in proof of concept, pilot, and production systems, in
retail point of sale solutions, in federal government systems, and
consumer-facing applications. The results of that research are continuously fed
back into future iterations of the work.

## What We're Trying to Accomplish

In general, what Data Integrity attempts to accomplish is to take a document
that looks like this:

```
{
  "title": "Hello world!"
}
```

and digitally sign it so the document ends up looking like this;

```
{
  "title": "Hello world!",
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "example-signature-2022",
    "created": "2020-11-05T19:23:24Z",
    "verificationMethod": "https://di.example/issuer#z6MkjL...XfxnG",
    "proofPurpose": "assertionMethod",
    "proofValue": "zQeVbY4oey5q2M3XKaxu...FkXJeV6doDwLWx"
  }
}
```

The digitally signed document above can then be sent, received, verified, and
stored in a way that enables the receiver to determine if the data has been
tampered with in transit.

## How it Works

The operation of Data Integrity is conceptually simple. To create a
cryptographic proof, the following steps are performed: 1) Transformation, 2)
Hashing, and 3) Proof Generation.

![image](https://github.com/w3c/vc-data-integrity/assets/108611/43b43b44-5865-491f-9584-d351bfb72eb9)

Figure 1 To create a cryptographic proof, data is transformed, hashed, and
cryptographically protected.

Transformation is a process described by a transformation algorithm that takes
input data and prepares it for the hashing process. One example of a possible
transformation is to take a record of people's names that attended a meeting,
sort the list alphabetically by the individual's family name, and rewrite the
names on a piece of paper, one per line, in sorted order. Possible
transformations include canonicalization and binary-to-text encoding.

Hashing is a process described by a hashing algorithm that calculates an
identifier for the transformed data using a cryptographic hash function. This
process is conceptually similar to how a phone address book functions, where one
takes a person's name (the input data) and maps that name to that individual's
phone number (the hash). Possible cryptographic hash functions include SHA-3 and
BLAKE-3.

Proof Generation is a process described by a proof serialization algorithm that
calculates a value that protects the integrity of the input data from
modification or otherwise proves a certain desired threshold of trust. This
process is conceptually similar to the way a wax seal can be used on an envelope
containing a letter to establish trust in the sender and show that the letter
has not been tampered with in transit. Possible proof serialization functions
include digital signatures and proofs of stake.

To verify a cryptographic proof, the following steps are performed: 1)
Transformation, 2) Hashing, and 3) Proof Verification.

![image](https://github.com/w3c/vc-data-integrity/assets/108611/5708c953-4813-4b90-a060-6e9ead9f0f07)

Figure 2 To verify a cryptographic proof, data is transformed, hashed, and
checked for correctness.

During verification, the transformation and hashing steps are conceptually the
same as described above.

Proof Verification is a process that is described by a proof verification
algorithm that applies a cryptographic proof verification function to see if the
input data can be trusted. Possible proof verification functions include digital
signatures and proofs of stake.

## Key scenarios

### Signatures in Plain JSON 

Mastodon was one of the first implementations that used Data Integrity
signatures (many years ago) to digitally sign messages from individuals. There
is a more recent effort to use the updated Data Integrity cryptosuites to
digitally sign messages. There is a desire to use JSON Canonicalization Scheme
to canonicalize and then embed the proof in messages such that people in the
Fediverse know that a message came from you and it has not been tampered with in
transit (or by the server).

### Signatures on Semantics (Verifiable Credentials)

In order to disambiguate the meaning of properties like "website" or "name", it
is useful to use globally unambiguous URLs for properties. The Verifiable
Credentials Data Model uses JSON-LD to accomplish this task, while attempting to
keep the objects in a JSON format that is familiar to developers. When digitally
signing a Verifiable Credential, transformations are done (RDF Dataset
Canonicalization) to ensure that the information model is secured appropriately.
The result is a JSON object that is familiar to a developer that is also secured
using an information model with global semantics.

### Selective Disclosure

When an entity protects a document, they might want to empower the receiver of
that document to share a subset of the document. This is called "selective
disclosure". The Data Integrity specification (when working in concert with
specific cryptosuites that allow for selective disclosure) enable subsets of
documents to be shared while not breaking the digital signature on the subset of
the document that's shared.

### Unlinkability

Repeatedly sharing information in a document can create a correlation and
tracking risk. For example, repeatedly sharing a driver's license to prove that
one is above a certain age can lead to using the driver's license number to
track the individual. A more appropriate sharing would be to just share one's
age in a way that cannot be used to track the individual. Note that a digital
signature is unique enough to be used as a global tracking token, thus the need
for a digital signature that changes every time its presented while still
allowing the signature to be verified.

## High-level Design discussion

### Canonicalization

One of the difficult design decisions we had to make in the specification had to
do with whether or not we would support canonicalization of content. The
requirement for this came from the schema.org work, where content authors were
marking up things like products for sale, store locations, store hours, and
other impactful data in a way that could be utilized in other venues if it was
digitally signed. This data is often embedded in a web page (as JSON-LD) and
indexed by the search crawlers. The data often contains whitespace content, or
reordering of content in a semantically meaningless manner, that typically
invalidates digital signatures. So, we needed a way to ensure that 1) authors
could continue to embed content for search crawlers in the way they were doing
(JSON-LD embedded in web pages) while 2) ensuring that white space and
reordering didn't negatively affect the digital signature. 

The solution was to use canonicalization, which adds complexity to the design
but allows for the use case (and others like it) to be accomplished. At present,
Data Integrity uses two types of canonicalization. JSON Canonicalization Scheme
(JCS) is used when an author is working with pure JSON data (with no formal
semantics). RDF Dataset Canonicalization is used when the author is working with
formal semantics and might desire the data to be transformed from one
serialization (JSON-LD) into another serialization (NQuads) to be stored and
processed in a graph-based processing engine (and/or) graph database.
Representation in a statement-ordered, graph-based format also provides a
variety of benefits for selective disclosure and unlinkable signatures.

The main cost for performing canonicalization is processing time and memory
overhead. In practice, it does not seem to add significant cost to the systems
in which it has been integrated.

### Use of RDF

One of the Data Integrity canonicalization options includes the ability to
transform to and from RDF. This approach adds complexity, but allows for some
advanced use cases, such as being able to digitally sign over an information
model (and not a base-64 encoded serialization), and the ability to perform
[Semantic Compression via the use of
CBOR-LD](https://lists.w3.org/Archives/Public/public-json-ld-wg/2020Jul/att-0004/Introduction_to_CBOR-LD.pdf).
While Data Integrity does not have to use RDF Dataset Canonicalization (and can
use JCS instead), it's existence continues to be the source of a variety of
criticisms against Data Integrity.

## Considered alternatives

### JOSE and JWTs

The initial solutions for securing Verifiable Credentials used the JOSE/JWT
stack, but the lack of canonicalization, selective disclosure, unlinkability,
and not being able to support an information model with semantics caused us to
look into the alternative proposed in these specifications.

### COSE and CWTs

The same design considerations that led to the avoidance of JOSE and JWT also
led to the avoidance of the COSE and CWT stack, though we do utilize CBOR to
structure the digital signatures in some of the selective disclosure mechanisms.

### SD-JWTs and JWPs

The Data Integrity work was started far in advance of he SD-JWT and JWP work at
IETF. While we are tracking that work, it continues to not align well with the
scenarios outlined above.

## Feedback

There were 17 implementers that demonstrated interoperability on a subset of the
Data Integrity work:

https://docs.google.com/presentation/d/19GmJ3bLMrbVadesnkmsWaaUr-U71Y9Kr775tZvgs-xI/edit#

Additionally, the US Federal Government (DHS) has aligned their latest profile
for digital identity to use Verifiable Credentials and Data Integrity.

There are a number of critics of the Data Integrity approach that are focusing
their efforts on VC-JWT, SD-JWT, and JWP. Those approaches continue to not
address the scenarios and use cases that are a focus for the Data Integrity
work.

## Acknowledgements

Many thanks for valuable feedback and advice from:

Christopher Allen, David Ammouial, Joe Andrieu, Bohdan Andriyiv, Ganesh Annan,
Kazuyuki Ashimura, Tim Bouma, Pelle Braendgaard, Dan Brickley, Allen Brown, Jeff
Burdges, Daniel Burnett, ckennedy422, David Chadwick, Chaoxinhu, Kim (Hamilton)
Duffy, Lautaro Dragan, enuoCM, Ken Ebert, Eric Elliott, William Entriken, David
Ezell, Nathan George, Reto Gmür, Ryan Grant, glauserr, Adrian Gropper, Joel
Gustafson, Amy Guy, Lovesh Harchandani, Daniel Hardman, Dominique
Hazael-Massieux, Jonathan Holt, David Hyland-Wood, Iso5786, Renato Iannella,
Richard Ishida, Ian Jacobs, Anil John, Tom Jones, Rieks Joosten, Gregg Kellogg,
Kevin, Eric Korb, David I. Lehn, Michael Lodder, Dave Longley, Christian
Lundkvist, Jim Masloski, Pat McBennett, Adam C. Migus, Liam Missin, Alexander
Mühle, Anthony Nadalin, Clare Nelson, Mircea Nistor, Grant Noble, Darrell
O'Donnell, Nate Otto, Matt Peterson, Addison Phillips, Eric Prud'hommeaux, Liam
Quin, Rajesh Rathnam, Drummond Reed, Yancy Ribbens, Justin Richer, Evstifeev
Roman, RorschachRev, Steven Rowat, Pete Rowley, Markus Sabadello, Kristijan
Sedlak, Tzviya Seigman, Reza Soltani, Manu Sporny, Orie Steele, Matt Stone,
Oliver Terbu, Ted Thibodeau Jr, John Tibbetts, Mike Varley, Richard Varn,
Heather Vescent, Christopher Lemmer Webber, Benjamin Young, Kaliya Young, Dmitri
Zagidulin, and Brent Zundel.
