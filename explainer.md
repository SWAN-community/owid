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

A concrete implementation of OWID is available the Go programming language under
the [owid-go](https://github.com/51degrees/owid-go) repo.

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

-   Data processors signaling their compliance with laws such as GDPR or CCPA.

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

OWID is dependent on the following IETF standards.

-   PKCS \#1 Version 1.5 [[RFC 2313](https://tools.ietf.org/html/rfc2313)]

PKCS \#1 Version 1.5 has been chosen because of the relatively large number of
programming languages and libraries that offer support. Simple and consistent
implementations exist in JavaScript and Go. This algorithm is also comparatively
fast.

As the algorithm is only used to sign and verify data the encryption drawbacks
are mitigated. The computing costs associated with a bad actor faking signatures
at scale is likely to be unjustifiably expensive.

Future versions of OWID could use a different cryptographic algorithm. If
auditors require a more modern algorithm this could be included in the first
deployed version.

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
[SWAN](https://github.com/51Degrees/swan).

## Definitions

| Term        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Creator     | The processor that generated the OWID.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Date        | The date the OWID was created in UTC.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Domain      | The domain that is associated with the Creator of the OWID. The registration information associated with the domain must provide contact details for other entities seeking to consume and verify the OWID. The domain can not be marked private or hidden behind a registrar. Well known end points must be exposed by the domain to support retrieval of the Processor’s common name, or public keys. Optional end points might be provided to support verification of OWIDs. See [End Points](\#\\End Points). The number of characters used for a domain should be as small as possible. As such it is expected OWID creators |
| Implementor | The entity that implements OWID. May be the same as the Creator or could be an agent operating on behalf of the Creator.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Parent      | OWID can be stand alone and not relate to other OWIDs. Where an OWID is dependent on another OWID then a parent child relationship is formed. The OWIDs could be considered to form a tree.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Payload     | The bytes that are optionally included in the OWID and which form part of the data used to sign the OWID. A payload does not need to be provided if the OWID is recording that some other data stored elsewhere has been processed by the creator.                                                                                                                                                                                                                                                                                                                                                                                |
| Processor   | The entity that is processing the data associated with the OWID. This explanation does not assign an identical meaning to “Processor” or “Data Processor” as GDPR although they do overlap.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Signature   | The byte array embedded into the OWID. The OWID cannot be changed after it has been signed. It becomes immutable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Version     | A single byte indicating the version of the OWID. Always the first byte of the array. 1 is the current version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## Data Structure

OWIDs are stored in the following data structure.

| Field          | Bytes                                            | Data Type               | Description                                                                                                                                                                                                                                                                                                               |
|----------------|--------------------------------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Version        | 1                                                | Byte                    | The byte version of the OWID. Version 1 only.                                                                                                                                                                                                                                                                             |
| Domain         | Length of string plus 1 for null (0) terminator. | String                  | Domain associated with the creator.                                                                                                                                                                                                                                                                                       |
| Date           | 2                                                | Unsigned 16 bit integer | The number of days that have elapsed since 01/01/2020 in UTC. Therefore the 01/01/2021 would be 365. The field is included primarily for the purposes of changing signing keys on a daily basis. If a more precise timestamp is important to a particular purpose then the payload can be used to store this information. |
| Payload length | 4                                                | Unsigned 32 bit integer | Number of bytes that form the payload                                                                                                                                                                                                                                                                                     |
| Payload        | See Payload length                               | Byte array              | Bytes that form the payload, if any.                                                                                                                                                                                                                                                                                      |
| Signature      | 64                                               | Byte array              | A 64 byte array containing the signature.                                                                                                                                                                                                                                                                                 |

The minimum size of an OWID with a domain of six characters, for example 51d.io
or opx.io, and no payload is 77 bytes. If 1000 processors, all using six
characters domains, were involved in a single transaction the bytes consumed by
the associated OWID data that would result would be 77,000 bytes or 77kb.

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

| End Point               | Requirement | Parameters                                                                                                                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-------------------------|-------------|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /owid/api/v1/creator    | Mandatory   |                                                                                                                                | Provides a JSON response with the creator information associated with the OWID.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| /owid/api/v1/public-key | Mandatory   | format: valid values are “spki” or “pkcs”                                                                                      | Returns the public key associated with the creator in Subject Public Key Info (SPKI) or Public Key Cryptography Standards (PKCS) form. The response is a text string in RSA Public Key form. For example: -----BEGIN RSA PUBLIC KEY----- MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJ8pVQDNLQ3xaBJc6ey6/rA/zrPytDbp\\naaIyBC99DECMzbTffXG7DHsuqKmQjbPUAUYa+66dW7UIzlU6avt1/XsCAwEAAQ== -----END RSA PUBLIC KEY----- Issue: It is likely that the date will also form part of the request to enable OWID creators to change keys on a daily basis. This is a future enhancement. |
| /owid/register          | Mandatory   |                                                                                                                                | Used to register a domain as supporting OWID. Provides a simple user interface to capture the name of the organisation. This name is returned when the creator endpoint is called. Once the domain has been registered this end point should no longer respond. It is provided to avoid operations staff needing to be provided access to persistent storage or be involved in the creating of the keys.                                                                                                                                                              |
| /owid/api/v1/verify     | Optional    | OWID: the OWID as a base 64 URL encoded string. Parent (optional): another OWID that might have been used to create this OWID. | May be provided to support verification by entities that do not support public key verification. For reasons of latency and operational cost use of this end point in production is discouraged.                                                                                                                                                                                                                                                                                                                                                                      |

If the mandatory end points are not provided then other participants in an OWID
transaction should ignore the OWID for the purposes of verification and inform
the registered domain owner that the OWID end points did not respond.

### Verification

The number of processors in a OWID compliant transaction are likely to be
relatively small in computing terms. In a use case involving participants in an
auction transaction a reasonable limit might be 1000 participants. On any given
day a total of 1000 public keys will be needed to verify the other OWID
participants.

It is expected that verifiers of OWIDs will maintain a local copy of public keys
for the purposes of verification and avoid repeated requests to the public-key
endpoint of the other processors.

## Assumptions

OWID assumes the relationship between a domain and an processing entity can be
established via the existing infrastructure for DNS and SSL certificates. If
this proves un true then it would be possible for a bad actor to temporarily
impersonate another entity via hijacking the domains of others.
