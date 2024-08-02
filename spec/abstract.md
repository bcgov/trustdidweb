## Abstract

Trust DID Web (`did:tdw`) is an enhancement to the `did:web` DID method,
providing complementary web-based features that address `did:web`'s
limitations. `did:tdw` features include:

- Ongoing publishing of all DID Document ([[ref: DIDDoc]]) versions for a DID instead of,
  or alongside a current `did:web` DID/DIDDoc.
- The same DID-to-HTTPS transformation as `did:web`.
- Supports the same [High Assurance DID-to-DNS] mechanism.
- The ability to resolve the full history of the DID using a verifiable chain of
  updates to the [[ref: DIDDoc]] from genesis to deactivation.
- A [[ref: self-certifying identifier]] (SCID) for the DID. The SCID, globally unique and
  embedded in the DID, is derived from the initial [[ref: DID log entry]]. It ensures the integrity
  of the DID's history mitigating the risk of attackers creating a new object with
  the same identifier.
- An optional mechanism for enabling [[ref: DID portability]] via the [[ref: SCID]], allowing
  the DID's web location to be moved and the DID string to be updated, both while retaining
  a connection to the predecessor DID(s) and preserving the DID's verifiable history.
- [[ref: DIDDoc]] updates contain a proof signed by the [[ref: DID Controllers]] *authorized* to
  update the DID.
- An optional mechanism for publishing "pre-rotation" keys to prevent the loss of
  control of a DID in cases where an active private key is compromised.
- An optional mechanism for having collaborating [[ref: witnesses]]
  that approve of updates to the DID by a [[ref: DID Controller]] before publication.
- DID URL path handling that defaults (but can be overridden) to automatically
  resolving `<did>/path/to/file` by using a comparable DID-to-HTTPS translation
  as for the [[ref: DIDDoc]].
- A DID URL path `<did>/whois` that defaults to automatically returning (if
  published by the [[ref: DID controller]]) a [[ref: Verifiable Presentation]] containing
  [[ref: Verifiable Credentials]] with the DID as the `credentialSubject`,
  signed by the DID.

[High Assurance DID-to-DNS]: https://datatracker.ietf.org/doc/draft-carter-high-assurance-dids-with-dns/

Combined, the additional features enable greater trust and security without
compromising the simplicity of `did:web`.

The incorporation of the DID Core compatible "/whois" path, drawing inspiration
from the traditional WHOIS protocol [[spec:rfc3912]], offers an easy-to-use,
decentralized, trust registry. The `did:tdw` method aims to establish a more
trusted and secure web environment by providing robust verification processes
and enabling transparency and authenticity in the management of decentralized
digital identities.
