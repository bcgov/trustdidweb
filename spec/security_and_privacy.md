## Security and Privacy Considerations

### DNS Considerations

#### DNS Security Considerations

Implementers must secure DNS resolution to protect against attacks like Man in
the Middle, following the detailed guidance in the [did:web
specification](https://w3c-ccg.github.io/did-method-web/#dns-considerations).
The use of DNSSEC [[spec:RFC4033]], [[spec:RFC4034]], [[spec:RFC4035]] is
essential to prevent spoofing and ensure authenticity of DNS records.

#### DNS Privacy Considerations

Resolving a `did:tdw` identifier can expose users to tracking by DNS providers
and web servers. To mitigate this risk, it's recommended to use privacy-enhancing
technologies such as VPNs, TOR, or trusted universal resolver services, in line
with strategies outlined in the [did:web
specification](https://w3c-ccg.github.io/did-method-web/#dns-considerations)
including emerging RFCs such as [Oblivious DNS over
HTTPS](https://datatracker.ietf.org/doc/html/draft-pauly-dprive-oblivious-doh-03)
for DNS privacy.

### In-transit Security

For in-transit security, the guidance provided in the [did:web
specification](https://w3c-ccg.github.io/did-method-web/#in-transit-security)
regarding the encryption of traffic between the server and client should be
followed.

### International Domain Names

[[spec:DID-CORE]] identifier syntax does not allow Unicode in method name nor
method specific identifiers.

Implementers should be cautious when implementing support for DID URLs that rely
on domain names or path components that contain Unicode characters.

See also:

- [UTS-46](https://unicode.org/reports/tr46/)
- [[spec:RFC5895]]
