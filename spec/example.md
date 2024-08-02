## `did:tdw` Example

The following shows the evolution of a `did:tdw` from inception through several
versions, showing the DID, [[ref: DIDDoc]], [[ref: DID Log]], and some of the
intermediate data structures.

**The examples are aligned with version 0.3 of the `did:tdw` specification.**

In some of the following examples the data for the [[ref: DID log entries]] is displayed
as prettified JSON for readability. In the log itself, the JSON has all
whitespace removed, and each line ends with a `CR`, per the [[ref: JSON Lines]] convention.

### DID Creation Data

These examples show the important structures used in the [Create (Register)](#create-register) operation for a `did:tdw` DID.

#### Input to the SCID Generation Process with Placeholders

The following JSON is an example of the input that the [[ref: DID Controller]]
constructs and passes into the
[SCID Generation Process](#scid-generation-and-verification). In this example, the [[ref: DIDDoc]] is
particularly boring, containing the absolute minimum for a valid [[ref: DIDDoc]].

This example includes both the initial "authorized keys" to sign the [[ref: Data Integrity]] proof
(`updateKeys`) and the [[ref: pre-rotation]] commitment to the next authorization keys (`nextKeyHashes`). Both
are in the `parameters` item in the [[ref: log entry]].

```json
[
  "{SCID}",
  "2024-07-29T17:00:27Z",
  {
    "prerotation": true,
    "updateKeys": [
      "z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc"
    ],
    "nextKeyHashes": [
      "QmcbM5bppyT4yyaL35TQQJ2XdSrSNAhH5t6f4ZcuyR4VSv"
    ],
    "method": "did:tdw:0.3",
    "scid": "{SCID}"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:{SCID}:domain.example"
    }
  }
]
```

#### Output of the SCID Generation Process

After the [[ref: SCID]] is generated, the literal `{SCID}` placeholders are
replaced by the generated [[ref: SCID]] value (below). This JSON is the input to
the [`entryHash` generation process](#entry-hash-generation-and-verification) --
with the [[ref: SCID]] as the first item of the array. Once the process has run,
the version number of this first version of the DID (`1`), a dash `-` and the
resulting output hash replace the [[ref: SCID]] as the first item in the array
-- the `versionId`.

```json
[
  "Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu",
  "2024-07-29T17:00:27Z",
  {
    "prerotation": true,
    "updateKeys": [
      "z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc"
    ],
    "nextKeyHashes": [
      "QmcbM5bppyT4yyaL35TQQJ2XdSrSNAhH5t6f4ZcuyR4VSv"
    ],
    "method": "did:tdw:0.3",
    "scid": "Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"
    }
  }
]
```

#### Data Integrity Proof Generation and First Log Entry

The last step in the creation of the first [[ref: log entry]] is the generation
of the data integrity proof. One of the keys in the `updateKeys` [[ref:
parameter]] **MUST** be used (in the form of a `did:key`) to generate the
signature in the proof, with the `versionId` value (item 1 of the [[ref: did log
entry]]) used as the `challenge` item. The generated proof is added to the JSON
Line as the sixth item, and the entire array becomes the first entry in the
[[ref: DID Log]].

The following is the JSON prettified version of the entry log file that is published
as the `did.jsonl` file. When published, all extraneous whitespace is removed, as
shown in the block below the pretty-printed version.

```json
[
  "1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ",
  "2024-07-29T17:00:27Z",
  {
    "prerotation": true,
    "updateKeys": [
      "z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc"
    ],
    "nextKeyHashes": [
      "QmcbM5bppyT4yyaL35TQQJ2XdSrSNAhH5t6f4ZcuyR4VSv"
    ],
    "method": "did:tdw:0.3",
    "scid": "Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"
    }
  },
  [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-jcs-2019",
      "verificationMethod": "did:key:z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc#z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc",
      "created": "2024-07-29T17:00:27Z",
      "proofPurpose": "authentication",
      "challenge": "1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ",
      "proofValue": "zDk24L4vbVrFm5CPQjRD9KoGFNcV6C3ub1ducPQEvDQ39U68GiofAndGbdG9azV6r78gHr1wKnKNPbMz87xtjZtcq9iwN5hjLptM9Lax4UeMWm9Xz7PP4crToj7sZnvyb3x4"
    }
  ]
]
```

The same content "un-prettified", as it is found in the `did.jsonl` file:

```json
["1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ", "2024-07-29T17:00:27Z", {"prerotation": true, "updateKeys": ["z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc"], "nextKeyHashes": ["QmcbM5bppyT4yyaL35TQQJ2XdSrSNAhH5t6f4ZcuyR4VSv"], "method": "did:tdw:0.3", "scid": "Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu"}, {"value": {"@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/multikey/v1"], "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"}}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc#z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc", "created": "2024-07-29T17:00:27Z", "proofPurpose": "authentication", "challenge": "1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ", "proofValue": "zDk24L4vbVrFm5CPQjRD9KoGFNcV6C3ub1ducPQEvDQ39U68GiofAndGbdG9azV6r78gHr1wKnKNPbMz87xtjZtcq9iwN5hjLptM9Lax4UeMWm9Xz7PP4crToj7sZnvyb3x4"}]]
```

#### `did:web` Version of DIDDoc

As noted in the [publishing a parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification a `did:tdw` can be published
by replacing `did:tdw` with `did:web` in the [[ref: DIDDoc]], adding an `alsoKnownAs` entry for the `did:tdw`
and publishing the resulting [[ref: DIDDoc]] at `did.json`, logically beside the `did.jsonl` file.

Here is what the `did:web` [[ref: DIDDoc]] looks like for the `did:tdw` above.

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/multikey/v1"
  ],
  "id": "did:web:domain.example",
  "alsoKnownAs": ["did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"]
}
```

### Version 2 of the DIDDoc

Time passes, and the [[ref: DID Controller]] of the `did:tdw` DID decides to
update its DID to a new version, version 2. In this case, the only change
the [[ref: DID Controller]] makes is transition the authorization key to
the [[ref: pre-rotation]] key.

#### Version 2 Entry Hashing Input

To generate a new version of the DID, the [[ref: DID Controller]] needs to
provide the existing [[ref: DID log]] file, the updated `parameters`, and the new [[ref: DIDDoc]].
The following processing is done to create the new [[ref: DID log entry]]:

- The `versionId` from the previous (first) [[ref: log entry]] is made the first item in the new [[ref: log entry]].
- The `versionTime` is generated as the current time, and made the new `versionTime` (second) item.
- The `parameters` entry passed in is processed. In this case, since the
`updateKeys` array is updated, and [[ref: pre-rotation]] is active, the a
verification is done to ensure that the hash of the `updateKeys` are found in
the `nextKeyHashes` item from version 1 of the DID. As required by the `did:tdw`
specification, a new `nextKeyHashes` is included in the `parameters`.
- The new (but unchanged) [[ref: DIDDoc]] is included in its entirety, as the value of `value`, set as the third item in the array.
  - The implementation could have used [[ref: JSON Patch]] to generate value of the `patch` item.
- The resulting array is passed into the [`entryHash` generation
  process](#entry-hash-generation-and-verification) which outputs the
  `entryHash` for this [[ref: log entry]]. Once again, the first item
  (`versionId`) in the [[ref: log entry]] is replaced by the version number (the
  previous version number plus `1`), a dash (`-`), and the new `entryHash`.
- The [[ref: data integrity]] proof is generated added to the [[ref: log entry]]
  as the sixth item, and the entire entry is added to the existing [[ref: DID
  log]].

The [[ref: DID log]] file can now be published, optionally with an updated version of the corresponding `did:web` DID.

The following is the JSON pretty-print [[ref: log entry]] for the second version of an example `did:tdw`. Things to note in this example:

- The [[ref: data integrity]] proof `verificationMethod` is the `did:key` from the first [[ref: log entry]], and the `challenge` is the `versionId` from this [[ref: log entry]].
- A new `updateKeys` item in the `parameters` has been added, a commit to a future key that will control updates to the DID.

```json
[
  "2-QmY2v1VzkeMxF7MSfrLfZswQ74Y6FfrMR1LmuvPJQJwhi6",
  "2024-07-29T17:00:28Z",
  {
    "updateKeys": [
      "z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ"
    ],
    "nextKeyHashes": [
      "QmcCbGzGNr2EFduauzCoh3Hwt1GkRW4Gnkk5nxbr3625de"
    ]
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"
    }
  },
  [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-jcs-2019",
      "verificationMethod": "did:key:z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc#z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc",
      "created": "2024-07-29T17:00:28Z",
      "proofPurpose": "authentication",
      "challenge": "2-QmY2v1VzkeMxF7MSfrLfZswQ74Y6FfrMR1LmuvPJQJwhi6",
      "proofValue": "z2VDUyVapPpb6rC4YbLZRLcWS2zg9o53JU97QjNQYH7JvGs5Ccnf2b647Gw96G5N8rvEKc77uQTGqYvLJ6zrqNwGnqNLraTPD2AL2rR2eUiRKnM5KhbwWumDy5eqmTumm1FWp"
    }
  ]
]
```

#### Log File For Version 2

The new version 2 `did.jsonl` file contains two entries, one for each version
of the [[ref: DIDDoc]].

```json
["1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ", "2024-07-29T17:00:27Z", {"prerotation": true, "updateKeys": ["z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc"], "nextKeyHashes": ["QmcbM5bppyT4yyaL35TQQJ2XdSrSNAhH5t6f4ZcuyR4VSv"], "method": "did:tdw:0.3", "scid": "Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu"}, {"value": {"@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/multikey/v1"], "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"}}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc#z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc", "created": "2024-07-29T17:00:27Z", "proofPurpose": "authentication", "challenge": "1-QmdwvukAYUU6VYwqM4jQbSiKk1ctg12j5hMTY6EfbbkyEJ", "proofValue": "zDk24L4vbVrFm5CPQjRD9KoGFNcV6C3ub1ducPQEvDQ39U68GiofAndGbdG9azV6r78gHr1wKnKNPbMz87xtjZtcq9iwN5hjLptM9Lax4UeMWm9Xz7PP4crToj7sZnvyb3x4"}]]
["2-QmY2v1VzkeMxF7MSfrLfZswQ74Y6FfrMR1LmuvPJQJwhi6", "2024-07-29T17:00:28Z", {"updateKeys": ["z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ"], "nextKeyHashes": ["QmcCbGzGNr2EFduauzCoh3Hwt1GkRW4Gnkk5nxbr3625de"]}, {"value": {"@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/multikey/v1"], "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example"}}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc#z82LkvR3CBNkb9tUVps4GhGpNvEVP6vWzdwgGwQbA1iYoZwd7m1F1hSvkJFSe6sWci7JiXc", "created": "2024-07-29T17:00:28Z", "proofPurpose": "authentication", "challenge": "2-QmY2v1VzkeMxF7MSfrLfZswQ74Y6FfrMR1LmuvPJQJwhi6", "proofValue": "z2VDUyVapPpb6rC4YbLZRLcWS2zg9o53JU97QjNQYH7JvGs5Ccnf2b647Gw96G5N8rvEKc77uQTGqYvLJ6zrqNwGnqNLraTPD2AL2rR2eUiRKnM5KhbwWumDy5eqmTumm1FWp"}]]
```

#### Log File For Version 3

The same process is repeated for version 3 of the DID. In this case:

- The [[ref: DIDDoc]] is changed, and a patch generated.
  - an `authentication` method is added.
  - two services are added.
- No changes are made to the authorized keys to update the DID. As a result, the `parameters` entry is empty (`{}`), and the [[ref: parameters]] in effect from previous versions of the DID remain in effect.

Here is the pretty-printed log entry:

```json
[
  "3-QmNwk72WkEjUMQxqkxYoKWNx8Y1pDiGUZCn9PpeLtfPtyk",
  "2024-07-29T17:00:28Z",
  {},
  {
    "patch": [
      {
        "op": "add",
        "path": "/authentication",
        "value": [
          "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#z6Mkq57k27wL26zrxpvGVdEsCKe5kfpJhzy7GciVUfmosTdv"
        ]
      },
      {
        "op": "add",
        "path": "/assertionMethod",
        "value": [
          "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#z6Mkq57k27wL26zrxpvGVdEsCKe5kfpJhzy7GciVUfmosTdv"
        ]
      },
      {
        "op": "add",
        "path": "/service",
        "value": [
          {
            "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#domain",
            "type": "LinkedDomains",
            "serviceEndpoint": "https://domain.example"
          },
          {
            "id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#whois",
            "type": "LinkedVerifiablePresentation",
            "serviceEndpoint": "https://domain.example/.well-known/whois.vc"
          }
        ]
      },
      {
        "op": "add",
        "path": "/@context/2",
        "value": "https://identity.foundation/.well-known/did-configuration/v1"
      },
      {
        "op": "add",
        "path": "/@context/3",
        "value": "https://identity.foundation/linked-vp/contexts/v1"
      }
    ]
  },
  [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-jcs-2019",
      "verificationMethod": "did:key:z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ#z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ",
      "created": "2024-07-29T17:00:28Z",
      "proofPurpose": "authentication",
      "challenge": "3-QmNwk72WkEjUMQxqkxYoKWNx8Y1pDiGUZCn9PpeLtfPtyk",
      "proofValue": "z2TBssHyJj7dB4LDGjHWm3EfHhBiu534w4ucRF95XG3KzLg4m5kjtYbupGmf4txjqQRPko8Qd8PGeHgykWdutHXxJUmvvpGuiJxNBRpfwfKxnsbrT7jWeT6GqaFYqqkDcCG35"
    }
  ]
]
```

Here is the [[ref: log entry]] for just version 3 of the DID.

```json
["3-QmNwk72WkEjUMQxqkxYoKWNx8Y1pDiGUZCn9PpeLtfPtyk", "2024-07-29T17:00:28Z", {}, {"patch": [{"op": "add", "path": "/authentication", "value": ["did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#z6Mkq57k27wL26zrxpvGVdEsCKe5kfpJhzy7GciVUfmosTdv"]}, {"op": "add", "path": "/assertionMethod", "value": ["did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#z6Mkq57k27wL26zrxpvGVdEsCKe5kfpJhzy7GciVUfmosTdv"]}, {"op": "add", "path": "/service", "value": [{"id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#domain", "type": "LinkedDomains", "serviceEndpoint": "https://domain.example"}, {"id": "did:tdw:Qma6mc1qZw3NqxwX6SB5GPQYzP4pGN2nXD15Jwi4bcDBKu:domain.example#whois", "type": "LinkedVerifiablePresentation", "serviceEndpoint": "https://domain.example/.well-known/whois.vc"}]}, {"op": "add", "path": "/@context/2", "value": "https://identity.foundation/.well-known/did-configuration/v1"}, {"op": "add", "path": "/@context/3", "value": "https://identity.foundation/linked-vp/contexts/v1"}]}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ#z82Lkvgj5NKYhoFh4hWzax9WicQaVDphN8MMzR3JZhontVfHaoGd9JbC4QRpDvmjQH3BLeQ", "created": "2024-07-29T17:00:28Z", "proofPurpose": "authentication", "challenge": "3-QmNwk72WkEjUMQxqkxYoKWNx8Y1pDiGUZCn9PpeLtfPtyk", "proofValue": "z2TBssHyJj7dB4LDGjHWm3EfHhBiu534w4ucRF95XG3KzLg4m5kjtYbupGmf4txjqQRPko8Qd8PGeHgykWdutHXxJUmvvpGuiJxNBRpfwfKxnsbrT7jWeT6GqaFYqqkDcCG35"}]]
```

And so on...
