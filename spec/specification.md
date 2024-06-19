## did:tdw DID Method Specification

### Target System

The target system of the Trust DID Web (TDW) DID method is the host (or domain)
name when the domain specified by the DID is resolved through the Domain Name
System (DNS).

### Method Name

The namestring that identifies this DID method is: `tdw`. A DID that uses this
method MUST begin with the following prefix: `did:tdw`. Per the DID
specification, this string MUST be in lowercase. The remainder of the DID, after
the prefix, is the [method-specific identifier](#method-specific-identifier),
specified below.

### Method-specific Identifier

The method specific identifier is a fully qualified domain name that is secured
by a TLS/SSL certificate with an optional path to the [[ref: DID Log]]. The
identifier **MUST** contain a [[ref: self-certifying identifier]] (SCID) as
either a subdomain component of the domain name, or as a component of
the optional path. The content of the [[ref: SCID]] is
[generated](#scid-generation-and-validation) in creating the DID. The formal
rules describing valid domain name syntax are described in [[spec:RFC1035]],
[[spec:RFC1123]], and [[spec:RFC2181]].

The domain name component of the method specific identifier MUST match the
common name used in the SSL/TLS certificate, and it MUST NOT include IP
addresses. A port MAY be included and the colon MUST be percent encoded to
prevent a conflict with paths. Directories and subdirectories MAY optionally be
included, delimited by colons rather than slashes.

The [[ref: SCID]], a globally unique identifier, is generated as part of the
creation of the DID and placed into the DID identifier.

As specified in the following Augmented Backus-Naur Form (ABNF) notation
[[spec:rfc2234]] the [[ref: SCID]] **MUST** be present in the DID string, but
may be placed in the (optional) subdomain component of the domain, or as a
segment in the (optional) URL path. See examples below. The `domain-segment` and
`path-segment` elements refer to [[spec:rfc3986]]'s ABNF for a Generic URL (page
49). Attempting to replicate here the full ABNF of those elements from that RFC
would inevitably be wrong.

```abnf
tdw-did = "did:tdw:" *( domain-segment “.” ) scid 2*( “.” domain-segment ) *( ":" path-segment )
tdw-did = "did:tdw:" 2*( domain-segment “.” ) *( ":" path-segment ) (":" scid ) *( ":" path-segment )
domain-segment = ; A part of a domain name as defined in RFC3986, such as "example" and "com" in "example.com"
path-segment= ; A part of a URL path as defined in RFC3986, such as "path", "to", "folder" in "path/to/folder"
scid = 28+( lower-base32 )
lower-base32 = [2-7a-z]
```

::: example

`did:tdw` DIDs and the corresponding web locations for their `did:tdw` log file.
`{SCID}` is a placeholder for where the generated SCID will be placed in the
examples.

---

subdomain

`did:tdw:{SCID}.example.com` -->

`https://{SCID}.example.com/.well-known/did.jsonl`

path

`did:tdw:example.com:dids:{SCID}` -->

`https://example.com/dids/{SCID}/did.jsonl`

path w/ port

`did:tdw:example.com%3A3000:dids:{SCID}` -->

`https://example.com:3000/dids/{SCID}/did.jsonl`

:::

The location of the `did:tdw` `did.jsonl` [[ref:DID Log]] file is the same as
where the comparable `did:web` `did.json` file is published. A [[ref: DID
Controller]] **MAY** choose to publish both DIDs and so both files. The process
to do so is described in the [publishing a parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification.

### DID Method Operations

#### Create (Register)

Creating a `did:tdw` DID is done by carrying out the following steps.

1. Define the DID string, and hence, the web location at which the DID Log
   (`did.jsonl`) will be published. Identify (using the placeholder `{SCID}`)
   where the required [[ref: SCID]] will be placed in the DID string (ie.
   `did:tdw:example.com:{SCID}`).
2. Generate the authorization key pair(s) that will be used in creating the DID and
   authorizing the first update to the DID (at least).
   1. If the DID is to use [[ref: pre-rotation]], additional processing at this point will be necessary to generate the necessary pre-rotation hashes.
3. For each authorization key pair, generate a [[spec DID-KEY]] DID based on the public key of the key pair.
4. Create the initial DIDDoc (`did.json`) file for the DID, with whatever
   content is required. The top level `id` item **MUST** be included and
   **MUST** be the DID string from step 1. Wherever there is an absolute
   self-reference to the DID in the DIDDoc, use the form defined in step 1, with
   the identified placeholder for the [[ref: SCID]] (ie.
   `did:tdw:example.com:{SCID}#key-1`). The public key(s) of the authorization key
   pair(s) **MAY** be placed into the DIDDoc, but that is not required.
5. Define a JSON array of valid [[ref: parameters]] that affect the generation
   of the DID. The [DID Generation and Validation
   Parameters](#didtdw-did-method-parameters) section of this specification
   defines the permitted [[ref: parameters]]. That section defines what items
   **MUST** be included in this first log entry for the DID.
6. Pass the DID string created in step 1, the initial DIDDoc, and [[ref: parameters]] to a `did:tdw`
   "Create" process that **MUST**:
   1. Verify that the DIDDoc contains the a top level `id` item whose value
      is the same as the specific DID string, and verify that it is a valid
      `did:tdw` DID that contains the `{SCID}` placeholder in an allowed
      location in the DID string as per the ABNF of a `did:tdw` DID as defined
      in the [Method-Specific Identifier](#method-specific-identifier) section
      of this specification.
   2. Calculate the [[ref: SCID]] for the DID as defined in the [SCID Generation
      and Validation](#scid-generation-and-validation) section of this
      specification.
   2. Replace in the DIDDoc the placeholder for the [[ref: SCID]] `{SCID}` with
      the calculated `SCID`.
   3. Generate a DID Entry as a JSON array with the following five JSON items:
      1. The [[ref: SCID]] as the `entryHash` value.
      2. An integer, `1`, that is the `versionId` for this first version of the
         DIDDoc.
      3. A string that is the current time in [[ref: ISO8601]] format (e.g.,
         `"2024-04-04T07:32:58Z"`) that is the `versionTime` for the first version of the DID.
      4. The input [[ref: parameters]] passed in as a JSON dict.
      5. The contents of the initial DIDDoc, in the form: `{"value": <DIDDoc>}`
   4. Calculate the [[ref: Entry Hash]] (`entryHash`) of the DID Entry as
      defined in the [Entry Hash Generation and
      Validation](#entry-hash-generation-and-validation) section of this
      specification.
   5. Update the value of the `entryHash` item with the value produced in the previous step.
   6. Generate a [[ref: Data Integrity]] proof on the initial DIDDoc using an
      authorized key from a [[spec DID KEY]] DID in the required `updateKeys` item in the
      [[ref: parameters]], and the `entryHash` as the proof `challenge`.
      The proof becomes the sixth and last JSON item in the DID log entry.
   7. Put the resulting entry, with extraneous white space removed as the
      contents of a file `did.jsonl`.
	8. Publish the file at the appropriate location defined by the `did:tdw` DID identifier.
        - This is a logical operation -- how a deployment serves the `did.jsonl`
          content is not constrained.

A controller **MAY** generate an equivalent `did:web` DIDDoc and publish it as
defined in the [Publishing a Parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification. The
`did:web` DIDDoc could be used for backwards compatibility as a transition is
made from `did:web` to `did:tdw`. Verifiers using the `did:web` lose the
verifiable properties and history of the `did:tdw` for the convenience of the
simple retrieval of the `did:web` DIDDoc.

#### Read (Resolve)

The following steps MUST be executed to resolve the DIDDoc for a `did:tdw` DID:

1. Remove the literal `did:tdw` prefix from the DID.
2. Replace `:` with `/` in the method specific identifier to obtain the fully
   qualified domain name and optional path.
3. If the domain contains a port percent decode the colon.
4. Generate an HTTPS URL to the expected location of the DIDDoc by prepending
   `https://`.
5. Append `/did.jsonl` to complete the URL.
6. Perform an HTTP GET request to the URL using an agent that can successfully
   negotiate a secure HTTPS connection, which enforces the security requirements
   as described in [Security and privacy
   considerations](#security-and-privacy-considerations).
7. When performing the DNS resolution during the HTTP GET request, the client
   SHOULD utilize [[spec:rfc8484]] in order to prevent tracking of the identity
   being resolved.
8. Process the [[ref: DID Log]] file as described below.

To process the retrieved [[ref: DID Log]] file, the resolver **MUST** carry out
the following steps:

1. Process the Log entries in the order they appear in the file, applying the
   [[ref: parameters]] set from the current and previous entries. As noted in the
   [Create (Register)](#create-register), each log entry consists of a JSON
   array of 6 items:
   1. `entryHash`
   2. `versionId`
   3. `versionTime`
   4. `parameters`
   5. DIDDoc content -- either the full `value` or a [[ref: JSON Patch]] `patch`
      to be applied to the prior version of the DIDDoc.
   6. A Data Integrity proof for the current version of the DIDDoc of
      the entry.
2. For each entry:
   1. Update the currently active [[ref: parameters]] with the parameters from
      the entry (if any). Continue processing using the now active set of [[ref:
      parameters]].
   2. Verify the Data Integrity proof in the entry, and ensure it is signed by
      an authorized key as defined in the [Authorized Keys](#authorized-keys)
      section of this specification.
   3. Verify the `entryHash` for the entry using the process defined in the
      [Entry Hash Generation and
      Verification](#entry-hash-generation-and-validation) section of this
      specification.
   4. Verify that the first log entry's `versionId` is `1`, and that the `versionId` is incremented by one for each subsequent log entry.
   5. Verify that the `versionTime` for each log entry is greater than the previous entry, and that the `versionTime` values are all earlier than the current time.
   6. For the initial version of the DIDDoc (`1`) verify that the [[ref: SCID]]
      (defined in the [[ref: parameters]]) is being used in the DID, and
      verifies according to the [SCID Generation and
      Verification](#scid-generation-and-validation) section of this
      specification.
   7. Generate the version of the DIDDoc for the entry by using the JSON value
      of the `value` item, or by using [[ref: JSON Patch]] to apply the JSON
      value of the `patch` entry item to the previous version of the DIDDoc.
   8. If [[ref: Key Pre-Rotation]] is being used, verify that any `updateKeys`
      in the `parameters` item have a valid pre-rotation entry in the active
      array of `nextKeyHashes` [[ref: parameter]] as defined
      in the [Key Pre-Rotation Hash Generation and
      Verification](#pre-rotation-key-hash-generation-and-validation) section of
      this specification.
   9. If any verifications fail, discard the DID as invalid.
   10. As each log entry is processed and verified, collect the following information
      about each version:
         1. DIDDoc.
         2. `versionId` of the DIDDoc.
         3. `versionTime`of the DIDDoc.
         4. The latest list of active [[spec DID-KEY]] DIDs authorized to update the DID, from the `updateKeys` lists in the [[ref: parameters]].
         5. If pre-rotation is being used, the hashes of [[spec DID KEY]] DIDs that will be used in later `updateKeys` lists. The pre-rotation hashes are in the `nextKeyHashes` list in the [[ref: parameters]].

On completing the processing and successful verification of all entries in the [[ref: DID Log]], respond to
the DID resolution request, including the application of query parameters such
as `?versionId=` and `?versionTime=` with the appropriate DIDDoc version.

The following error codes and descriptions may be returned when resolving a DID.

:::todo

Document the full list of error codes that can be generated in resolving a DID.

:::

- Code 404: The `did:tdw` DID Log file `did.jsonl` was not found.

##### Reading did:tdw DID URLs

A `did:tdw` resolver **MAY** implement the resolution of the `/whois` and a DID
URL Path using the [whois LinkedVP Service](#whois-linkedvp-service) and [DID
URL Path Resolution Service](#did-url-path-resolution-service) as defined in
this specification by processing the [[ref: DID Log]] and then dereferencing the
DID URL based on the contents of the DIDDoc. The client of a resolver that does
not implement those capabilities must use the resolver to resolve the
appropriate DIDDoc, and then process the resulting DID URLs themselves. Since
the default DID-to-HTTPS URL transformation is trivial, `did:tdw` [[ref: DID
Controllers]] are strongly encouraged to use the default behavior for DID URL
Path resolution.

#### Update (Rotate)

To update a DID a new, verifiable [[ref: DID Log Entry]] must be generated,
appended to the existing [[ref: DID Log]] (`did.jsonl`) and published to the
web location defined by the DID. The process to generate a verifiable [[ref: DID
Log Entry]] follows a similar process to the [Create](#create-register) process,
as follows:

1. Make the desired changes to the DIDDoc. While the contents of a new DIDDoc
   version are (mostly) up to the DID controller, there are some limitations:
   1. The `id` of the DIDDoc **MAY** be changed when the DID Controller wants to
      (or is forced to) publish the DID at a different location and wants to retain the
      [[ref: SCID]] and history of the DID. For details, see the section [Moving
      a DID's Web Location](#moving-a-dids-web-location).
2. Define a JSON array of valid [[ref: parameters]] that affect the evolution of
   the DID. The [`did:tdw` DID Method Parameters](#didtdw-did-method-parameters)
   section of this specification defines the permitted [[ref: parameters]].
3. Pass the current [[ref: DID Log]], the updated DIDDoc, and the [[ref:
   parameters]] to a `did:tdw` update implementation which **MUST**:
   1. Generate a DID Entry as a JSON array with the following JSON items:
      1. The `entryHash` from the previous [[ref: DID Log Entry]] as the
         `entryHash` value.
      2. An integer that is one more than the `versionId` of the previous [[ref:
         DID Log Entry]].
      3. A string that is the current time in [[ref: ISO8601]] format.
      4. The [[ref: parameters]] passed in as a JSON dict.
         1. [[ref: parameters]] from previous versions continue to apply and do
            not need to be repeated in each version. As a result, the
            `parameters` item may be an empty dict (`{}`).
      5. Generate a [[ref: JSON Patch]] to evolve the previous DIDDoc version to
         the new DIDDoc version, and put the resulting patch in the item
         `{"patch": <DIDDoc Patch>}`. For details see the [Generating and
         Applying a JSON Patch](#generating-and-applying-a-json-patch) section
         of this specification.
         1. An implementation **MAY** skip the [[ref: JSON Patch]] process and
            simply put the full new version of the DIDDoc in the item `{"value":
            <DIDDoc>}` as is done in the initial entry in the log.
   2. Calculate the [[ref: Entry Hash]] (`entryHash`) of the DID Entry as
      defined in the [Entry Hash Generation and
      Validation](#entry-hash-generation-and-validation) section of this
      specification.
   3. Update the value of the `entryHash` item with the value produced in the previous step.
   4. Generate a [[ref: Data Integrity]] proof on the new DIDDoc of the entry
      using an authorized key, and the `entryHash` as the proof
      `challenge`. The definition of "authorized" is formalized in the
      [Authorized Keys](#authorized-keys) section of this specification. The
      proof becomes the last JSON item in the entry.
   5. Append the resulting entry to the existing contents of the [[ref: DID
      Log]] file `did.jsonl` on a new line.
4. Publish the updated [[ref: DID Log]] file at the appropriate location defined by the
      `did:tdw` identifier.
      - This is a logical operation -- how a deployment serves the `did.jsonl`
      content is not constrained.

A controller **MAY** generate an equivalent, updated `did:web` DIDDoc and
publish it as defined in the [Publishing a Parallel `did:web`
DID](#publishing-a-parallel-didweb-did) section of this specification.

A controller may use the `move` [[ref: DID log entry]] [[ref: parameter]] to
change the DID string. That action effectively creates a new DID, but a DID that
retains the [[ref: SCID]] and verifiable history of the original DID. A [[ref:
DID Controller]] may do that when forced (for example, loss of domain name) or
by choice (moving from a DID hosting provider to another) without losing the
history and reputation of the DID. See the [Moving a DID](#moving-a-did) section
of the Implementers Guide for a discussion of moving a `did:tdw` DID.

#### Deactivate (Revoke)

To deactivate the DID, the [[ref: DID Controller]] **SHOULD** add to the [[ref:
DID log entry]] [[ref: parameters]] the item `"deactivated": true`. A [[ref: DID
Controller]] **MAY** update the DIDDoc further to indicate the deactivation of
the DID, such as including an empty `updateKeys` list (`"updateKeys": []`) in the [[ref: parameters]],
preventing further versions of the DID.

A resolver encountering in the [[ref: DID log entry]] [[ref: parameters]] the
item `"deactivated": true` should return in the DIDDoc Metadata the JSON item
`"deactivated": true`, as per the [[spec:DID-CORE]] specification.

### DID Method Processes

The [DID Method Operations](#did-method-operations) reference several processes
that are executed during DIDDoc generation and DID resolution verification. Each
of those processes is specified in the following sections.

#### `did:tdw` DID Method Parameters

Entries in the `did:tdw` [[ref: DID Log]] file contain, as the 4th item, a JSON
object that define the DID processing parameters being used by the [[ref: DID
Controller]] when publishing that and subsequent DID Entries. A DID Resolver
will use the same parameters when processing the [[ref: DID Log]] to resolve the
DID. The parameters object **MUST** only include items defined in this
specification.

::: example

An example of the parameters item in the first DID Log entry for a DID:

`{"method":"did:tdw:1","scid":"4c99uuenu8gk6n3bgf09fuf350gx"}`

:::

The permitted parameter items and (where applicable) enumerated values for those
items are defined below.

- `method`: Defines the version of the [[ref: DID Log]] processing specification
  to use when processing a given DID's log file. As new versions of the
  processing specifications are defined, additional values will be added to the
  list of acceptable values.
  - This item **MUST** appear in the first [[ref: DID Log]] entry.
  - This item **MAY** appear in later [[ref: DID Log]] entries to indicate that
    the processing rules for that and later entries have been changed to a
    different version.
  - Acceptable values for this specification are:
    - `did:tdw:1`: Requires that the rules defined in this specification be used
      in processing the log.
- `scid`: The value of the [[ref: SCID]] for this DID.
  - This item **MUST** appear in the first [[ref: DID log entry]].
- `updateKeys`: A list of one or more [[spec DID KEY]] DIDs that are authorized
  to sign the log entries that update the DID from one version to the next. An instance of the list in an entry replaces the previously active list. If an entry does not have the `updateKeys` item, the currently active list continues to apply. See the
  [Authorized Keys](#authorized-keys) section of this specification for additional details.
  - This item **MUST** appear in the first [[ref: DID log entry]].
- `hash`: The hashing algorithm to use when executing hashes.
  - By default, the value is initialized to `sha256`.
  - Acceptable values:
    - `sha256`: Use the `SHA-256` algorithm from [[spec:rfc4634]].
- `cryptosuite`: The Data Integrity cryptosuite to use when generating and verifying the authentication proofs on the [[ref: DID log entries]].
  - By default, the value is initialized to `eddsa-jcs-2022`
  - Acceptable values:
    - `eddsa-jcs-2022`: Use the [eddsa-jcs-2022](https://www.w3.org/TR/vc-di-eddsa/#eddsa-jcs-2022) cryptosuite.
- `prerotation`: A boolean value indicating that subsequent authentication keys
  added to the DIDDoc (after this version) **MUST** have their hash included in
  a `nextKeyHashes` parameter item.
  - The value is initialized to `false` until the item is included in an [[ref:
    DID log entry]].
  - Once the value is set to `true` in a [[ref: DID log entry]] it **MUST NOT**
    be set to `false` in a subsequent entry.
- `nextKeyHashes`: An array of strings that are hashes of [[spec DID KEY]] DIDs that will be added to the `updateKeys` list in a future version of the DID.
  - The process for generating the hashes is defined in the [Pre-Rotation Key
    Hash Generation and
    Validation](#pre-rotation-key-hash-generation-and-validation) section of
    this specification.
  - If the parameter `prerotation` has been set to `true`, all [[spec DID KEY]] DIDs added to `updateKeys` lists **MUST** have a corresponding hash 
    listed in the `nextKeyHashes` items from a previous [[ref: DID log entries]].
  - See the section of this specification [Using Pre-Rotation
    Keys](#using-pre-rotation-keys) for non-normative guidance in using
    pre-rotation keys.
- `moved`: A string that is a new `did:tdw` DID string. The new `did:tdw`:
  - **MUST** contain the same [[ref: SCID]].
  - The accompanying DIDDoc **MUST** have the new DID as the value of the
    top-level `id`.
  - The accompanying DIDDoc **MUST** have an `alsoKnownAs` array with all
    previous DID strings as entries in the array.
- `deactivated`: A JSON boolean that should be set to `true` when the DID is to
  be deactivated. See the [deactivate (revoke)](#deactivate-revoke) section of
  this specification for more details.
- `ttl`: A number, the number of seconds that a cache entry for a resolved
  `did:tdw` DID should last, as recommended by the [[ref: DID Controller]]. A
  resolver can use this value in deciding whether to retrieve a new version of
  the DID's `did.jsonl` file. If not specified, resolvers may set a default
  based on the business needs of the resolver clients.
  - Caching of a `did:tdw` can be valuable in places where the business rules
    require resolving a number of DID URLs for the same DID. For example, a
    client might want call the resolver to the current DIDDoc, and then make
    repeated calls to get all of the previous versions of the DIDDoc. By caching
    the DIDDoc state, the resolver would not have to retrieve and process the
    [[ref: DID Log]] on every call.
  - A Web Server handling one or more `did.jsonl` files **MAY** be configured to
   use a comparable HTTP TTL per [[spec-inform:rfc9111]].

#### SCID Generation and Validation

The [[ref: Self-certifying identifier]] or `scid` is a required parameter in the
first [[ref: DID log entry]] and is a portion of the hash of the DID's inception event.

##### Generate SCID

To generate the required [[ref: SCID]] for a `did:tdw` DID, the DID Controller
**MUST** execute the following function:

 `left(base32_lower(hash(JCS(partial initial log entry with placeholders))), <length>)`

Where:

1. The `partial initial log entry with placeholders` consists of the following elements of
   the first log entry, as a JSON Lines array. The placeholder is the literal string "`{SCID}`".

   - The `entryHash` as a placeholder.
   - The `versionId` entry, which must be `1`.
   - The `versionTime` entry, which must be a string that is the current time in [[ref: ISO8601]] format, e.g.,
         `"2024-04-05T07:32:58Z"`
   - The complete `parameters` for the entry, with the placeholder wherever the SCID will eventually be placed.
   - The `{"value": <DIDDoc>}` element with the placeholders wherever the SCID will eventually be placed.
2. `JCS` is an implementation of the [[ref: JSON Canonicalization Scheme]]
   [[spec:rfc8785]]. It outputs a canonicalized representation of its input.
3. `hash` is either `sha256` or an alternative hash algorithm defined in the
   `hash` item in the [[ref: parameters]]. Its output is the hash of its input.
4. `base32_lower` as defined by the [[ref: base32_lower]] function. Its output
   is the lower case of the Base32 encoded string of its input.
5. `left` extracts the `<length>` number of characters from the string input.
   1. `<length>` **MUST** be at least 28 characters.

##### Verify SCID

To verify the [[ref: SCID]] of a `did:tdw` DID being resolved, the resolver
**MUST** execute the following process:

1. Extract from the [[ref: parameters]] in the first [[ref: DID log entry]] for
   the DID the `scid` item's value.
2. Verify that the length of the `scid` is at least 28 characters.
   1. If less than 28 characters, terminate the resolution process with an
      error.
3. Remove from the first [[ref: DID log entry]] the data integrity proof.
4. Replace the `entryHash` value with the placeholder literal "`{SCID}`"
4. Treat the resulting log entry as a string and do a text replacement of the `scid`
   from the first step with the literal string `{SCID}`.
5. Execute the hashing process defined in the generation defined above to
   generate the value `calculatedSCID`.
   1. For the `<length>` value, use the length of the `scid` extracted in step
6. Verify that the `scid` matches the `calculatedSCID`.

#### Entry Hash Generation and Validation

The `entryHash` is the first item in each [[ref: DID log entry]] and is a hash
calculate across the entry, excluding the [[ref: Data Integrity]] proof.

##### Generate Entry Hash

To generate the required hash for a `did:tdw` DID entry, the DID Controller
**MUST** execute the following process:

1. Make the value of the `entryHash` for this entry to the `entryHash` of the
   previous entry. For the first entry in the log, set the value of `entryHash`
   to the [[ref: SCID]] for the DID.
2. Create the full entry JSON array with elements `entryHash`, `versionId`,
   `versionTime`, and `didDocVersion` (which is either a `value` or `patch`).
   See an example below.
3. Calculate the hash string as `base32_lower(hash(JCS(entry)))`, where:
   1. `JCS` is an implementation of the [[ref: JSON Canonicalization Scheme]]
      ([[spec:rfc8785]]). Its output is a canonicalized representation of its
      input.
   2. `hash` is either `sha256` or an alternative hash algorithm defined in the
      `hash` item in the [[ref: parameters]]. Its output is the hash of its
      input JCS content.
   3. `base32_lower` as defined by the [[ref: base32_lower]] function. Its
      output is the lower case of the Base32 encoded string of the input hash.
4. Replace the `entryHash` value in the entry with the calculated value from
   step 3.

Example of a [[ref:DID log entry]] that is processed in step 3 to produce a hash. As
this is a first entry in a DID Log, the `entryHash` is the SCID of the DID.

```json
["4c99uuenu8gk6n3bgf09fuf350gx",1,"2024-04-15T19:56:18Z",{"method": "did:tdw:1","scid": "4c99uuenu8gk6n3bgf09fuf350gx"},{"value": {"@context": ["https://www.w3.org/ns/did/v1","https://w3id.org/security/multikey/v1"],"id": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx","controller": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx","authentication": ["did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx#y4SDXopT"],"assertionMethod": ["did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx#5b48Zj6B"],"verificationMethod": [{"id": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx#y4SDXopT","controller": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx","type": "Multikey","publicKeyMultibase": "z6Mksta2t7db1WSx2JBorfYFcJnaJMBKUyupD2qPy4SDXopT"},{"id": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx#5b48Zj6B","controller": "did:tdw:example.com:4c99uuenu8gk6n3bgf09fuf350gx","type": "Multikey","publicKeyMultibase": "z6Mkw1KSvGWNAwSwWbcpwPgFARX4vKPa1xvcDMsJ5b48Zj6B"}]}}]
```

Resulting entry hash: `4fbja27mgf0bumtbg2b4hbzqc2ux9a9crrqx7w6cfnd97k9u7k5g`

##### Verify Entry Hash

To verify the `entryHash` for a `did:tdw` DID entry, a DID Resolver **MUST**
execute the following process:

1. Extract the first item in the [[ref: DID log entry]] as the `entryHash`
2. Remove the [[ref: Data Integrity]] proof (5th item) from the entry array.
3. Set the first value of the entry to the `entryHash` of the previous entry. If
   this is the first entry in the log, set the value to the [[ref: SCID]] for
   the DID.
4. Calculate the hash string as `base32_lower(hash(JCS(entry)))`, where:
   1. `JCS` is an implementation of the [[ref: JSON Canonicalization Scheme]]
      ([[spec:rfc8785]]). Its output is a canonicalized representation of its
      input.
   2. `hash` is either [sha256] or an alternative hash algorithm defined in the
      `hash` item in the [[ref: parameters]]. Its output is the hash of its
      input JCS content.
   3. `base32_lower` as defined by the [[ref: base32_lower]] function. Its
      output is the lower case of the Base32 encoded string of the input hash.
5. Verify that the calculated value from Step 4 matches the extracted value from
   Step 1.

#### Authorized Keys

Each entry in the [[ref: DID Log]] **MUST** include a [[ref: Data Integrity]] proof signed by a key **authorized** to control (create, update, deactivate) the DID. For `did:tdw`, the authorized keys are those referenced by the list of [[spec DID-KEY]] DIDs in the **active** `updateKeys` list in the `parameters` item of the [[ref: log entries]].

For the first [[ref: log entry]] the **active** `updateKeys` list is the one in that first [[ref: log entry]]. For all subsequent entries, **active** means
the most recent `updateKeys` list **before** the entry to be signed is created. Thus, the general case is that each log entry is signed by the keys from the **previous** log entry.

A resolver of the DID **MUST** verify that the key used for signing the [[ref:
DID Log]] entry is one from a [[spec DID KEY]] DID in the list of currently active `updateKeys` [[ref: parameter]], and that the signature verifies.

The `did:tdw` Implementation Guide contains further discussion on the management of keys authorized to update the DID.

::: note

Note: The initial approach we considered using to find an authorized key was to
use key references in the DIDDoc itself. However, we struggled to understand how
to apply the guidance in the [[spec: DID-CORE]] specification in determining the
appropriate key(s) in the DIDDoc to use, and so decided instead to use the
mechanism specified in this section. A future version of this specification
could revert that decision if the [[ref: DID Core]] guidance is clarified, or if
an appropriate interpretation of the existing guidance is agreed upon.

One scenario in particular that we did not think is satisfied by the [[spec:
DID-CORE]] specification is in determining the authorized keys when the
`controller` of the DID is not the DID itself, and so might need to be found by
(potentially recursively) resolving other DIDs. We felt it a requirement that in
resolving and verifying a `did:tdw` DID, resolvers must be able to access all
necessary public keys in the [[ref: DID Log Entries]] themselves, and not have
to retrieve them after retrieving the [[ref: DID Log]]. The approach outlined in
this section ensures that is the case for `did:tdw`.

:::

#### Generating and Applying a JSON Patch

Each time a new `did:tdw` version is created, the [[ref: DID Controller]]
**MAY** generate a [[ref: JSON Patch]] to concisely define the changes in the
DIDDoc from the previous version. A [[ref: DID log entry]] that uses [[ref: JSON Patch]]
has a JSON object with a `patch` property, with the value the [[ref:
JSON Patch]] as its 5th item. A [[ref: DID Controller]] **MAY** set the
fourth item of a [[ref: DID log entry]] to be the JSON item `value`, with its
value the complete DIDDoc. Typically (but not required), a [[ref: DID Controller]]
will use `value` for the first [[ref: DID log entry]] and `patch` for all
subsequent entries.

To create the value for a `patch` item for a [[ref: DID log entry]], the [[ref:
DID Controller]] **MUST**:

1. Have the fully resolved previous version of the DIDDoc.
2. Have the updated new version of the DIDDoc to be added.
3. Execute an implementation of [[ref: JSON Patch]] that takes the two DIDDocs
   as inputs (previous before, new after) and outputs the resulting [[ref: JSON Patch]]
   from before to after.
4. Set the fourth item of the [[ref: DID log entry]] to `{"patch": "<patch>"}`,
   removing all extraneous whitespace from the `<patch>`.

When processing a [[ref: DID log entry]] with a `patch` , a resolver **MUST**:

1. Have the fully resolved previous version of the DIDDoc.
2. Execute an implementation of [[ref: JSON Patch]] that takes the previous
   DIDDoc and the patch as inputs, and outputs the resulting new version of the
   DIDDoc.

#### Publishing a Parallel `did:web` DID

Each time a `did:tdw` version is created, the [[ref: DID Controller]] **MAY**
generate a corresponding `did:web` to publish along with the `did:tdw`. To do
so, the [[ref: DID Controller]] **MUST**:

1. Start with the resolved DIDDoc from `did:tdw`.
2. Execute a text replacement across the DIDDoc of `did:tdw` to `did:web`.
3. Add to the DIDDoc `alsoKnownAs` array, the full `did:tdw` DID. If the
   `alsoKnownAs` array does not exist in the DIDDoc, it **MUST** be added.
4. Publish the resulting DIDDoc as the file `did.json` at the web location
   determined by the specified `did:web` DID to HTTP transformation.

Note that the [[ref: SCID]] remains in the `did:web` DID string.

The benefit of doing this is that resolvers that have not been updated to
support `did:tdw` can continue to resolve the [[ref: DID Controller]]'s DIDs.
`did:web` resolvers that are aware of `did:tdw` features can use that knowledge,
and the existence of the `alsoKnownAs` `did:tdw` data in the DIDDoc to get the
verifiable history of the DID.

The risk of publishing the `did:web` in parallel with the `did:tdw` is that the
added security and convenience of using `did:tdw` are lost.

#### Pre-Rotation Key Hash Generation and Validation

Pre-rotation is a term defining how a [[ref: DID Controller]] can commit to the
authorization keys that will be used ("rotated to") when updating the DIDDoc.
The purpose of committing to future keys is that if the currently authorized
keys are compromised by an attacker, the attacker should not be able to rotate
the compromised keys to new ones only the attacker controls and take over
control of the DID. The effectiveness of pre-rotation is based on the idea that
an attacker cannot compromise the future keys. See the non-normative section
about [Using Pre-Rotation Keys](#using-pre-rotation-keys) in the implementer's
guide section of this specification.

As described in the [parameters](#did-generation-and-validation-parameters)
section of this specification, a [[ref: DID Controller]] **MAY** define that
`prerotation` is active for the DID. When that is the case, all of the [[spec DID KEY]]s in the
`updateKeys` parameters item in future versions of the DIDDoc **MUST** have their hash in one or
more of the `nextKeyHashes` arrays of previous [[ref: DID log entry]] parameters.

To create a hash to be included in the nextKeyHashes array, the [[ref: DID Controller]]
**MUST** execute the following process:

1. Generate a new key pair.
2. Generate a [[spec DID KEY]] based on the public key of the new key pair.
3. Calculate the hash string as `base32_lower(hash(did:key DID))`,
   where:
   1. `hash` is either `sha256` or an alternative hash algorithm defined in the
      `hash` item in the [[ref: parameters]]. Its output is the hash of its
      input JCS content.
   2. `base32_lower` as defined by the [[ref: base32_lower]] function. Its
      output is the lower case of the Base32 encoded string of the input hash.
4. Add the hash calculated in Step 3 to a [[ref: DID log entry]] `nextKeyHashes` item
   in the [[ref: parameters]] item (4th item of the entry array).
5. The [[spec DID KEY]] from Step 2 can be inserted into the `updateKeys` item
   in the [[ref: parameters]] item of a future [[ref: log entry]].

When processing a [[ref: DID log entry]] where the `prerotation` parameter is
active, a resolver **MUST**:

1. When processing each [[ref: DID log entry]], after verifying a DIDDoc, add an
   optional array of `nextKeyHashes` from the parameters into an array of hash
   strings.
   1. The collection of the `nextKeyHashes` from the current entry must occur after
      the processing and verification of that entries` DIDDoc version so that a
      new key and its pre-rotation hash cannot be added in the same entry.
2. For all DIDDoc versions after versionId 1, if there is an `updateKeys` item, verify that it contains a list of [[spec DID KEY]] DIDs, and for each:
   1. Calculate a hash string as
      `base32_lower(hash(did:key DID))`, where:
      1. `hash` is either `sha256` or an alternative hash algorithm defined in
         the `hash` item in the [[ref: parameters]]. Its output is the hash of
         its input [[spec DID KEY]].
      2. `base32_lower` as defined by the [[ref: base32_lower]] function. Its
         output is the lower case of the Base32 encoded string of the input
         hash.
   3. Check to see if the hash string is listed in the collected list of
      `nextKeyHashes`.
      1. If so, the new key is verified.
      2. If not, the verification process (and resolution) failed.

### DID URL Resolution

The `did:tdw` DID Method embraces the power and usefulness of DID URLs, along
with the semantic simplicity of using them in a web-based DID method.
Specifically, a `did:tdw` implementation **MUST**:

- Resolve the `/whois` DID URL path using a [[spec:LINKED-VP]] service, whether
  or not it exists in the `did:tdw` DIDDoc, returning a [[ref: Verifiable
  Presentation]], if published by the [[ref: DID Controller]], found at the same
  path as the `did.jsonl` file is found with `/whois.json` appended to it.
  - For example, `did:tdw:{SCID}.example.com/whois` returns the verifiable
    presentation from `https://{SCID}.example.com/.well-known/whois.json`.
- Resolve any `did:tdw` DID URL using a [[spec:DID-CORE]] `relativeRef` DID
  parameter, whether or not a supporting service exists in the `did:tdw` DIDDoc,
  returning the file, found at the web location relative to where the
  `did.jsonl` file is found.
  - For example, `did:tdw:{SCID}.example.com/governance/issuers.json` returns
    the file `https://{SCID}.example.com/.well-known/governance/issuer.json`

In both cases, a [[ref: DID Controller]] **MAY** define services in the DIDDoc
that override the default services that **MUST** be resolved by the `did:tdw`
DID Method.

The sections below formalize the services that exist by default in `did:tdw` and
how a [[ref: DID Controller]] can override them.

#### whois LinkedVP Service

The `#whois` service enables those that receive a `did:tdw` DID to retrieve and
a [[ref: Verifiable Presentation]] (and embedded [[ref: Verifiable
Credentials]]) the [[ref: DID Controller]] has decided to publish about itself.
The intention is that anyone interested in a particular `did:tdw` DID can
resolve the `<did>/whois` DID URL, to retrieve a [[ref: Verifiable
Presentation]] published by the [[ref: DID Controller]] that contains [[ref:
Verifiable Credentials] about the DID that might be useful to the resolver in
learning more about who is the controller of the DID and if they should be
trusted. It is up to the [[ref: DID Controller]] to decide to publish a `whois`
verifiable presentation, along with which verifiable credentials to put into the
verifiable presentation.

See the [The `/whois` Use Case](#the-whois-use-case) in this specification for
the background about why this capability is so useful, particularly for a
web-based DID Method like `did:tdw`.

`did:tdw` DIDs **automatically** supports a `#whois` service endpoint with the
following definition based on the [[ref: Linked VP]] specification, with the
`serviceEndpoint` matching the `did:tdw` DID-to-HTTPS DIDDoc transformation and
`did.jsonl` changed to `whois.json`.

```json
{
   "@context": "https://identity.foundation/linked-vp/contexts/v1",
   "id": "#whois",
   "type": "LinkedVerifiablePresentation",
   "serviceEndpoint": "https://example.com/dids/<scid>/whois.json"
}
```

The returned `whois.json` **MUST** contain a [[ref: W3C VCDM]] [[ref: verifiable
presentation]] signed by the DID and containing [[ref: verifiable credentials]]
with the DID as the `credentialSubject`.

A [[ref: DID Controller]] **MAY** explicitly add to their DIDDoc a `did:tdw`
service with the `"id": "#whois"`. Such an entry **MUST** override the implicit
`service` above. If the [[ref: DID Controller]] wants to publish the `/whois`
[[ref: verifiable presentation]] in a different format than the [[ref: W3C
VCDM]] format, they **MUST** explicitly add to their DIDDoc a service with the
`"id": "#whois"` to specify the name and implied format of the [[ref: verifiable
presentation]].

To resolve the DID URL `<did:tdw DID>/whois`, the resolver **MUST**:

1. Resolve the given `did:tdw` DID by retrieving, processing, and verifying the
   [[ref: DID log]] for the `did:tdw` as defined in this specification.
2. Find the DIDDoc `service` with the `id` `#whois`, if any, or use the implicit
   service (above).
3. Resolve the `serviceEndpoint` URL, if possible, and return the document
   found.
   1. If the `serviceEndpoint` URL can't be resolved by the resolver (such as if
      the URL protocol is not supported by the resolver), the error `Error XXX:
      YYY` **MUST** be returned.
   2. If the file at the defined `serviceEndpoint` is not found, `Error
      404: Not Found` **MUST** be returned.

#### DID URL Path Resolution Service

The automatic resolution of `did:tdw` DID URL paths follows the
[[spec:DID-CORE]] `relativeRef` specification, as follows:

- a DID path, such as example 2 in [section 3.2 DID URL
  Syntax](https://www.w3.org/TR/did-core/#example-2) gives the example:
  `did:example:123456/resume.pdf`
- In turn, that can be treated as the following, as shown in example 8 in the
  same section: `did:example:123456?service=files&relativeRef=/resume.pdf`
- The `#files` service defined below then defines the `serviceEndpoint` for the
  `relativeRef`.
  - For `did:tdw`, that service is implicitly defined, with the
    `serviceEndpoint` matching the `did:tdw` DID-to-HTTPS DIDDoc transformation
    and `did.jsonl` replaced by the DID URL Path.

Thus, the implicit service for DID `did:tdw:example.com:dids:<scid>` is:

```json
{
   "id": "#files",
   "type": "relativeRef",
   "serviceEndpoint": "https://example.com/dids/<scid>"
}
```

A [[ref: DID Controller]] **MAY** explicitly add to their DIDDoc a service with
the `"id": "#files"`. Such an entry **MUST** override the implicit `service`
defined above.

To resolve the DID URL `<did:tdw DID>/path/to/file`, the resolver **MUST**:

1. Resolve the given `did:tdw` DID by retrieving, processing, and verifying the
   [[ref: DID log]] for the `did:tdw` as defined in this specification.
2. Find the DIDDoc `service` with the `id` `#files`, if any, or use the implicit
   service (above).
3. Resolve the `serviceEndpoint` URL with the DID URL Path appended, if
   possible, and return the document found at that location.
   1. If the `serviceEndpoint` URL can't be resolved by the resolver (such as if
      the URL protocol is not supported by the resolver), the error `Error XXX:
      YYY` **MUST** be returned.
   2. If the file at the path appended to the defined `serviceEndpoint` is not
      found, the error `Error 404: Not Found` **MUST** be returned.
