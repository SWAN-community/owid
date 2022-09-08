![Open Web Id](images/owl.128.pxls.100.dpi.png)

# Open Web Id (OWID)

## Abstract

Many data audit solutions require a cryptographically verifiable method of
validating the source of data, and the entities who processed that data. Records
of the data being sent or received, or just the meta-data of the parties, are
not cryptographically verifiable and can easily be faked. As such existing logs
are open to dispute.

OWID provides a simple and space efficient binary data structure to add the
details of the organisation that captured or generated the data (known as
processors of data) in a form that is cryptographically verifiable and exposes
the legal terms and conditions used. Processors become identifiable to all
recipients with the associated proofs.

A complex transaction involving multiple organizations becomes fully auditable
when all organisations add OWIDs to the data they generate.

OWIDs can be added to any data type including a single field or an entire set of
transactions identifying all the participating processors.

Concrete implementations of OWID are available in the following repositories.

-   [.NET](https://github.com/SWAN-community/owid-dotnet) (version 5+)
-   [Go](https://github.com/SWAN-community/owid-go)
-   [JavaScript](https://github.com/SWAN-community/owid-js) (verify only)

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
-   There is a one-to-one relationship between the domain used to host OWID and
    the terms and conditions associated with data captured. Therefore, the ACME
    processor might use sub domains a.acme.org and b.acme.org to represent two
    different terms and conditions.
-   Binary data structures are preferred in favour of more widely used human
    readable formats like JSON or XML to reduce data storage and transmission
    costs. JSON is also supported where human readability is essential.
-   A single cryptographic signing algorithm. Implementors are not free to
    choose which algorithm to use. This reduces implementation complexity and
    data overheads. A version field is used to supports changes to new
    cryptographic algorithms in the future.
-   Public keys used to verify OWID signatures for audit purposes are
    distributed by OWID hosts via well-known HTTP end points.

### JSON Web Tokens

OWID was inspired by [JWT](https://jwt.io/) and adopted some similar conventions
and names such as payload and signature.

JWT were considered as a basis for OWIDs but do not meet all the design goals.
Specifically, JWT provides flexibility concerning a choice of algorithm. Using a
single algorithm reduces implementation complexity.

Further, JWT embeds the data into the token. OWIDs are intended to be added to
existing data rather than be a container for that data. The approach makes
adding OWIDs to existing data simpler.

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
controllers or data processors and the advice of their privacy counsel. OWID
hosts must provide the terms and conditions used to capture or generate data at
a well-known end point of the domain included in the OWID. Users of their
services, and receivers of the data, will be able to inspect these documents to
determine compliance with laws.

### Encryption

The data associated with the OWID does not need to be encrypted. It is left to
the signer of the OWID to determine if the associated data should be encrypted.
Where encryption is used care should be taken to make clear to the receiver if
the OWID is associated with the encrypted or decrypted data.

### Key management

Implementors must ensure private keys are stored securely but are free to
determine how to achieve this.

Note: Some of the referenced implementations contain features to reduce the need
for operational personnel to be exposed to sensitive keys. Knowledge of Open SSL
or command line tools is not required when used with these implementations.

### Transaction Audit

The process for assembling multiple OWIDs for the purposes of verifying a
transaction. For advertising supply chain audit this use case is covered in
[SWAN](https://github.com/SWAN-community/swan).

## Definitions

| Term               | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Associated Data    | The data that is associated with the OWID and held in another data structure. Typically, the OWID will be included within the associated data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Date               | The date the OWID was created in UTC to the nearest minute.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Domain             | The domain that is associated with the Signer of the OWID. The registration information associated with the domain must provide contact details for other entities seeking to consume and verify the OWID. The domain cannot be marked private or hidden behind a registrar. Well known end points must be exposed by the domain to support retrieval of the Processor’s common name and public keys used by the Signer. Optional end points might be provided to support verification of OWIDs. See [End Points](\#\\End Points). The number of characters used for a domain should be as small as possible. As such it is expected OWID signers will operate short OWID specific domains rather than longer brand identifying domains. |
| Implementor / Host | The entity that implements OWID. May be the same as the Signer or could be an agent operating on behalf of the Signer.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Processor          | The entity that is processing the data associated with the OWID. This explanation does not assign an identical meaning to “Processor” or “Data Processor” as GDPR although they do overlap.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Receiver           | The entity that obtains data with an OWID who has the option of verifying the integrity of the OWID with the associated data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Signature          | The byte array embedded into the OWID. The associated data cannot be changed after it has been signed. It becomes immutable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Signer             | The processor that generated the OWID when capturing or generating the associated data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Version            | A single byte indicating the version of the OWID. Always the first byte of the array when stored in binary format. 1 is the current version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## Data Structure

OWIDs are stored in the following binary format data structure.

| Field     | Bytes                                            | Data Type               | Description                                                                                                                                                                                                                                                                                                        |
|-----------|--------------------------------------------------|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Version   | 1                                                | Byte                    | The byte version of the OWID. Note: Version 1and 2 have been deprecated during development as they used RSA which was considered insecure during initial review, or contained an insufficiently precise time indicator.                                                                                            |
| Domain    | Length of string plus 1 for null (0) terminator. | String                  | Domain associated with the signer.                                                                                                                                                                                                                                                                                 |
| Date      | 4                                                | Unsigned 32 bit integer | The number of minutes that have elapsed since 01/01/2020 in UTC. Therefore the 01/01/2021 would be 365. The field is included for the purposes of changing signing keys, and resolve conflicts where multiple data items of the same type exist for a web browser and the most recent OWID needs to be determined. |
| Signature | 64                                               | Byte array              | A 64 byte array containing the ECDSA signature.                                                                                                                                                                                                                                                                    |

The minimum size of an OWID with a domain of six characters, for example 51d.io
or opx.io, is 73 bytes. If 1000 processors, all using six characters domains,
were involved in a single transaction the bytes consumed by the associated OWID
data that would result would be 73,000 bytes or 73kb.

## Signature

The signing algorithm first generates a SHA-256 digest of an OWID data structure
without the signature field, and then signs it using the P-256 private key. This
64-byte signature is then appended to the original data to create the final
OWID.

## End Points

Each OWID signer must host a domain with a number of well-known end points to be
used by other participants in an OWID compliant transaction. The following table
describes the end points.

| End Point           | Requirement | Parameters                                                                                                                                                               | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /owid/api/v1/signer | Mandatory   |                                                                                                                                                                          | Provides a JSON response with the OWID signer information. The fields returned must include the domain, common name and public keys. The following is an example JSON response for the domain [new-pork-limes.uk](https://new-pork-limes.uk/owid/api/v1/creator). {"domain": "new-pork-limes.uk", "name": "New Pork Limes", “termsUrl”: “terms.com/conditions.html”, “keys”: { "publicKey": "…" }} Other fields could be included in time such as the [DUNS number](https://www.dnb.co.uk/duns-number/lookup.html), [registered company number](https://find-and-update.company-information.service.gov.uk/), or the URL of the applicable privacy policy. |
| /owid/register      | Optional    |                                                                                                                                                                          | Used to register a domain as supporting OWID. Provides a simple user interface to capture the name of the organisation and the terms and conditions URL. This name is returned when the signer endpoint is called. Once the domain has been registered this end point should no longer respond. It is provided to avoid operations staff needing to be provided access to persistent storage or be involved in the creating of the keys.                                                                                                                                                                                                                   |
| /owid/api/v1/verify | Optional    | OWID: the OWID as a base 64 URL encoded string. Data: the data as a byte array that relates to the OWD. The data must be the exact byte array used to generate the OWID. | May be provided to support verification by entities that do not support public key verification. For reasons of latency and operational cost use of this end point in production is discouraged.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

If the mandatory end points are not provided the other participants in an OWID
transaction should ignore the OWID for the purposes of verification and inform
the registered domain owner that the OWID end points did not respond.

### Verification

This explainer does not require parties to verify OWIDs when they are received
or used. Receivers have the option of doing so only if they wish. Therefore, a
policy could be supported where a fraction of OWIDs are verified at random.

Verification is recommended via the use of the published public key of the OWID
signer. The explainer supports an optional end point operated by the OWID signer
at the domain used in the OWID to provide a verification service. This could be
used in web browsers that do not support ECDSA cryptography.

The number of processors of OWIDs in even a complex OWID compliant transaction
are likely to be relatively small in computing terms. In a use case involving
participants in an advertising auction transaction a reasonable limit might be
1000 participants. Assuming keys are changed daily, on any given day a total of
1000 public keys will be needed to verify the participants.

It is expected that verifiers of OWIDs will maintain a local copy of public keys
for the purposes of verification and avoid repeated requests to the public-key
endpoint of the OWID processors.

## Assumptions

OWID assumes the relationship between a domain and a processing entity can be
established via the existing infrastructure for DNS and SSL certificates. If
this proves untrue, then it would be possible for a bad actor to temporarily
impersonate another entity via hijacking the domains of others. This is
considered unlikely and would indicate a wider failure of internet protocols and
practices.
