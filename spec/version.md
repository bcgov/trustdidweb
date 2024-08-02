## `did:tdw` Version Changelog

The following lists the substantive changes in each version of the specification.

- Version 0.3
  - Change base32 encoding with [[ref: base58btc]], as it offers a better expansion rate.
  - Remove the step to extract part of the [[ref: base58btc]] result during the generation of the [[ref: SCID]].
  - Use [[ref: multihash]] in the [[ref: SCID]] to differentiate the different hash function outputs.
- Version 0.2
  - Changes the location of the [[ref: SCID]] in the DID to always be the first
    component after the DID Method prefix -- `did:tdw:<scid>:...`.
  - Adds the [[ref: parameter]] `portable` to enable the capability to move a
    `did:tdw` during the creation of the DID.
  - Removes the first two [[ref: Log Entry]] items `entryHash` and `versionId`
    and replacing them with the new `versionId` as the first item in each [[ref: log
    entry]]. The new versionId takes the form `<version number>-<entryHash>`,
    where `<version number>` is the incrementing integer of version of the
    entry: 1, 2, 3, etc.
  - The `<did>/whois` media type is changed to `application/vp` and the file is
    changed to `whois.vp` to match the IANA registration of a [[ref: Verifiable
    Presentation]].
