## `did:tdw` Example

The following shows the evolution of a `did:tdw` from inception through several
versions, showing the DID, DIDDoc, [[ref: DID Log]], and some of the
intermediate data structures.

In the some of the following examples the data for the [[ref: DID log entries]] is displayed
as prettified JSON for readability. In the log itself, the JSON has all
whitespace removed, and each line ends with a `CR`, per the [[ref: JSON Lines]] convention.

### DID Creation Data

These examples show the important structures used in the [Create (Register)](#create-register) operation for a `did:tdw` DID.

#### Input to the SCID Generation Process with Placeholders

The following JSON is an example of the input that the [[ref: DID Controller]]
constructs and passes into the
[SCID Generation Process](#scid-generation-and-validation). In this example, the DIDDoc is
particularly boring, containing the absolute minimum for a valid DIDDoc.

This example includes both the initial "authorized keys" to sign the Data Integrity proof
(`updateKeys`) and the pre-rotation commitment to the next authorization keys (`nextKeyHashes`). Both
are in the `parameters` item in the [[ref: log entry]].

```json
[
  "{SCID}",
  1,
  "2024-06-18T19:25:55Z",
  {
    "hash": "sha3-256",
    "prerotation": true,
    "updateKeys": [
      "z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"
    ],
    "nextKeyHashes": [
      "enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"
    ],
    "method": "did:tdw:1",
    "scid": "{SCID}"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:domain.example:{SCID}"
    }
  }
]
```

#### Output of the SCID Generation Process

After the SCID is generated, the literal `{SCID}` placeholders are replaced by the
generated [[ref: SCID]] value (below). This JSON is the input to the
[`entryHash` generation process](#entry-hash-generation-and-validation) -- with the
[[ref: SCID]] as the first item of the array. Once the process has run, the
resulting output hash replaces the [[ref: SCID]] as the first item in the array.

```json
[
  "lkq33irvim6iktjkagnw2ee6bgcw",
  1,
  "2024-06-18T19:25:55Z",
  {
    "hash": "sha3-256",
    "prerotation": true,
    "updateKeys": [
      "z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"
    ],
    "nextKeyHashes": [
      "enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"
    ],
    "method": "did:tdw:1",
    "scid": "lkq33irvim6iktjkagnw2ee6bgcw"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"
    }
  }
]
```

#### Data Integrity Proof Generation and First Log Entry

The last step in the creation of the first log entry is the generation of the data integrity proof.
One of the `did:key` DIDs in the `updateKeys` parameter **MUST** be used to generate the signature
in the proof, with the `entryHash` value used as the `challenge` item. The generated proof is
added to the JSON Line as the sixth item, and the entire array becomes the first entry in the DID Log.

The following is the JSON prettified version of the entry log file that is published
as the `did.jsonl` file. When published, all extraneous whitespace is removed, as
shown in the block below the pretty-printed version.

```json
[
  "yza4hoyihukpz5wxcag26clggwqbpsjv26zpp3kqc7gtusokvfpa",
  1,
  "2024-06-18T19:25:55Z",
  {
    "hash": "sha3-256",
    "prerotation": true,
    "updateKeys": [
      "z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"
    ],
    "nextKeyHashes": [
      "enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"
    ],
    "method": "did:tdw:1",
    "scid": "lkq33irvim6iktjkagnw2ee6bgcw"
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"
    }
  },
  [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-jcs-2019",
      "verificationMethod": "did:key:z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe#z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe",
      "created": "2024-06-18T19:25:55Z",
      "proofPurpose": "authentication",
      "challenge": "yza4hoyihukpz5wxcag26clggwqbpsjv26zpp3kqc7gtusokvfpa",
      "proofValue": "zUDcd2Yjuzu8XLmZufkYV3Xm8VEZRWgwE9SoKFP83d1ZnKtEwGLf42KCm6nmr8YQVQ6ykusRSSaq8iJjiPa8axNpeTSR2ERjVDiMGf33npx9ayVFopZjXYd92g7RYXmGhSSw"
    }
  ]
]
```

The same content "un-prettified", as it is found in the `did.jsonl` file:

```json
["yza4hoyihukpz5wxcag26clggwqbpsjv26zpp3kqc7gtusokvfpa", 1, "2024-06-18T19:25:55Z", {"hash": "sha3-256", "prerotation": true, "updateKeys": ["z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe"], "nextKeyHashes": ["enkkrohe5ccxyc7zghic6qux5inyzthg2tqka4b57kvtorysc3aa"], "method": "did:tdw:1", "scid": "lkq33irvim6iktjkagnw2ee6bgcw"}, {"value": {"@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/multikey/v1"], "id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"}}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe#z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe", "created": "2024-06-18T19:25:55Z", "proofPurpose": "authentication", "challenge": "yza4hoyihukpz5wxcag26clggwqbpsjv26zpp3kqc7gtusokvfpa", "proofValue": "zUDcd2Yjuzu8XLmZufkYV3Xm8VEZRWgwE9SoKFP83d1ZnKtEwGLf42KCm6nmr8YQVQ6ykusRSSaq8iJjiPa8axNpeTSR2ERjVDiMGf33npx9ayVFopZjXYd92g7RYXmGhSSw"}]]
```

#### `did:web` Version of DIDDoc

As noted in the [publishing a parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification a `did:tdw` can be published
by replacing `did:tdw` with `did:web` in the DIDDoc, adding an `alsoKnownAs` entry for the `did:tdw`
and publishing the resulting DIDDoc at `did.json`, logically beside the `did.jsonl` file.

Here is what the `did:web` DIDDoc looks like for the `did:tdw` above.

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/multikey/v1"
  ],
  "id": "did:web:domain.example:lkq33irvim6iktjkagnw2ee6bgcw",
  "alsoKnownAs": ["did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"]
}
```

### Version 2 of the DIDDoc

Time passes, and the [[ref: DID Controller]] of the `did:tdw` DID decides to
update its DID to a new version, version 2. In this case, the only change
the [[ref: DID Controller]] makes is transition the authorization key to
the pre-rotation key.

#### Version 2 Entry Hashing Input

To generate a new version of the DID, the [[ref: DID Controller]] needs to
provide the existing DID log file, the updated `parameters`, and the new DIDDoc.
The following processing is done to create the new [[ref: DID log entry]]:

- The `entryHash` from the previous (first) log entry is made the first item in the new log entry.
- The `versionId` from the previous (first) log entry is retrieved, incremented and made the new `versionId` (second) item.
- The `versionTime` is generated as the current time, and made the new `versionTime` (third) item.
- The `parameters` entry passed in is processed. In this case, since the
`updateKeys` array is updated, and pre-rotation is active, the a verification is done to ensure that the
hash of the `updateKeys` are found in the previous `nextKeyHashes` item. A new `nextKeyHashes` is included
in the `parameters`.
- The DIDDoc from the previous (first) log entry is extracted and then it and the new DIDDoc are passed into a [[ref: JSON Patch]] implementation to generate the `patch` that is the fourth item of the log entry.
  - This step is not required, and in this case, instead of a patch, the DIDDoc is provided as a `value`, just as with the first log entry.
- The resulting array is passed into the [`entryHash` generation process](#entry-hash-generation-and-validation) which outputs the `entryHash` for this log entry. The first item in the log entry is replaced with this new `entryHash`.
- The data integrity proof is generated added to the log entry as the sixth item, and the entire entry is added to the existing DID log.

The DID log file can now be published, optionally with an updated version of the corresponding `did:web` DID.

The following is the JSON pretty-print log entry for the second version of an example `did:tdw`. Things to note in this example:

- The data integrity proof `verificationMethod` is the `did:key` from the first log entry, and the `challenge` is the `entryHash` from this log entry.
- A new `updateKeys` item in the `parameters` has been added, rotating the key to control future updates to the DID.
- The [[ref: JSON Patch]] content that evolves the DIDDoc from version 1 to version 2.

```json
[
  "laqmqc7hfp4l3ycazz2uiuvhvzh4ce4rrcmsbys7uukyd6ekyf7a",
  2,
  "2024-06-18T19:25:56Z",
  {
    "updateKeys": [
      "z82Lkkmv852dvmqSMN9sFWSPPGAv1UEzfLWjcFFtJPSaLE2gEwjofSHZBE3tKSptU2qYspS"
    ],
    "nextKeyHashes": [
      "rmchj4qvc2lypifk6ziflp4bkqjx7nuvxl7sss2g5rgrhcprv4aq"
    ]
  },
  {
    "value": {
      "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/multikey/v1"
      ],
      "id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"
    }
  },
  [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-jcs-2019",
      "verificationMethod": "did:key:z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe#z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe",
      "created": "2024-06-18T19:25:56Z",
      "proofPurpose": "authentication",
      "challenge": "laqmqc7hfp4l3ycazz2uiuvhvzh4ce4rrcmsbys7uukyd6ekyf7a",
      "proofValue": "zk4dv1FJzQnn8ug4Ga3u8JHf6ne5eFZeK1EmsrkQhccZfmFDADAdmA4NMxsGYikFhMzZRPYdUzqHt3qJ99kk49z77XR6Yb1bfmN53xnt96d7hYXXi6WGEKkT1hrtbbDkg99K"
    }
  ]
]
```

#### Log File For Version 2

The new version 2 `did.jsonl` file contains two entries, one for each version
of the DIDDoc.

```json
["laqmqc7hfp4l3ycazz2uiuvhvzh4ce4rrcmsbys7uukyd6ekyf7a", 2, "2024-06-18T19:25:56Z", {"updateKeys": ["z82Lkkmv852dvmqSMN9sFWSPPGAv1UEzfLWjcFFtJPSaLE2gEwjofSHZBE3tKSptU2qYspS"], "nextKeyHashes": ["rmchj4qvc2lypifk6ziflp4bkqjx7nuvxl7sss2g5rgrhcprv4aq"]}, {"value": {"@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/multikey/v1"], "id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw"}}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe#z82LkqR25TU88tztBEiFydNf4fUPn8oWBANckcmuqgonz9TAbK9a7WGQ5dm7jyqyRMpaRAe", "created": "2024-06-18T19:25:56Z", "proofPurpose": "authentication", "challenge": "laqmqc7hfp4l3ycazz2uiuvhvzh4ce4rrcmsbys7uukyd6ekyf7a", "proofValue": "zk4dv1FJzQnn8ug4Ga3u8JHf6ne5eFZeK1EmsrkQhccZfmFDADAdmA4NMxsGYikFhMzZRPYdUzqHt3qJ99kk49z77XR6Yb1bfmN53xnt96d7hYXXi6WGEKkT1hrtbbDkg99K"}]]
```

#### Log File For Version 3

The same process is repeated for version 3 of the DID. In this case:

- The DIDDoc is changed, and a patch generated.
  - an `authentication` method is added.
  - two services are added.
- No changes are made to the authorized keys to update the DID. As a result, the `parameters` entry is empty (`{}`), and the parameters in effect from previous versions of the DID remain in effect.

Here is the log entry for version 3 of the DID.

```json
["z3mteqfaq5xkcgyhwbgsfaaxzuybt43mcwsrrfzjec2bm67uuvqa", 3, "2024-06-18T19:25:56Z", {}, {"patch": [{"op": "add", "path": "/authentication", "value": ["did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw#z6MknVQ1dPZwBYRsXRtgxF82D5Sz79CCFRNLFwuy5jNo2iou"]}, {"op": "add", "path": "/service", "value": [{"id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw#domain", "type": "LinkedDomains", "serviceEndpoint": "https://domain.example"}, {"id": "did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw#whois", "type": "LinkedVerifiablePresentation", "serviceEndpoint": "https://domain.example/.well-known/whois.jsonld"}]}, {"op": "add", "path": "/assertionMethod", "value": ["did:tdw:domain.example:lkq33irvim6iktjkagnw2ee6bgcw#z6MknVQ1dPZwBYRsXRtgxF82D5Sz79CCFRNLFwuy5jNo2iou"]}, {"op": "add", "path": "/@context/2", "value": "https://identity.foundation/.well-known/did-configuration/v1"}, {"op": "add", "path": "/@context/3", "value": "https://identity.foundation/linked-vp/contexts/v1"}]}, [{"type": "DataIntegrityProof", "cryptosuite": "ecdsa-jcs-2019", "verificationMethod": "did:key:z82Lkkmv852dvmqSMN9sFWSPPGAv1UEzfLWjcFFtJPSaLE2gEwjofSHZBE3tKSptU2qYspS#z82Lkkmv852dvmqSMN9sFWSPPGAv1UEzfLWjcFFtJPSaLE2gEwjofSHZBE3tKSptU2qYspS", "created": "2024-06-18T19:25:56Z", "proofPurpose": "authentication", "challenge": "z3mteqfaq5xkcgyhwbgsfaaxzuybt43mcwsrrfzjec2bm67uuvqa", "proofValue": "zJPZB5z8Ao86FUYUXA7cK6CP6ywAcQPiNHMFX7CCM3BEud2j9ePepZMcFSCuvtthMvx4aXuFiW82bawQQqUNfmRUfggCsqAaig2qtysiYdjA6P63g4F6wEG9pPxYcfMyjQPQ"}]]
```

And so on...
