# PDTF Interoperability Profile

The Property Data Trust Framework consists of 4 main components

- **Credential Exchange**
- **Schemas**
- **Roles & Permissions**
- **Discovery**

## Credential Exchange
The PDTF enables the exchange of property data expressed as PDTF Credentials. PDTF credentials are expressed and exchanged according to these 
two interoperability profiles based around OpenID 4 VCs, that enable the issuance and presentation of w3c VCs in VC-JWT format:

- [https://identity.foundation/jwt-vc-presentation-profile](https://identity.foundation/jwt-vc-presentation-profile/)
- [https://identity.foundation/jwt-vc-issuance-profile](https://identity.foundation/jwt-vc-issuance-profile/)

Additionally, a PDTF credential must include a single valid PDTF credential `type` and reference a PDTF Credential Schema, see [schemas](#schemas).

Each participant of the PDTF must control at least one DID of a supported method defined in these profiles, and must support the full presentation or issuance profile.

## Schemas

The PDTF will publish PDTF Credential Schemas, where a PDTF Credential Schema is a JsonSchema associated with a single canonical `type`.
Schemas are organised into the following categories, where each category represents a different type of subject.


**Property Credentials**

Credentials where the subject is a property. A property can be identified with a UPRN or an address.

**Person Credentials**

Credentials where the subject is a person. A person can be identified using a DID of a supported type defined in the Credential Exchange profiles.

**Title Extract Credentials**

Credentials where the subject is a title (extracted from the land registry). A title identified can be identified using ??? .

PDTF credentials must reference a schema using the mechanism descried in [vc-json-schema](https://www.w3.org/TR/vc-json-schema/#jsonschema), 
except applied to the vc data model 1.1. For now we use JsonSchema, but will explore JsonSchemaCredential. 
JsonSchemas are kept [here](../model/pdtf/schemas), and published as described in [Discovery](#discovery).

In practice this means a PDTF credential must include a single valid PDTF credential type in the credential `type`, and reference the corresponding PDTF schema in the `credentialSchema` field, with type `JsonSchema` .
i.e. a PDTF credential would look like:

```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "type": [
    "VerifiableCredential",
    "ExampleCredential"
  ],
  "issuanceDate": "2024-01-29T15:00:45Z",
  "issuer": "did:example",
  "credentialSubject": {
    "id": "did:alice",
    "name": "alice"
  },
"credentialSchema": {
    "id": "https://https://propdata.org.uk/schemas/example.json",
    "type": "JsonSchema"
 }
}

```

Where the `credentialSchema.id` resolves to:

```json
{
  "$id": "http://propdata.org.uk/schemas/example.json",
  "$schema": "https://json-schema.org/draft/2019-09/schema",
  "title": "ExampleCredential",
  "description": "Example person credential",
  "type": "object",
  "properties": {
    "credentialSubject": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "name": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "name"
      ],
      "additionalProperties": false
    }
  }
}
```

The recognised PDTF Credential Schemas are published as a single trust assertion in a Trust Establishment Document. The assertion follows this
[topic](../model/pdtf/pdtf-credentials.topic.json), and can be found [here](../model/pdtf/pdtf-credentials.topic.json)

## Roles and Permissions

The PDTF will assign participants roles which are associated with certain permissions related to the exchange of PDTF credentials. 
Participants of the PDTF are expected to adhere to these permissions.


Role are assigned using a [Trust Establishment Document](https://identity.foundation/trust-establishment/) maintained by the PDTF. 
The PDTF will control and publicize a DID that it will use to make role assertions about participants. 
Each participant must declare a public did to the PDTF, that will be used as the subject of role assertions. 
Additionally a participant must declare an HTTPS domain (ideally this can be discovered using `serviceEndpoints` in the DID doc 
however not all did methods will be suitable for this) and a [Wellknown DID configuration](https://identity.foundation/.well-known/resources/did-configuration/) to link this domain with their declared DID.

Each role has an associated trust establishment topic. The current roles are:

* [Participant](../model/pdtf/roles/pdtf-participant.topic.json) - This entity is a particpant of the PDTF.
* Issuer - This entity can issue any of the PDTF credential defined in their role entry.
* Verifier - This entity can request any of the PDTF credential defined in their role entry.

## Discovery

The PDTF maintains schemas and roles in this public github repository. 
Collectively they constitute a "trust model" that is built into a single Trust Establishment Document that describes the trust framework as a whole. 
The Trust Establishment Document is published along-side any referenced resources such as json schemas, 
to an official PDTF web server and to an official PDTF artifact registry.

Additionally, the PDTF will run a discovery service that offers an API and web interface capable of

- resolving a participant did to its associated permissions
- resolving a credential type to its associated schema and to participants that have permission to issue or verify credentials of that type.

The code for this service will be open source, and anyone can host it. Participants do not have to use it, they just have to be confident they adhere to the roles.

Generally, participants are recommended to take an offline approach to discovery. That is, roles and schemas should be treated like a software package / dependency of their larger application. It should be pulled and embedded into your application, and kept up to date using regular re-deployments.