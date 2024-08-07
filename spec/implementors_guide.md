## Implementors Guide

### Implementations

Proof of concept implementations of `did:tdw` software for [[ref: DID Controllers]] and resolvers can be found here:

- Typescript: [https://github.com/bcgov/trustdidweb-ts](https://github.com/bcgov/trustdidweb-ts)
- Python: [https://github.com/bcgov/trustdidweb-py](https://github.com/bcgov/trustdidweb-py)

Both currently (as of 2024.04.11) support all of the features of the core `did:tdw` including [[ref: Key Pre-Rotation]]. Not yet supported is the the concept of [[ref: witnesses]].

### Using Pre-Rotation Keys

In an effort to prevent the loss of control over a decentralized identifier (DID) due to a compromised private key, [[ref: pre-rotation]] keys are introduced. These commitments, made by the [[ref: DID Controller]], are declarations about the keys that will be published in future versions of the DID document, without revealing the keys themselves.

The primary goal of [[ref: pre-rotation]] keys is to ensure that even if an attacker
gains access to the current active key, they will not be able to take control of
the DID. This safeguard is achieved because the attacker could not simply rotate to
a key they generate and control. Rather, they would need to have also
compromised the unpublished (and presumably securely stored) [[ref: pre-rotation]] key in
order to rotate the DID keys.

The cost of having [[ref: pre-rotation]] protection is a much more complicated process to update
the keys of a DID. The following are some considerations we have come across in
considering how to use the [[ref: pre-rotation]] feature. The feature definitely adds a
layer of key management complexity in return for the benefit.

#### Key Rotation with Pre-Rotation

In using [[ref: pre-rotation]], a [[ref: DID Controller]] should generate an "active" key
for the [[ref: DIDDoc]] where it can be used for "production" purposes (signing,
decrypting), and generates the "next key" in an isolated location from
production. This prevents both the "active" and "next key" from being compromised in the
same attack. For example, an intruder gets into your infrastructure and is able to extract all of your
private keys both DID control keys would be lost. Thus, we expect the feature to be used as follows:

- The [[ref: DID Controller]] creating the DID would request from an isolated
  service the hash of the "next key" as defined in this specification. For
  example, an entity might have the "active" DID/key hosted by one Cloud
  Provider, and the "next key" by another, on the theory that an attacker might
  get into one environment or another but not both.
- When a key rotation is to be done, two entries are put in the log, using the following steps by the [[ref: DID Controller]]:
  1. Get the full key reference entry from the isolated service for the [[ref: pre-rotation]] "nextKey".
  2. Locally generate a [[ref: pre-rotation]] key hash for a new key that will soon become the "active" key.
  3. Add a [[ref: DID log]] entry that includes the items from the previous two steps, and signs the proof using an authorized key (that presumably it controls, though not required).
    1. Although the [[ref: DID log]] could be published now, it is probably best to hold off and publish it after adding a second, as described by the rest of the steps.
  4. Get a new [[ref: pre-rotation]] hash from the isolated service.
  5. Get the full key-rotation key reference for the [[ref: pre-rotation]] hash created for the last [[ref: DID log entry]].
  6. Add a [[ref: DID Log]] entry that includes the items from the previous two step
  7. If the key rotated in the previous [[ref: DID log entry]] was a/the
     authorized key to make updates to the DID, call the isolated service to produce
     the [[ref: Data Integrity]] proof over the entry using the key the isolated
     service controls.
     1. This step is not needed if the active service has a key authorized to sign the [[ref: DIDDoc]] update.
  8. Publish the new [[ref: DID log]] containing the two new entries.

#### Post Quantum Attacks

One of the potential benefits of this approach to [[ref: pre-rotation]] is that it is
"post-quantum safe". The idea is that in a post-quantum world, the availability
of the published key and signatures may enable the calculation of the
corresponding private key. Since the [[ref: pre-rotation]] value is a hash of the
`nextKey` and not the public key itself, a post-quantum attack would not
compromise that key, and so a further rotation by the attacker would not be
possible. If there was a (suspected) need to transition to using a quantum-safe
key, the same process listed above would be used, but key reference and the
pre-rotation hash added into the second [[ref: DID log]] entry would presumably
both be for quantum-safe keys.

#### Challenges in Using Pre-Rotation

This draft specification states that once [[ref: pre-rotation]] is enabled (via [[ref:
DID log entry]] [[ref: parameter]]), it **MUST** apply to all of the keys in the
DIDDoc. However, we're not sure if that is needed, or if the [[ref: pre-rotation]] should
only apply to keys that are authorized to update the DID.

Key management is hard enough without having to maintain isolated key generation
environments for creating keys for different purposes. Enabling connectivity between
the key generation environments to enable automated key rotation while maintaining the
key recovery environment as "isolated" is technically challenging.

### Using DID Portability

As noted in the [DID Portability](#did-portability) section of the
specification, a `did:tdw` DID can be renamed (ported) by changing the `id` DID string in
the [[ref: DIDDoc]] to one that resolves to a different HTTPS URL, as long as the
specified conditions are met.

While the impact of the feature is in fact the creation of a new DID, we think
there is significant value in some use cases for supporting the specified
capability. Ideally, the HTTPS URL for the "old" DID is changed to a redirect to
the new DID, allowing for a seamless, verifiable evolution of the DID.

An interesting example use case is a DID that replaces an email address hosted
by a particular service. The extra capabilities of having the identifier being a
DID vs. an email address is compelling enough, allowing it to be used for a
range of services beyond email. The [[ref: portability]] benefit comes when the owner of
the DID decides to move to a new service, taking their DID with them. The
verifiable history carried over to the renamed DID hosted by the new service
provides assurance to those who interacted with the old DID (through chats,
emails, postings, etc.) that they are still engaging with the same entity,
despite the DID renaming. Compare that with what happens today when you switch
from one email provider to another, and you have to reach out to all your
contacts to assure them that you changed providers.

While [[ref: portability]] is powerful, it must be used with care and only in use
cases where the capability is specifically required. When used, both the
pre-rotation and [[ref: witnesses]] features of `did:tdw` **SHOULD** also be enabled.

#### Mergers, Acquisitions and Name Changes

Organizations change over time and such changes often involve names changes.
Name changes in turn trigger domain name changes, as organizations match their
Web location with their names. Mergers, acquisitions, and simple name changes,
all can cause an organization's "known" domain name to change, including the
relinquishment of control over their previous domain name. When such changes
occur, it is very unlikely that just because the organization's DIDs use the old
domain name will prevent the changes. Thus the DIDs need to "adapt" to the new
domain -- the domain name portion of the DID has to change. Ideally, the old
location and domain can be retained and a web redirect used to resolve the old
DID to the new, but even if that cannot be done, the ability to use the same
SCID and retain the full history can be preserved.

#### DID Hosting Service Providers

Consider being able to replace the current identifiers we are given (email
addresses, phone numbers) with `did:tdw` DIDs. Comparable hosting platforms
might publish our DIDs for us (ideally, with us in custody of our own private
keys...). Those DIDs, with the inherent public keys can be used for many
purposes -- encrypted email (hello PGP!), messaging, secure file sharing, and
more.

From time to time in that imagined future, we may want to move our DIDs
from one hosting service to another, just as we move from one email or mobile
provider to another. With DIDs that can move and retain the history, we can make
such moves smoothly. Contacts will see the change, but also see that the history
of the DID remains.

#### Challenges in Moving a DID

While we see great value (and even a hard requirement) for being able to move a
DID's web location, it does create challenges in aligning with the
[[spec:DID-CORE]] specification. These challenges are listed below.

Moving a `did:tdw` is actually the (partial or complete) deactivation of the old
DID and the creation of a new DID. The use of the [[ref: SCID]] and the way it
is generated is designed to prevent an attacker from being able to create a DID
they control but with the same [[ref: SCID]] as existing DID. Thus, "finding" a `did:tdw`
with the same [[ref: SCID]] implies the DIDs are the same. That can be verified by
processing the [[ref: DID Log]].

By retaining the incrementing of the `versionId` after a move, the "new" DID
does not start at `versionId` of `1`. Further, resolving `<new-did>?versionId=1`
is going to return a [[ref: DIDDoc]] with the top-level `id` equal to the `<old-did>`.
This is useful from a business perspective, but unexpected from a
[[spec:DID-CORE]] perspective.

### Witnesses and Watchers

To be added.
