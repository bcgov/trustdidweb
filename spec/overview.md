## Overview

The emergence of [[ref: Decentralized Identifiers]] (DIDs) and with them the
evolution of [[ref: DID Methods]] continues to be a dynamic area of
development in the quest for trusted, secure and private digital identity
management where the users are in control of their own data.

The `did:web` method, for example, leverages the Domain Name System (DNS) to
perform the DID operations. This approach is praised for its simplicity and
ease of deployment, including DID-to-HTTPS transformation and addressing
some aspects of trust by allowing for DIDs to be associated with a domain's
reputation or published on platforms such as GitHub. However, it is not
without its challenges--
from trust layers inherited from the web and the absence of a verifiable
history for the DID.

Tackling these concerns, the proposed `did:tdw` (Trust DID Web)
method aims to enhance `did:web` by introducing additional features such 
as a [[ref: self-certifying identifier]] (SCID), update key(s)
and a verifiable history, akin to what is available with ledger-based DIDs,
without relying on a ledger.

This approach not only maintains backward compatibility but also offers an
additional layer of assurance for those requiring more robust verification
processes. By publishing the resulting DID as both `did:web` and `did:tdw`, it
caters to a broader range of trust requirements, from those who are comfortable
with the existing `did:web` infrastructure to those seeking greater security
assurances provided by `did:tdw`. This innovative step represents a significant
stride towards a more trusted and secure web, where the integrity of
cryptographic key publishing is paramount.

The key differences between `did:web` and `did:tdw` revolve around the core
issues of decentralization and security. `did:web` is recognized for its
simplicity and cost-effectiveness, allowing for easy establishment of a
credential ecosystem. However, it is not inherently decentralized as it relies
on DNS domain names, which require centralized registries. Furthermore, it lacks a
cryptographically verifiable, tamper-resistant, and persistently stored DID
document. In contrast, `did:tdw` (Trust DID Web) is proposed as an enhancement
to `did:web`, aiming to address these limitations by adding a verifiable history
to the DID without the need for a ledger. This method seeks to provide a more
decentralized approach by ensuring that the security of the embedded
SCID does not depend on DNS. Additionally, `did:tdw` is
capable of resolving a cryptographically verifiable trust registry and status
lists, using DID-Linked Resources, which `did:web` lacks. These features are
designed to build a trusted web, offering a higher level of assurance for
cryptographic key publishing and management.

