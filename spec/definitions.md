## Definitions

[[def: base58btc]]

~ Applies [[spec:draft-msporny-base58-03]] to convert
data to a `base58` encoding. Used in `did:tdw` for encoding hashes for [[ref: SCIDs]] and [[ref: entry hashes]].

[[def: Data Integrity]]

~ [W3C Data
Integrity](https://www.w3.org/community/reports/credentials/CG-FINAL-data-integrity-20220722/)
is a specification of mechanisms for ensuring the authenticity and integrity of
structured digital documents using cryptography, such as digital signatures and
other digital mathematical proofs.

[[def: Decentralized Identifier, Decentralized Identifiers]]

~ Decentralized Identifiers (DIDs) [[spec:did-core]] are a type of identifier that enable
verifiable, decentralized digital identities. A DID refers to any subject (e.g.,
a person, organization, thing, data model, abstract entity, etc.) as determined
by the controller of the DID.

[[def: DIDDoc]]

~ A DID Document as defined by the [[spec: DID-Core]] -- the document returned when a DID is resolved.

[[def: DID:key]]

~ `DID:key`...

[[def: DID Log, DID Logs]]

~ A DID Log is a list of [[ref: Entries]] one being added for each update of an entry item,
including new versions of the [[ref: DIDDoc]] or changed information necessary to generate or validate the DID.

[[def: DID Log Entry, DID Log Entries, Entries]]

~ A DID Log Entry is a JSON array of five items which define the authorized
transformation of a [[ref: DIDDoc]] from one version to the next. The initial entry
establishes the DID and version 1 of the [[ref: DIDDoc]]. All entries are stored
in the [[ref: DID Log]].

[[def: DID Method, DID Methods]]

~ DID methods are the mechanism by which a particular type of DID and its
associated DID document are created, resolved, updated, and deactivated. DID
methods are defined using separate DID method specifications. This document is
the DID Method Specification for `DID:tdw`.

[[def: DID Portability, DID:tdw portability, `DID:tdw` portability]]

~ `did:tdw` [[ref: portability]] encompasses the ability to change the DID string for the
DID while retaining the [[ref: SCID]] and the history of the DID. This is useful
when forced to change (such as when an organization is acquired by another,
resulting in a change of domain names) and when changing DID hosting service
providers.

[[def: did:web]]

~ `did:web` as described in the [W3C specification](https://w3c-ccg.github.io/did-method-web/)
is a DID method that leverages the Domain Name System (DNS) to perform the DID operations.
It is valued for its simplicity and ease of deployment compared to DID methods that are
based on distributed ledgers or blockchain technology, but also comes with increased
challenges related to trust and security. `did:web` provides a starting point for `did:tdw`,
which complements `did:web` with specific features to address the limitations
while still providing ease of deployment.

[[def: Entry Hash, entryHash]]

~ A `DID:tdw` entry hash is a hash generated using a formally defined process
over the input data to a [[ref: log entry]], excluding the [[ref: Data Integrity]] proof
item. The input data includes content from the predecessor to the version of the
DID, ensuring that all the versions are "chained" together in a microledger. The
generated [[ref: entry hash]] is subsequently included in the `versionId` element of the [[ref: log entry]] and **MUST** be
verified by a resolver.

[[def: ISO8601, ISO8601 String]]

~ A date/time expressed using the [ISO8601
Standard](https://en.wikipedia.org/wiki/ISO_8601).

[[def: JSON Canonicalization Scheme]]

~ [[spec:rfc8785]] defines a method for canonicalizing a JSON
structure such that is suitable for verifiable hashing or signing.

[[def: JSON Lines]]

~ A file of JSON Lines, as described on the site
[https://jsonlines.org/](https://jsonlines.org/). In short, `JSONL` is lines of JSON with
whitespace removed and separated by a newline that is convenient for handling
streaming JSON data or log files.

[[def: JSON Patch]]

~ [[spec:rfc6902]] is a web standard format for describing how to change a JSON
document from one state to another. A [[ref: DID Controller]] **MAY** use it in `DID:tdw`
to define how a [[ref: DIDDoc]] is changed from one version to the next.

[[def: Pre-Rotation]]

~ A technique for a controller of a cryptographic key to commit to the public
key it will rotate to next, without exposing that actual public key. It protects
from an attacker that gains knowledge of the current private key from being
able to rotate to a new key known only to the attacker.

[[def: Linked-VP, Linked Verifiable Presentation]]

~ A [[spec:DID-CORE]] `service` entry that specifies where a [[ref: verifiable
presentation]] about the DID subject can be found. The [Decentralized Identity
Foundation](https://identity.foundation/) hosts the [Linked VP
Specification](https://identity.foundation/linked-vp/).

[[def: multihash]]

~ Per the [[spec:draft-multiformats-multihash-07]], [[ref: multihash]] is a specification
for differentiating instances of hashes. Software creating a hash prefixes
(according to the specification) data to the hash indicating the algorithm used
and the length of the hash, so that software receiving the hash knows how to
verify it. Although [[ref: multihash]] supports many hash algorithms, for
interoperability, [[ref: DID Controllers]] **MUST** only use the hash algorithms defined
in this specification as permitted.

[[def: multi-sig, multisig]]

~ A cryptographic signature that to be valid **MUST** contain a defined threshold
(for example, 4 of 7) individual signatures to be considered valid. The
multi-signature key reference points to a verification method that defines what
keys may contribute to the signature, and under what conditions the
multi-signature is considered valid.

[[def: parameters]]

~ `DID:tdw` [[ref: parameters]] are a defined set of configurations that control how the
issuer has generated the DID, and how the resolver should process the DID [[ref: Log
entries]]. The use of [[ref: parameters]] allows for the controlled evolution of `DID:tdw`
log handling, such as evolving the permitted hash algorithms.

[[def: self-certifying identifier, SCID, SCIDs]]

~ An object identifier derived from initial data such that an attacker could not
create a new object with the same identifier. The input for a `DID:tdw` SCID is
the initial [[ref: DIDDoc]] with the placeholder `{SCID}` wherever the SCID is to be
placed.

[[def: Verifiable Credential, Verifiable Credentials]]

~ A verifiable credential can represent all of the same information that a physical credential represents, adding technologies such as digital signatures, to make the credentials more tamper-evident and so more trustworthy than their physical counterparts. The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) is a W3C Standard.

[[def: Verifiable Presentation, Verifiable Presentations]]

~ A [[ref: verifiable presentation]] data model is part W3C's [Verifiable Credential Data
Model](https://www.w3.org/TR/vc-data-model/) that contains a set of [[ref:
verifiable credentials]] about a `credentialSubject`, and a signature across the
verifiable credentials generated by that subject. In this specification, the use
case of primary interest is where the DID is the `credentialSubject` and the DID
signs the [[ref: verifiable presentation]].

[[def: witness, witnesses]]

~ Witnesses are participants in the process of creating and verifying a version
of a `DID:tdw` [[ref: DIDDoc]]. Notably, a witness receives from the [[ref: DID Controller]] a [[ref: DID
Log]] entry ready for publication, verifies it according to this specification,
and approves it according to its ecosystem governance (whatever that might be). If the verification and
approval process results are positive, witnesses returns to the DID Controller a [[ref: Data Integrity]] proof
attesting to that positive result.

[[def: W3C VCDM]]

~ A Verifiable Credential that uses the Data Model defined by the W3C [[spec: W3C-VC]] specification.
