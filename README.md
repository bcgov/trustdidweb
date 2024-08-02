# Trust DID Web - A DID Method

The spec repository for did:tdw -- Trust DID Web DID method.

Read the spec: [https://bcgov.github.io/trustdidweb/](https://bcgov.github.io/trustdidweb/)

Implementations available:

- Typescript: [https://github.com/bcgov/trustdidweb-ts](https://github.com/bcgov/trustdidweb-ts)
- Python: [https://github.com/bcgov/trustdidweb-py](https://github.com/bcgov/trustdidweb-py)

## Abstract

The `did:tdw` (Trust DID Web) method is an enhancement to the
`did:web` protocol, providing a complementary web-based DID method that addresses limitations
of `did:web`. It's features include the following.

- Ongoing publishing of all DID Document (DIDDoc) versions for a DID instead of,
  or alongside a `did:web` DID/DIDDoc.
- Uses the same DID-to-HTTPS transformation as `did:web`.
- Provides resolvers the full history of the DID using a verifiable chain of
  updates to the DIDDoc from genesis to deactivation.
- A self-certifying identifier (SCID) for the DID that is globally
  unique and derived from the initial DIDDoc that enables DID portability, such
  as moving the DIDs web location (and so the DID string itself) while retaining
  the DID's history.
- DIDDoc updates include a proof signed by the DID Controller(s) *authorized* to
  update the DID.
- An optional mechanism for publishing "pre-rotation" keys to prevent loss of
  control of the DID in cases where an active private key is compromised.
- An optional mechanism for having collaborating "witnesses"
  that approve updates to the DID by the DID Controller before publication.
- DID URL path handling that defaults (but can be overridden) to automatically
  resolving `<did>/path/to/file` by using a comparable DID-to-HTTPS translation
  as for the DIDDoc.
- A DID URL path `<did>/whois` that defaults to automatically returning (if
  published by the DID controller) a Verifiable Presentation containing
  Verifiable Credentials with the DID as the `credentialSubject`,
  signed by the DID.

Combined, the additional features enable greater trust and security without
compromising the simplicity of `did:web`. The incorporation of the DID Core
compatible "/whois" path, drawing inspiration from the traditional WHOIS
protocol, offers an easy to use, decentralized, trust registry.
This `did:tdw` aims to establish a more trusted and secure web environment by
providing robust verification processes and enabling transparency and
authenticity in the management of decentralized digital identities.

## Contributing to the Specification

Pull requests (PRs) to this repository may be accepted. Each commit of a PR must
have a DCO (Developer Certificate of Origin -
[https://github.com/apps/dco](https://github.com/apps/dco)) sign-off. This can
be done from the command line by adding the `-s` (lower case) option on the `git
commit` command (e.g., `git commit -s -m "Comment about the commit"`).

Rendering and reviewing the spec locally for testing requires `npm` and `node`
installed. Follow these steps:

- Fork and locally clone the repository.
- Install `node` and `npm`.
- Run `npm install` from the root of your local repository.
- Edit the spec documents (in the `/spec` folder).
- Run `npm run render`'
  - Use `npm run edit` to interactively edit, render and review the spec.
- Review the resulting `index.html` file in a browser.

The specification is currently in
[Spec-Up](https://github.com/decentralized-identity/spec-up) format. See the
[Spec-Up Documentation](https://identity.foundation/spec-up/) for a list of
Spec-Up features and functionality.