For backwards compatibility, and for verifiers that "trust" `did:web`, a
`did:tdw` can be trivially modified and published in parallel to a `did:web`
DID. For resolvers that want more assurance, `did:tdw` provides a way to "trust
did:web" (or to enable a "trusted web" if you say it fast) enabled by the
features listed in the [Abstract](#abstract).

The following is a `tl;dr` summary of how `did:tdw` works:

1. `did:tdw` uses the same DID-to-HTTPS transformation as `did:web`, so
   `did:tdw`'s  `did.jsonl` ([[ref: JSON Lines]]) file is found in the same
   location as `did:web`'s `did.json` file, and supports an easy transition
   from `did:web` to gain the added benefits of `did:tdw`.
2. The `did.jsonl` is a list of JSON [[ref: DID log entries]], one per line,
   whitespace removed (per [[ref: JSON Lines]]). Each entry contains the
   information needed to derive a version of the [[ref: DIDDoc]] from its preceding
   version. The `did.jsonl` is also referred to as the [[ref: DID Log]].
3. Each [[ref: DID log entry]] includes a JSON array of five items:
    1. The `versionId` of the entry, a value that combines the version number
       (starting at `1` and incrementing by one per version), a literal dash
       `-`, and a hash of the entry. The [[ref: entry hash]] calculation links each entry
       to its predecessor in a ledger-like chain.
    2. The `versionTime` (as stated by the [[ref: DID Controller]]) of the entry.
    3. A set of `parameters` that impact the processing of the current and
      future [[ref: log entries]].
        - Example [[ref: parameters]] are the version of the `did:tdw` specification and
        hash algorithm being used as well as the [[ref: SCID]] and update key(s).
    4. The new version of the [[ref: DIDDoc]] as either a `value` (the full document) or
      a `patch` derived using [[ref: JSON Patch]] to update the new version from
      the previous entry.
    5. A [[ref: Data Integrity]] (DI) proof across the entry, signed by a [[ref: DID
      Controller]] authorized to update the [[ref: DIDDoc]], using the `versionId` as the
      challenge.
4. In generating the first version of the [[ref: DIDDoc]], the [[ref: DID Controller]] calculates
  the [[ref: SCID]] for the DID from the entire first [[ref: log entry]] (which
  includes the [[ref: DIDDoc]]) by adding the string {SCID} everywhere the actual [[ref: SCID]]
  is to be placed. The [[ref: DID Controller]] then replaces these placeholders
  with the just calculated [[ref: SCID]], including it as a `parameter` in the first [[ref: log
  entry]], and inserting it where needed in the initial (and all subsequent)
  DIDDocs. The [[ref: SCID]] enables an optional [[ref: portability]] capability, allowing a DID's
  web location to be moved to a new location while retaining the DID and version
  history of the DID.
5. A [[ref: DID Controller]] generates and publishes the new/updated [[ref: DID Log]] file by making it
  available at the appropriate location on the web, based on the identifier of the
  DID.
6. Given a `did:tdw` DID, a resolver converts the DID to an HTTPS URL,
  retrieves, and processes the [[ref: DID Log]] `did.jsonl`, generating and verifying
  each [[ref: log entry]] as per the requirements outlined in this specification.
    - In the process, the resolver collects all the [[ref: DIDDoc]] versions and public
      keys used by the DID currently, or in the past. This enables
      resolving both current and past versions of the DID.
7. `did:tdw` DID URLs with paths and `/whois` are resolved to documents
  published by the [[ref: DID Controller]] that are by default in the web location relative to the
  `did.jsonl` file. See the [note below](#the-whois-use-case) about the
   powerful capability enabled by the `/whois` DID URL path.
8. Optionally, a [[ref: DID Controller]] can easily generate and publish a `did:web` DIDDoc
  from the latest `did:tdw` [[ref: DIDDoc]] in parallel with the `did:tdw` [[ref: DID Log]].

  ::: warning
    A resolver settling for just the `did:web` version of the DID does not get the
    verifiability of the `did:tdw` log.
  :::

An example of a `did:tdw` evolving through a series of versions can be seen in
the [did:tdw Examples](#didtdw-example) of this specification.

This draft specification was developed in parallel with the development of two
proof of concept implementations. The specification/implementation interplay
helped immensely in defining a practical, intuitive, straightforward, DID
method. The existing proof of concept implementations of the `did:tdw` DID
Method are listed in the [Implementors Guide](#Implementations). The current
Typescript and Python implementations are each less than 1500 lines of code.

### The `/whois` Use Case

This DID Method introduces what we hope will be a widely embraced convention for
all DID Methods -- the `/whois` path. This feature harkens back to the `WHOIS`
protocol that was created in the 1970s to provide a directory about people and
entities in the early days of ARPANET. In the 80's, `whois` evolved into
[[spec-inform:rfc920]] that has expanded into the [global
whois](https://en.wikipedia.org/wiki/WHOIS) feature we know today as
[[spec-inform:rfc3912]]. Submit a `whois` request about a domain name, and get
back the information published about that domain.

We propose that the `/whois` path for a DID enable a comparable, decentralized,
version of the `WHOIS` protocol for DIDs. Notably, when `<did>/whois` is
resolved (using a standard DID `service` that follows the [[ref: Linked-VP]]
specification), a [[ref: Verifiable Presentation]] (VP) may be returned (if
published by the [[ref: DID Controller]]) containing [[ref: Verifiable Credentials]] with
the DID as the `credentialSubject`, and the VP signed by the DID. Given a DID,
one can gather verifiable data about the [[ref: DID Controller]] by resolving
`<did>/whois` and processing the returned VP. That's powerful -- an efficient,
highly decentralized, trust registry. For `did:tdw`, the approach is very simple
-- transform the DID to its HTTPS equivalent, and execute a `GET <https>/whois`.
Need to know who issued the VCs in the VP? Get the issuer DIDs from those VCs,
and resolve `<issuer did>/whois` for each. This is comparable to walking a CA
(Certificate Authority) hierarchy, but self-managed by the [[ref: DID Controllers]] --
and the issuers that attest to them.

The following is a use case for the `/whois` capability. Consider an example of
the `did:tdw` controller being a mining company that has exported a shipment and
created a "Product Passport" Verifiable Credential with information about the
shipment. A country importing the shipment (the Importer) might want to know
more about the issuer of the VC, and hence, the details of the shipment. They
resolve the `<did>/whois` of the entity and get back a Verifiable Presentation
about that DID. It might contain:

- A verifiable credential issued by the Legal Entity Registrar for the
  jurisdiction in which the mining company is headquartered.
  - Since the Importer knows about the Legal Entity Registrar, they can automate
    this lookup to get more information about the company from the VC -- its
    legal name, when it was registered, contact information, etc.
- A verifiable credential for a "Mining Permit" issued by the mining authority
  for the jurisdiction in which the company operates.
  - Perhaps the Importer does not know about the mining authority for that
    jurisdiction. The Importer can repeat the `/whois` resolution process for
    the issuer of _that_ credential. The Importer might (for example), resolve
    and verify the `did:tdw` DID for the Authority, and then resolve the
    `/whois` DID URL to find a verifiable credential issued by the government of
    the jurisdiction. The Importer recognizes and trusts that government's
    authority, and so can decide to recognize and trust the mining permit
    authority.
- A verifiable credential about the auditing of the mining practices of the
  mining company. Again, the Importer doesn't know about the issuer of the audit
  VC, so they resolve the `/whois` for the DID of the issuer, get its VP and
  find that it is accredited to audit mining companies by the [London Metal
  Exchange](https://www.lme.com/en/) according to one of its mining standards.
  As the Importer knows about both the London Metal Exchange and the standard,
  it can make a trust decision about the original Product Passport Verifiable
  Credential.

Such checks can all be done with a handful of HTTPS requests and the processing
of the DIDs and verifiable presentations. If the system cannot automatically
make a trust decision, lots of information has been quickly collected that can
be passed to a person to make such a decision.

The result is an efficient, verifiable, credential-based, decentralized,
multi-domain trust registry, empowering individuals and organizations to verify
the authenticity and legitimacy of DIDs. The convention promotes a decentralized
trust model where trust is established through cryptographic verification rather
than reliance on centralized authorities. By enabling anyone to access and
validate the information associated with a DID, the "/whois" path contributes to
the overall security and integrity of decentralized networks.
