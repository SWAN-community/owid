![Open Web Id](images/owl.128.pxls.100.dpi.png)

# Open Web Id (OWID)

## Abstract

Many data audit solutions require a cryptographically verifiable method of
validating the source of data, and the entities who processed that data. Records
of the data being sent or received, or just the meta-data of the parties, are
not verifiable and can easily be faked. As such existing logs are open to
dispute.

OWID provides a simple and space efficient binary data structure for recording
the processors of transaction data in a form that is cryptographically
verifiable. OWIDs can optionally include additional data added by the processor
to the transaction.

A complex transaction involving multiple organizations is fully auditable when
all processors record their receipt or transmission of the unique identifier
associated with the transaction using OWIDs. OWIDs can be associated with one
another to form a tree that represents all the processors involved in a complex
transaction with the originating unique transaction identifier forming the root
of the tree.

Concrete implementations of OWID are available in the following repositories.

-   [.NET](https://github.com/SWAN-community/owid-dotnet) (version 5+)
-   [Go](https://github.com/SWAN-community/owid-go)
-   [JavaScript](https://github.com/SWAN-community/owid-js) (verify only)
-   [Rust](https://github.com/SWAN-community/owid-rust)
-   [Java](https://github.com/SWAN-community/owid-java)
-   [Python](https://github.com/SWAN-community/owid-python)
-   [PHP](https://github.com/SWAN-community/owid-php)

## Pre-requisites

This explainer assumes the reader is familiar with [public-key
cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) for the
purposes of signing data.

## Use Cases

The following is a non-exhaustive list of data that could be supported by OWID.

-   Pseudonymous identifiers used to relate multiple activities from the same
    application or device. For example, a common browser identifier, or mobile
    advertising identifier
    ([MAID](https://www.adsquare.com/mobile-advertising-ids-or-finding-the-right-mobile-users)).
-   Directly identifiable personal information. For example, name, email or
    telephone number.
-   Privacy or consent preferences.
-   Data processors signalling their compliance with laws such as GDPR or CCPA.

## Design Considerations

The following user cases and goals were considered in the design of OWIDs.

-   Scale
    -   Support for single transactions that involve thousands of unique
        processors.
    -   Use in environments where trillions of transactions will be generated
        daily.
-   Audit
    -   Random audit of entire transactions and all the associated processors.
        No processor should ever be aware of the transactions that will be
        audited by others.
    -   Partial audits of specific entities involved in a transaction.
-   Scope
    -   Offline processing to support situations where data exchange does not
        occur in real time. For example, where data is transmitted for machine
        learning purposes after the real time component of the transaction has
        been completed.
    -   Online processing where data is sent and received in real time.
-   Ease of adoption
    -   Simple deployed for implementors where this does not conflict with other
        design considerations.
-   Efficiency
    -   Minimize computing and data storage overhead.

## Implementation Considerations

The following considerations have been applied to the implementation.

-   Each processor must operate at least one domain to host OWID. For example,
    the processor ACME might use the domain acme.org to implement OWID.
-   Binary data structures are used in favor of more widely used human readable
    formats like JSON or XML to reduce data storage and transmission costs.
-   A single cryptographic signing algorithm. Implementors are not free to
    choose which algorithm to use. This reduces implementation complexity and
    data overheads.
-   A simple distribution of keys used to verify signatures for audit purposes
    via well-known end points associated with processors.
-   Provide a complete solution that incorporates the generation of the keys to
    avoid innocent mistakes that might otherwise lead to the transfer and
    exposure of keys.

### JSON Web Tokens

OWID was inspired by [JWT](https://jwt.io/) and adopted some similar conventions
and names such as payload and signature.

JWT were considered as a basis for OWIDs but do not meet all the design goals.
Specifically, JWT provides flexibility concerning a choice of algorithm and uses
JSON for data storage. JSON as a data structure is less efficient than binary
data structures. Using a single algorithm reduces implementation complexity.

## Standards

OWID is dependent on the following standards.

-   ECDSA NIST P-256 (FIPS 186-3, section D.2.3), also known as secp256r1 or
    prime256v1
-   Secure Hash Algorithm SHA-256 (FIPS 180-2, section 6.2)

SHA-256 is a robust hashing algorithm and widely supported.

P-256 has been chosen because of the relatively large number of programming
languages and libraries that offer support. Simple and consistent
implementations exist in JavaScript and Go. This algorithm is also comparatively
fast.

As the algorithm is only used to sign and verify data the encryption drawbacks
are mitigated. The computing costs associated with a bad actor faking signatures
at scale is likely to be unjustifiably expensive.

Future versions of OWID could use a different cryptographic algorithms or larger
sizes. If auditors require more modern algorithms this could be included in the
first deployed version.

## Out of Scope

### Compliance

Complying with GDPR or any other regional law is a matter for web authors, data
controllers or data processors and the advice of their privacy counsel.

### Encryption

The data held within the OWID payload does not need to be encrypted. It is left
to the creator of the OWID to determine the payload data, if any, and whether it
is encrypted.

### Key management

Implementors must ensure private keys are stored securely but are free to
determine how to achieve this.

Note: The implementation contains features to reduce the need for operational
personnel to be exposed to sensitive keys. Knowledge of Open SSL or command line
tools is not required.

### Transaction Audit

The process for assembling multiple OWIDs for the purposes of verifying a
transaction. For advertising supply chain audit this use case is covered in
[SWAN](https://github.com/SWAN-community/swan).

## Definitions

| Term        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Creator     | The processor that generated the OWID.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Date        | The date the OWID was created in UTC to the nearest minute.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Domain      | The domain that is associated with the Creator of the OWID. The registration information associated with the domain must provide contact details for other entities seeking to consume and verify the OWID. The domain can not be marked private or hidden behind a registrar. Well known end points must be exposed by the domain to support retrieval of the Processor’s common name, or public keys. Optional end points might be provided to support verification of OWIDs. See [End Points](\#\\End Points). A domain must not exceed 253 characters. RFC 1035 section 2.3.4 restricts a domain name to 255 octets in wire form, counting a length octet before every label and one octet for the root. In the presentation form used here a dot stands in for each of those length octets except the first, which has nothing before it, and the root has no character at all, so two of the 255 octets have no text and a domain holds at most 253 characters. This is a hard limit rather than a recommendation, so a creator must not use a longer domain and a consumer must reject an OWID whose domain field exceeds it. Within that limit the number of characters used for a domain should be as small as possible. As such it is expected OWID creators |
| Implementor | The entity that implements OWID. May be the same as the Creator or could be an agent operating on behalf of the Creator.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Parent      | OWID can be stand alone and not relate to other OWIDs. Where an OWID is dependent on another OWID then a parent child relationship is formed. The OWIDs could be considered to form a tree.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Payload     | The bytes that are optionally included in the OWID and which form part of the data used to sign the OWID. A payload does not need to be provided if the OWID is recording that some other data stored elsewhere has been processed by the creator.                                                                                                                                                                                                                                                                                                                                                                                |
| Processor   | The entity that is processing the data associated with the OWID. This explanation does not assign an identical meaning to “Processor” or “Data Processor” as GDPR although they do overlap.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Signature   | The byte array embedded into the OWID. The OWID cannot be changed after it has been signed. It becomes immutable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Version     | A single byte indicating the version of the OWID. Always the first byte of the array. 3 is the current version, and the version byte selects the end point path used to look the OWID up (see End Points).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## Data Structure

OWIDs are stored in the following data structure.

| Field          | Bytes                                            | Data Type               | Description                                                                                                                                                                                                                                                                                                        |
|----------------|--------------------------------------------------|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Version        | 1                                                | Byte                    | The byte version of the OWID. Note: Version 1and 2 have been deprecated during development as they used RSA which was considered insecure during initial review, or contained an insufficiently precise time indicator.                                                                                            |
| Domain         | Length of string plus 1 for null (0) terminator, and at most 254 bytes because the string must not exceed 253 characters (RFC 1035 section 2.3.4). | String                  | Domain associated with the creator.                                                                                                                                                                                                                                                                                |
| Date           | 4                                                | Unsigned 32 bit integer | The number of minutes that have elapsed since 01/01/2020 in UTC. Therefore 01/01/2021 would be 527,040, as 2020 has 366 days. The field is included for the purposes of changing signing keys, and resolve conflicts where multiple data items of the same type exist for a web browser and the most recent OWID needs to be determined. |
| Payload length | 4                                                | Unsigned 32 bit integer | Number of bytes that form the payload                                                                                                                                                                                                                                                                              |
| Payload        | See Payload length                               | Byte array              | Bytes that form the payload, if any.                                                                                                                                                                                                                                                                               |
| Signature      | 64                                               | Byte array              | A 64 byte array containing the ECDSA signature.                                                                                                                                                                                                                                                                    |

The minimum size of an OWID with a domain of six characters, for example 51d.io
or opx.io, and no payload is 77 bytes. If 1000 processors, all using six
characters domains, were involved in a single transaction the bytes consumed by
the associated OWID data that would result would be 77,000 bytes or 77kb.

## Signature

The signing algorithm first generates a SHA-256 digest of an OWID data structure without the signature field, and then signs it using the P-256 private key. This 64-byte signature is then appended to the original data to create the final OWID.

## Trees

OWIDs may exist in isolation or be related to one another to form a tree where
nodes of the tree are OWIDs.

OWIDs are likely to be part of a larger data structure containing additional
information that does not need to be verifiable via OWIDs. As such this
specification is not prescriptive concerning the implementation of the tree
structure.

If JSON or XML were to be used to store the larger data structure the binary
data of the OWID could be encoded using base 64 string encoding.

## End Points

Each OWID creator must host a domain with a number of well-known end points to
be used by other participants in an OWID compliant transaction. The following
table describes the end points.

The key words MUST, MUST NOT, SHOULD, and MAY in the end-point descriptions below are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

| End Point               | Requirement | Parameters                                                                                                                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------------|-------------|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /owid/api/v3/creator    | Mandatory   | date (optional): unsigned 32 bit integer, minutes since 01/01/2020 UTC, the OWID's own Date value (see Data Structure) | Provides a JSON response with the OWID creator information. The fields returned must include the domain, common name and public key that is also available via the public-key end point. The following is an example JSON response for the domain [new-pork-limes.uk](https://new-pork-limes.uk/owid/api/v3/creator). { "domain": "new-pork-limes.uk", "name": "New Pork Limes", "publicKeySPKI": "…" } Other fields could be included in time such as the [DUNS number](https://www.dnb.co.uk/duns-number/lookup.html), [registered company number](https://find-and-update.company-information.service.gov.uk/), or the URL of the applicable privacy policy. Creators that rotate signing keys accept the optional date parameter here as on the public-key end point, and the same rules apply (see that end point), with the selected key reported in the publicKeySPKI field. |
| /owid/api/v3/public-key | Mandatory   | format: valid values are “spki” or “pkcs”, and both return Subject Public Key Info PEM. date (optional): unsigned 32 bit integer, minutes since 01/01/2020 UTC, the OWID's own Date value (see Data Structure)                                                                                      | Returns the public key associated with the creator as Subject Public Key Info (SPKI) PEM, whichever format value is given. The pkcs value is accepted for compatibility with existing clients only, because no PKCS document defines an EC public key PEM and every creator in use returns SPKI for both values. The response is a text string in ECDSA Public Key form. For example: -----BEGIN PUBLIC KEY----- MFwwDQYJKo… /zrPytDbp\\naa == -----END PUBLIC KEY----- Each signing key has a start, being the moment from which the creator signs with it, and a key stays in force until the next key starts. A creator that has ever rotated its signing key MUST accept the optional date parameter on this end point and on the creator end point. A creator that has only ever signed with one key MAY omit it and MUST NOT be considered non-conformant for doing so. When date is supplied, the creator MUST return the key in force at date, being the key with the latest start on or before date, and MUST return a 404 response when date precedes the start of the oldest known key. The moment key material was generated is not its start and MUST NOT be used to select, because a creator may generate several periods of keys in one run and a key whose period has not started has signed nothing. When date is omitted, or is later than the moment of the request, the creator MUST return the key in force at the moment of the request, and MUST return 404 when no key is in force. Clients SHOULD send the OWID's own Date with every request. Clients SHOULD cache the keys returned, and SHOULD do so by creator end point rather than by request, where the end point is the scheme, the creator domain and the version, holding each key against the span of minutes the creator has confirmed it for. A key is in force from its start until the next key starts, so a key the creator returns for two minutes was in force at every minute between them, and an OWID whose Date falls inside a confirmed span MAY be verified with the held key without a request. A client MUST NOT extend a span past a minute the creator has confirmed, MUST NOT extend it across a minute the creator answered with a different key, and MUST NOT serve a held key for a minute outside its span. Clocks differ between creators and clients, and a creator reads a date later than its own now as now, so a client MUST NOT hold or serve a key for a minute within its clock drift allowance of its own now, or later, and SHOULD use an allowance of at least fifteen minutes. Such requests are put to the creator every time, which costs one request per minute per creator for OWIDs verified as they are created and none for older OWIDs. A client SHOULD bound the number of keys it holds and SHOULD offer a way to empty them, so that a key a creator withdraws after a compromise can be dropped. A client that holds only the current key can verify only the OWIDs signed since the most recent rotation, and MUST NOT report an older OWID as forged on the strength of the current key alone.                                                                                                                                                                                                       |
| /owid/register          | Optional    |                                                                                                                                | Used to register a domain as supporting OWID. Provides a simple user interface to capture the name of the organisation. This name is returned when the creator endpoint is called. Once the domain has been registered this end point should no longer respond. It is provided to avoid operations staff needing to be provided access to persistent storage or be involved in the creating of the keys.                                                                                                                                                                                                                                                        |
| /owid/api/v3/verify     | Optional    | OWID: the OWID as a base 64 URL encoded string. Parent (optional): another OWID that might have been used to create this OWID. | May be provided to support verification by entities that do not support public key verification. For reasons of latency and operational cost use of this end point in production is discouraged.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

The version segment of every end point path is the Version byte of the OWID
being looked up. A creator MUST serve the paths for every version it has ever
issued and MAY answer 404 for versions it has never issued.

If the mandatory end points are not provided the other participants in an OWID
transaction should ignore the OWID for the purposes of verification and inform
the registered domain owner that the OWID end points did not respond.

### Authentication

This specification does not require authentication on any end point, and a
creator MUST NOT be considered non-conformant either for requiring
authentication on the creator and public-key end points or for serving them
openly. A creator might require authentication where, for example, access to
the public key is part of a commercial subscription.

- An OWID creator MAY require authentication on the creator and public-key
  end points.
- A creator that requires authentication SHOULD accept the credential in each
  of these places: an HTTP request header, the query string, and the form
  body of a POST. The header and parameter names are chosen by the creator;
  for example an `X-Api-Key` header and a `key` parameter.
- When no credential is presented, a creator that requires authentication
  SHOULD respond with HTTP status code 401 and a body that describes every
  accepted way of supplying the credential.
- OWID clients MUST be prepared for both cases: a 200 response from a creator
  that serves these end points openly, and a 401 response with instructions
  from a creator that requires authentication.

### Verification

This specification does not require parties to verify OWIDs when they are
received or use. They have the option of doing so only if they wish. Therefore,
a policy could be supported where a fraction of OWIDs are verified at random.

Verification is recommended via the use of the published public key of the OWID
creator. The specification supports an optional end point operated by the OWID
creator at the domain used in the OWID to provide a verification service. This
could be used in web browsers that do not support ECDSA cryptography.

The number of processors in a OWID compliant transaction are likely to be
relatively small in computing terms. In a use case involving participants in an
auction transaction a reasonable limit might be 1000 participants excluding the
originating publisher. Assuming keys are changed daily, on any given day a total
of 1000 public keys will be needed to verify the non-publisher OWID
participants.

It is expected that verifiers of OWIDs will maintain a local copy of public keys
for the purposes of verification and avoid repeated requests to the public-key
endpoint of the other processors.

## Assumptions

OWID assumes the relationship between a domain and a processing entity can be
established via the existing infrastructure for DNS and SSL certificates. If
this proves untrue then it would be possible for a bad actor to temporarily
impersonate another entity via hijacking the domains of others. This is
considered unlikely and would indicate a wider failure of internet protocols and
practices.
