# SEDI Implementation Guide

The term ACDC stands for Authentic Chained Data Container. It is the supporting format and protocol used for SEDI. The ACDC protocol is an open standard with open-source tooling.

In this guide, the term *citizen* refers to persons issued State Endorsed Digital Identity (SEDI) entitlements. In some cases, a *citizen* may not technically be a citizen but may be a visitor or have temporary residence status.  The term *citizen* is used in a generic sense to refer to anyone who receives a SEDI, i.e., a citizen-sourced, state-endorsed digital identity. When more specificity is required, the guild will further qualify the citizen status. Fields in a citizen's core identity ACDC provide their citizenship status. The term *State* (capitalized) refers to the *State of Utah* or another state running a SEDI program using the ACDCs outlined in this guide. An *entitlement* generally refers to an ACDC issued by the State or one of its delegated agents. An entitlement may be a credential or license or some other instrument. 

The important detail in SEDI is that the citizen's identifier is an AID (autonomic IDentifier) totally under the citizen's control. An AID is a cryptographically derived pseudonym or cryptonym for short. Control over the AID cryptonym is via private cryptographic keys. Whoever controls the keys controls the identifier. Euphemistically, AIDs support the concept of "my keys, my identity or not my keys, not my identity". This requires citizens to manage their keys. The KERI (Key Event Receipt Infrastructure) is an open protocol that enables any entity to have total control over their identity (identifiers) via a fault-tolerant, decentralized key management infrastructure. ACDCs are build on top of KERI. 

The State and its agents all have their own AIDs. They use these to issue entitlements to citizens.
A citizen can then present a State-issued entitlement to a 2nd party, which can cryptographically verify that the State authentically issued it and that the citizen authentically presented it.

KERI and ACDCs support two unique, vitally important properties for SEDI. These are perpetual identity and perpetually verifiable issuances.  

Perpetual identity means that the controller of the identifier(s) (AIDs) associated with that identity can maintain control in perpetuity despite continuing rotations of the keys that control those identifier(s) (AIDs). This property enables true decentralization of digital identity. True perpetual identity means a citizen is not reliant on or beholden to trusted third parties (such as the State) to create and maintain control over that citizen's own digital identity.

Perpetually verifiable issuances mean that an issuing entity such as the State can issue endorsements from their identifier(s) (AIDs) that refer to a citizen's identifier(s) (AIDs) where such issuances may be verified in perpetuity despite continuing rotations of the issuer's keys that control their own identifiers (AIDs). This property enables persistent entitlements whose lifespan is not limited by the cryptoperiod (lifespan) of their cryptographic keys. Once issued, a citizen can use the entitlement without continuing reliance on the State. This removes the centralizing force of entitlements that must be reissued on short time scales determined by the cryptoperiod of the issuer keys. Such reissuance essentially forces continued interaction with the State in order to use the entitlements. Such reliance effectively defeats citizen control over their digital identity.

## SEDI ACDC Schemas

All schemas in this part of the guide are provided as Python dicts suitable for use with the keripy library. Pure JSON versions are provided in an appendix.

### Schema Table
The following table provides the schema ID  `$id` field and a description. ACDCs use immutable json schema identifiers computed as the SAID of the schema. This immutability means that an attacker can't tamper with or substitute a schema with different semantics.  This lets the schema ID (schema SAID) serve as the ACDC type. ACDCs with the same schema are of the same type. An ecosystem governance framework can define use cases tied to the Schema ID (schema SAID) as type.  ACDC JSON Schema has a version field, so the version is protected by the SAID. This lets tooling correctly identify which code path to use for each ACDC type and version. Discovering an ACDC type only requires published directory of schema IDs. Services that recognize those types often provide this via a .well-known resource on a website.


| Schema SAID $id | ACDc Description |
|:--|---|
| ELjJlSaExu9ss766dDpQoLE5aT6-wIRyR72X5YLC3ILc | SEDI Organizational Unit Authority Delegation |
| EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v | SEDI Issuing Agent Authority Delegation |
| N0JtdzBmFuTUzyJG9CXZhtdu-bW0V6L95bsVBzi-cY- | SEDI Citizen Core Identity Attributes|
| EH5hn7CbtDFxhS3IKioC8P-e4YyZoxqzhRpr4X_e32BD | SEDI Citizen Residence Address Attributes|
|  | SEDI Citizen Age Threshold Attributes|




###  Organizational Unit Schema
This defines an ACDC that the State's root-of-trust AID or root AID issues to delegate an Organizational Unit AID within the State hierarchy. Typically, an organizational unit is at a department, division, or program level within the State.

```python
UnitSchemaSaid = 'ELjJlSaExu9ss766dDpQoLE5aT6-wIRyR72X5YLC3ILc'
UnitSchema = \
{
  '$id': 'ELjJlSaExu9ss766dDpQoLE5aT6-wIRyR72X5YLC3ILc',
  '$schema': 'https://json-schema.org/draft/2020-12/schema',
  'title': 'SEDI Organizational Unit Schema',
  'description': 'SEDI Oganizational Unit JSON Schema for acm ACDC.',
  'credentialType': 'SEDI_Org_ACDC_acm_message',
  'version': '0.1.0',
  'type': 'object',
  'required': ['v', 'd', 'i', 'rd', 's', 'a', 'r'],
  'properties':
  {
    'v': {'description': 'ACDC version string', 'type': 'string'},
    't': {'description': 'Message type', 'type': 'string'},
    'd': {'description': 'Message SAID', 'type': 'string'},
    'u': {'description': 'Message UE', 'type': 'string'},
    'i': {'description': 'Issuer AID', 'type': 'string'},
    'rd': {'description': 'Registry SAID', 'type': 'string'},
    's':
    {
      'description': 'Schema Section',
      'oneOf':
      [
        {'description': 'Schema Section SAID', 'type': 'string'},
        {'description': 'Schema Section Detail','type': 'object'}
      ]
    },
    'a':
    {
      'description': 'Attribute Section',
      'oneOf':
      [
        {'description': 'Attribute Section SAID','type': 'string'},
        {
          'description': 'Attribute Section Detail',
          'type': 'object',
          'required':
          [
            'd',
            'u',
            'i',
            'issuedDate',
            'unit',
          ],
          'properties':
          {
            'd': {'description': 'Attribute Section SAID', 'type': 'string'},
            'u': {'description': 'Attribute Section UE', 'type': 'string'},
            'i': {'description': 'Issuee AID', 'type': 'string'},
            'rd': {'description': 'Issuee Presentation Registry SAID', 'type': 'string'},
            'issuedDate': {'description': 'Issued Date as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
            'unit': {'description': 'Organizational Unit', 'type': 'string'},
          },
          'additionalProperties': False
        }
      ]
    },
    'r':
    {
      'description': 'Rule Section',
      'oneOf':
      [
        {'description': 'Rule Section SAID', 'type': 'string'},
        {
          'description': 'Rule Section Detail',
          'type': 'object',
          'required': ['d', 'l'],
          'properties':
          {
            'd': {'description': 'Rule Section SAID', 'type': 'string'},
            'l': {'description': 'Legal Language', 'type': 'string'}
          },
        'additionalProperties': False
        }
      ]
    }
  },
  'additionalProperties': False
}
```

#### Organizational Unit ACDC Example

```python
{
  'v': 'ACDCCAACAAJSONAAIn.',
  't': 'acm',
  'd': 'EIojbncOpuCoMFWZSutjfkLf5KNu_nbCOsqgFClk3SMK',
  'u': '0AAxOP-l4O8GZPbv7dQBeNfM',
  'i': 'EAN4gwVu_a9EDxFn-camF6OoQvZAvi-_FyluyyZFKzs5',
  'rd': 'EA3Y6LeoyNFjnLS1xZoRQnzX0fWgUw0XjD4IgIf8ZHxD',
  's': 'EBbOpBe0lP_epHhakeiXdOmeal3leDfLRf0PAItsanoQ',
  'a': 
  {
    'd': 'EHydQsy-4di85pJaiPqVN9_APpLtU15QXcmVFEN2FQE-',
    'u': '0AAk4UxXJc8mbqWSdtAPkK6w',
    'i': 'EMrUfTuv8j3vBVAgPSlDi1D_o35F5uwsIAWjBatLhK9E',
    'issuedDate': '2020-08-01T00:00:00.000000+00:00',
    'unit': 'SediProgramOffice'
  },
  'r': 
  {
    'd': 'EFPxq4WPl29szUqbrQIviOh_Ls_RlrYbp4L-fdQH0XrX', 
    'l': ''
  }
}

```

###  Issuing Agent Schema
This defines an ACDC that a State Organizational Unit AID issues to delegate an Issuing Agent AID within the State hierarchy. An Issuing Agent operates within the aegis of a State Organizational Unit. Issuing Agents issue entitlements to citizens.

```python
AgentSchemaSaid = 'EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v'
AgentSchema = \
{
  '$id': 'EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v',
  '$schema': 'https://json-schema.org/draft/2020-12/schema',
  'title': 'SEDI Issuing Agent Schema',
  'description': 'SEDI Issuing Agent JSON Schema for acm ACDC.',
  'credentialType': 'SEDI_Agent_ACDC_acm_message',
  'version': '0.1.0',
  'type': 'object',
  'required': ['v', 'd', 'i', 'rd', 's', 'a', 'e', 'r'],
  'properties':
  {
    'v': {'description': 'ACDC version string', 'type': 'string'},
    't': {'description': 'Message type', 'type': 'string'},
    'd': {'description': 'Message SAID', 'type': 'string'},
    'u': {'description': 'Message UE', 'type': 'string'},
    'i': {'description': 'Issuer AID', 'type': 'string'},
    'rd': {'description': 'Registry SAID', 'type': 'string'},
    's':
    {
      'description': 'Schema Section',
      'oneOf':
      [
        {'description': 'Schema Section SAID', 'type': 'string'},
        {'description': 'Schema Section Detail','type': 'object'}
      ]
    },
    'a':
    {
      'description': 'Attribute Section',
      'oneOf':
      [
        {'description': 'Attribute Section SAID','type': 'string'},
        {
          'description': 'Attribute Section Detail',
          'type': 'object',
          'required':
          [
            'd',
            'u',
            'i',
            'issuedDate',
            'role',
            'name',
          ],
          'properties':
          {
            'd': {'description': 'Attribute Section SAID', 'type': 'string'},
            'u': {'description': 'Attribute Section UE', 'type': 'string'},
            'i': {'description': 'Issuee AID', 'type': 'string'},
            'rd': {'description': 'Issuee Presentation Registry SAID', 'type': 'string'},
            'issuedDate': {'description': 'Issued Date as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
            'role': {'description': 'Issuing Agent Role', 'type': 'string'},
            'name':
            {
              'description': 'Name Block',
              'oneOf':
              [
                {'description': 'Name SAID', 'type': 'string'},
                {
                  'description': 'Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Issuing Agent Name', 'type': 'string'},
                  },
                  'additionalProperties': False
                }
              ]
            },
          },
          'additionalProperties': False
        }
      ]
    },
    'e':
    {
      'description': 'Edge Section',
      'oneOf':
      [
        {'description': 'Edge Section SAID', 'type': 'string'},
        {
          'description': 'Edge Section Detail',
          'type': 'object',
          'required': ['d', 'u', 'orgUnit'],
          'properties':
          {
            'd': {'description': 'Edge Section SAID', 'type': 'string'},
            'u': {'description': 'Edge Section UE', 'type': 'string'},
            'orgUnit':
            {
              'description': 'Utah Organizational Unit Edge Block',
              'type': 'object',
              'required': ['n', 's', 'o'],
              'properties':
              {
                'd': {'description': 'Edge SAID', 'type': 'string'},
                'u': {'description': 'Edge UE', 'type': 'string'},
                'n': {'description': 'Far Node SAID', 'type': 'string'},
                's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                'o': {'description': 'Edge Unary Operator', 'type': 'string'}
              },
              'additionalProperties': False
            }
          },
          'additionalProperties': False
        }
      ]
    },
    'r':
    {
      'description': 'Rule Section',
      'oneOf':
      [
        {'description': 'Rule Section SAID', 'type': 'string'},
        {
          'description': 'Rule Section Detail',
          'type': 'object',
          'required': ['d', 'l'],
          'properties':
          {
            'd': {'description': 'Rule Section SAID', 'type': 'string'},
            'l': {'description': 'Legal Language', 'type': 'string'}
          },
        'additionalProperties': False
        }
      ]
    }
  },
  'additionalProperties': False
}

```

#### Issuing Agent ACDC Example

```python
{
  'v': 'ACDCCAACAAJSONAAO9.',
  't': 'acm',
  'd': 'EOD4QiWo1oa6UtiEr456Lu0ba3v1Cmd6qOPoRbAdx2hD',
  'u': '0ACpkR3a0WoJk8BHz4M6aZVo',
  'i': 'EMrUfTuv8j3vBVAgPSlDi1D_o35F5uwsIAWjBatLhK9E',
  'rd': 'EEy3daQxc9NrgzA1V7KjgOqLF_te2gs2-sYElTHsPzYE',
  's': 'EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v',
  'a': 
  {
    'd': 'EFPQAD6XLlU9bnNPMWfjl1Q-Rf20Jwoip_1lkp8XGKKt',
    'u': '0ADtdKN8DlnUKUFDN47QKfNj',
    'i': 'EKBCU6u_xObNhFc9uuz1VdntNt99xmB2fA5qz7Li-Sl-',
    'issuedDate': '2020-08-01T00:00:00.000000+00:00',
    'role': 'SediIssuingAgent',
    'name': 
    {
      'd': 'EJj1BPhB8uGFLA8QtNjNJ9MXJ8fvOliIWj5w7YLzReWI',
      'u': '0AC0eUUAKHVXdJMDVXtx1yS3',
      'value': 'Susan Park'
    }
  },
  'e': 
  {
    'd': 'EIZC3vk0dgDvZNV3Vf1z3mav0HkMbGHq0vxm7NcbC6WP',
    'u': '0AB4w_WXy8JmoUebF1aEnOD_',
    'orgUnit': 
    {
      'd': 'ENZvB2t1Tit1Z2PSSebS70IKEq33JynJrWK-rjR_cSMA',
      'u': '0ABl8aVsQRgBVSWnMQcwgDQ7',
      'n': 'EIojbncOpuCoMFWZSutjfkLf5KNu_nbCOsqgFClk3SMK',
      's': 'EBbOpBe0lP_epHhakeiXdOmeal3leDfLRf0PAItsanoQ',
      'o': 'DI2I'
     }
  },
  'r': 
  {
     'd': 'EFPxq4WPl29szUqbrQIviOh_Ls_RlrYbp4L-fdQH0XrX', 
     'l': ''
  }
}
```

### Core Identity Schema
This defines an ACDC that a State Issuing Agent AID issues to a citizen AID to endorse that citizen's core identity attributes. 

```python

CoreSchemaSaid = 'EN0JtdzBmFuTUzyJG9CXZhtdu-bW0V6L95bsVBzi-cY-'
CoreSchema = \
{
  '$id': 'EN0JtdzBmFuTUzyJG9CXZhtdu-bW0V6L95bsVBzi-cY-',
  '$schema': 'https://json-schema.org/draft/2020-12/schema',
  'title': 'SEDI Core Schema',
  'description': 'SEDI Core Identity JSON Schema for acm ACDC.',
  'credentialType': 'SEDI_Core_ACDC_acm_message',
  'version': '0.1.0',
  'type': 'object',
  'required': ['v', 'd', 'i', 'rd', 's', 'a', 'e', 'r'],
  'properties':
  {
    'v': {'description': 'ACDC version string', 'type': 'string'},
    't': {'description': 'Message type', 'type': 'string'},
    'd': {'description': 'Message SAID', 'type': 'string'},
    'u': {'description': 'Message UE', 'type': 'string'},
    'i': {'description': 'Issuer AID', 'type': 'string'},
    'rd': {'description': 'Registry SAID', 'type': 'string'},
    's':
    {
      'description': 'Schema Section',
      'oneOf':
      [
        {'description': 'Schema Section SAID', 'type': 'string'},
        {'description': 'Schema Section Detail','type': 'object'}
      ]
    },
    'a':
    {
      'description': 'Attribute Section',
      'oneOf':
      [
        {'description': 'Attribute Section SAID','type': 'string'},
        {
          'description': 'Attribute Section Detail',
          'type': 'object',
          'required':
          [
            'd',
            'u',
            'i',
            'primary',
            'givenName',
            'middleName',
            'familyName',
            'nameSuffix',
            'birthDate',
            'facialImageProof',
            'legalPresenceStatus',
            'issuedDate',
            'expirationDate',
          ],
          'properties':
          {
            'd': {'description': 'Attribute Section SAID', 'type': 'string'},
            'u': {'description': 'Attribute Section UE', 'type': 'string'},
            'i': {'description': 'Issuee AID', 'type': 'string'},
            'rd': {'description': 'Issuee Presentation Registry SAID', 'type': 'string'},
            "primary": { "description": "Primary True if not bulk issued else False", "type": "boolean"},
            'givenName':
            {
              'description': 'Given Name Block',
              'oneOf':
              [
                {'description': 'Given Name SAID', 'type': 'string'},
                {
                  'description': 'Given Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Given Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                }
              ]
            },
            'middleName':
            {
              'description': 'Middle Name(s) Block',
              'oneOf':
              [
                {'description': 'Middle Name SAID','type': 'string'},
                {
                  'description': 'Middle Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Middle Name(s) Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'familyName':
            {
              'description': 'Family Name Block',
              'oneOf':
              [
                {'description': 'Family Name SAID', 'type': 'string'},
                {
                  'description': 'Family Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Family Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'nameSuffix':
            {
              'description': 'Name Suffix Block',
              'oneOf':
              [
                {'description': 'Name Suffix SAID', 'type': 'string'},
                {
                  'description': 'Name Suffix Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Name Suffix Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'birthDate':
            {
              'description': 'Birth Date Block',
              'oneOf':
              [
                {'description': 'Birth Date SAID','type': 'string'},
                {
                  'description': 'Birth Date Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Birth Date Value as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
                  },
                'additionalProperties': False
                },
              ]
            },
            'facialImageProof':
            {
              'description': 'Facial Image Proof Block',
              'oneOf':
              [
                {'description': 'Facial Image Proof SAID', 'type': 'string'},
                {
                  'description': 'Facial Image Proof Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Facial Image Proof Value as SAID of typed media block', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'legalPresenceStatus':
            {
              'description': 'Legal Presence Status Block',
              'oneOf':
              [
                {'description': 'Legal Presence Status SAID', 'type': 'string'},
                {
                  'description': 'Legal Presence Status Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Legal Presence Status Value i.e. citizen', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'issuedDate':
            {
              'description': 'Issued Date Block',
              'oneOf':
              [
                {'description': 'Issued Date SAID', 'type': 'string'},
                {
                  'description': 'Issued Date Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                   'value': {'description': 'Issued Date Value as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'expirationDate':
            {
              'description': 'Expiration Date Block',
              'oneOf':
              [
                {'description': 'Expiration Date SAID', 'type': 'string'},
                {
                  'description': 'Expiration Date Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Expiration Date Value as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
                  },
                  'additionalProperties': False
                }
              ]
            },
          },
          'additionalProperties': False
        }
      ]
    },
    'e':
    {
      'description': 'Edge Section',
      'oneOf':
      [
        {'description': 'Edge Section SAID', 'type': 'string'},
        {
          'description': 'Edge Section Detail',
          'type': 'object',
          'required': ['d', 'u', 'utahAgent'],
          'properties':
          {
            'd': {'description': 'Edge Section SAID', 'type': 'string'},
            'u': {'description': 'Edge Section UE', 'type': 'string'},
            'utahAgent':
            {
              'description': 'Utah Agent Edge Block',
              'type': 'object',
              'required': ['n', 's', 'o'],
              'properties':
              {
                'd': {'description': 'Edge SAID', 'type': 'string'},
                'u': {'description': 'Edge UE', 'type': 'string'},
                'n': {'description': 'Far Node SAID', 'type': 'string'},
                's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                'o': {'description': 'Edge Unary Operator', 'type': 'string'}
              },
              'additionalProperties': False
            },
            'guardians':
            {
              'description': 'Guardian Edge Group Block',
              'type': 'object',
              'required': ['d', 'u', 'o', 'first'],
              'properties':
              {
                'd': {'description': 'Edge Group SAID', 'type': 'string'},
                'u': {'description': 'Edge Group UE', 'type': 'string'},
                'o': {'description': 'Edge Group M-ary Operator', 'type': 'string'},
                'first':
                {
                  'description': 'First Guardian Edge Block',
                  'type': 'object',
                  'required': ['d', 'u', 'n', 's', 'o'],
                  'properties':
                  {
                    'd': {'description': 'Edge SAID', 'type': 'string'},
                    'u': {'description': 'Edge UE', 'type': 'string'},
                    'n': {'description': 'Far Node SAID', 'type': 'string'},
                    's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                    'o': {'description': 'Edge Unary Operator', 'type': 'string'}
                  },
                  'additionalProperties': False
                },
                'second':
                {
                  'description': 'Second Guardian Edge Block',
                  'type': 'object',
                  'required': ['d', 'u', 'n', 's', 'o'],
                  'properties':
                  {
                    'd': {'description': 'Edge SAID', 'type': 'string'},
                    'u': {'description': 'Edge UE', 'type': 'string'},
                    'n': {'description': 'Far Node SAID', 'type': 'string'},
                    's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                    'o': {'description': 'Edge Unary Operator', 'type': 'string'}
                  },
                  'additionalProperties': False
                },
                'third':
                {
                  'description': 'Third Guardian Edge Block',
                  'type': 'object',
                  'required': ['d', 'u', 'n', 's', 'o'],
                  'properties':
                  {
                    'd': {'description': 'Edge SAID', 'type': 'string'},
                    'u': {'description': 'Edge UE', 'type': 'string'},
                    'n': {'description': 'Far Node SAID', 'type': 'string'},
                    's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                    'o': {'description': 'Edge Unary Operator', 'type': 'string'}
                  },
                  'additionalProperties': False
                },
                'fourth':
                {
                  'description': 'Fourth Guardian Edge Block',
                  'type': 'object',
                  'required': ['d', 'u', 'n', 's', 'o'],
                  'properties':
                  {
                    'd': {'description': 'Edge SAID', 'type': 'string'},
                    'u': {'description': 'Edge UE', 'type': 'string'},
                    'n': {'description': 'Far Node SAID', 'type': 'string'},
                    's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                    'o': {'description': 'Edge Unary Operator', 'type': 'string'}
                  },
                  'additionalProperties': False
                },
              },
              'additionalProperties': False
            },
          },
          'additionalProperties': False
        }
      ]
    },
    'r':
    {
      'description': 'Rule Section',
      'oneOf':
      [
        {'description': 'Rule Section SAID', 'type': 'string'},
        {
          'description': 'Rule Section Detail',
          'type': 'object',
          'required': ['d', 'l'],
          'properties':
          {
            'd': {'description': 'Rule Section SAID', 'type': 'string'},
            'l': {'description': 'Legal Language', 'type': 'string'}
          },
        'additionalProperties': False
        }
      ]
    }
  },
  'additionalProperties': False
}

```

#### Core Identity ACDC Example

```python
{
  'v': 'ACDCCAACAAJSONAAfN.',
  't': 'acm',
  'd': 'EGsPyGCyHtDSWP61rVsRh3F8qooP_jrX6qPHqHn7-FOg',
  'u': '0ABQNZNkD1y0W4mglDB4ei5Y',
  'i': 'EKBCU6u_xObNhFc9uuz1VdntNt99xmB2fA5qz7Li-Sl-',
  'rd': 'EDOfxmEeOsWdi5ZQyuy98W4s15vmV1RWVFuqm4GZflTn',
  's': 'EN0JtdzBmFuTUzyJG9CXZhtdu-bW0V6L95bsVBzi-cY-',
  'a': 
  {
    'd': 'EPIuMf4tkzdqVMSuq59rBD14LZpmFtFjhnh8Io1BWdbt',
    'u': '0ACd8yXBMGBLNDwr-MMgtris',
    'i': 'EDB8gKNwzurf33pV2hsyGR9XFOmitDhc0LUzDamcU2JR',
    'rd': 'ELUW5D0X0pMFM30ZHZKB997lad86PISushrKkKzlhQrl',
    'primary': True,
    'givenName': 
    {
      'd': 'EBu_EdUcstZq6woZ7NMe2pyU7jjcQOdC9w1ryHa1P_Sq',
      'u': '0ADGYtYEzEdGpaq_sDXwamDm',
      'value': 'Guy'
    },
    'middleName': 
    {
      'd': 'EOhalIHhb5ZbrJZ5SMY_vWBj2ds_z9W8mJ3j-FTjgSUz',
      'u': '0ACEZIR6pk97xr2cy-gBdod4',
      'value': 'Marty McFly'
    },
    'familyName': 
    {
      'd': 'EDBg78wYuNEQkcv-XciKFkxDQmRvIeTsXfjVMJLtUjYW',
      'u': '0ACUdqI4OVtXDL5BBO13QdrJ',
      'value': 'Brown'
    },
    'nameSuffix': 
    {
      'd': 'EFDPwXKE-3wg-WTUVu4GWfMeu4bj8rGNkvdFrMo4Ja4N',
      'u': '0ACXabyEAzJ1U-4qOek3adv5',
      'value': 'Jr'
    },
    'birthDate': 
    {
      'd': 'EMGKn6dwPJMd79vWaGEv7OlCQQ0oGd0nt0fRqBfz1ZyA',
      'u': '0ABR4UtpgSSmCSVF1eNJ6joz',
      'value': '2002-08-22T00:00:00.000000+00:00'
    },
    'facialImageProof': 
    {
      'd': 'EInFQsE3pfRWQp3KrpH91f7YMCR2wslrTOTKL5SF6_E7',
      'u': '0ABqxMB1vXX4RL4tFUOLzn8y',
      'value': 'EIQw_2CqmmC96YYUFXTW8XSkLQU2-v9bDCyItazmKhTW'
    },
    'legalPresenceStatus': 
    {
      'd': 'EMfOp3-Dd1ZXSDUHNWE4ohAsU0Qp8tfQV_gUYKSJE7-Q',
      'u': '0AAvTJo84OMiipKf_90mn7bZ',
      'value': 'citizen'
    },
    'issuedDate': 
    {
      'd': 'EFkQuDu6twqFQ3GdSpK-jxKRYsxxCx-iSbh9RMbTbIsE',
      'u': '0ACqPN2zhcII6TVGRKYC1ckJ',
      'value': '2020-08-22T00:00:00.000000+00:00'
    },
    'expirationDate': 
    {
      'd': 'EOjYb0hwGBe1rA0Ui8foLMoCu5lItuhnodomghuyY4r9',
      'u': '0AAfMYErqhjBCf4cgG5huzT2',
      'value': '2028-09-01T00:00:00.000000+00:00'
    }
  },
  'e': 
  {
    'd': 'EFwdv1MuJAVYnyuhee0kAjDIpF44oFn5oFo9xoVwtId1',
    'u': '0AADjz761fupJiCvj4GWFBfg',
    'utahAgent': 
    {
      'd': 'EIh9Bz0kGwGyfDbDe42W1zA9Ukj2DuH-95eebY57ctI6',
      'u': '0AAsHaHfgfNEaRuRwI1Y77ry',
      'n': 'EOD4QiWo1oa6UtiEr456Lu0ba3v1Cmd6qOPoRbAdx2hD',
      's': 'EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v',
      'o': 'DI2I'
    }
  },
  'r': 
  {
    'd': 'EFPxq4WPl29szUqbrQIviOh_Ls_RlrYbp4L-fdQH0XrX', 'l': ''
  }
}

```
### Residence Schema
This defines an ACDC that a State Issuing Agent AID issues to a citizen AID to endorse that citizen's place of residence. 

```python
ResidenceSchemaSaid = 'EH5hn7CbtDFxhS3IKioC8P-e4YyZoxqzhRpr4X_e32BD'
ResidenceSchema = \
{
  '$id': 'EH5hn7CbtDFxhS3IKioC8P-e4YyZoxqzhRpr4X_e32BD',
  '$schema': 'https://json-schema.org/draft/2020-12/schema',
  'title': 'SEDI Residence Schema',
  'description': 'SEDI Residence JSON Schema for acm ACDC.',
  'credentialType': 'SEDI_Residence_ACDC_acm_message',
  'version': '0.1.0',
  'type': 'object',
  'required': ['v', 'd', 'i', 'rd', 's', 'a', 'e', 'r'],
  'properties':
  {
    'v': {'description': 'ACDC version string', 'type': 'string'},
    't': {'description': 'Message type', 'type': 'string'},
    'd': {'description': 'Message SAID', 'type': 'string'},
    'u': {'description': 'Message UE', 'type': 'string'},
    'i': {'description': 'Issuer AID', 'type': 'string'},
    'rd': {'description': 'Registry SAID', 'type': 'string'},
    's':
    {
      'description': 'Schema Section',
      'oneOf':
      [
        {'description': 'Schema Section SAID', 'type': 'string'},
        {'description': 'Schema Section Detail','type': 'object'}
      ]
    },
    'a':
    {
      'description': 'Attribute Section',
      'oneOf':
      [
        {'description': 'Attribute Section SAID','type': 'string'},
        {
          'description': 'Attribute Section Detail',
          'type': 'object',
          'required':
          [
            'd',
            'u',
            'i',
            'street',
            'city',
            'county',
            'state',
            'postcode',
            'country',
            'issuedDate',
          ],
          'properties':
          {
            'd': {'description': 'Attribute Section SAID', 'type': 'string'},
            'u': {'description': 'Attribute Section UE', 'type': 'string'},
            'i': {'description': 'Issuee AID', 'type': 'string'},
            'rd': {'description': 'Issuee Presentation Registry SAID', 'type': 'string'},
            'street':
            {
              'description': 'Street Address Block',
              'oneOf':
              [
                {'description': 'Street Address SAID', 'type': 'string'},
                {
                  'description': 'Street Address Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                   'value': {'description': 'Street Address Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'city':
            {
              'description': 'City Name Block',
              'oneOf':
              [
                {'description': 'City Name SAID', 'type': 'string'},
                {
                  'description': 'City Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'City Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'county':
            {
              'description': 'County Name Block',
              'oneOf':
              [
                {'description': 'County Name SAID', 'type': 'string'},
                {
                  'description': 'County Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'County Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'state':
            {
              'description': 'State Name Block',
              'oneOf':
              [
                {'description': 'State Name SAID', 'type': 'string'},
                {
                  'description': 'State Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'State Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'postcode':
            {
              'description': 'Postcode Block',
              'oneOf':
              [
                {'description': 'Postcode SAID', 'type': 'string'},
                {
                  'description': 'Postcode Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Postcode Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'country':
            {
              'description': 'Country Name Block',
              'oneOf':
              [
                {'description': 'Country Name SAID', 'type': 'string'},
                {
                  'description': 'Country Name Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Country Name Value', 'type': 'string'},
                  },
                  'additionalProperties': False
                },
              ]
            },
            'issuedDate':
            {
              'description': 'Issued Date Block',
              'oneOf':
              [
                {'description': 'Issued Date SAID', 'type': 'string'},
                {
                  'description': 'Issued Date Detail',
                  'type': 'object',
                  'required': ['d', 'u', 'value'],
                  'properties':
                  {
                    'd': {'description': 'Block SAID', 'type': 'string'},
                    'u': {'description': 'Bock UE', 'type': 'string'},
                    'value': {'description': 'Issued Date Value as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
                  },
                'additionalProperties': False
                },
              ]
            },
          },
          'additionalProperties': False
        }
      ]
    },
    'e':
    {
      'description': 'Edge Section',
      'oneOf':
      [
        {'description': 'Edge Section SAID', 'type': 'string'},
        {
          'description': 'Edge Section Detail',
          'type': 'object',
          'required': ['d', 'u', 'coreIdentity'],
          'properties':
          {
            'd': {'description': 'Edge Section SAID', 'type': 'string'},
            'u': {'description': 'Edge Section UE', 'type': 'string'},
            'coreIdentity':
            {
              'description': 'Core Identity Edge Block',
              'type': 'object',
              'required': ['d', 'u', 'n', 's', 'o'],
              'properties':
              {
                'd': {'description': 'Edge SAID', 'type': 'string'},
                'u': {'description': 'Edge UE', 'type': 'string'},
                'n': {'description': 'Far Node SAID', 'type': 'string'},
                's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                'o':
                {
                    'description': 'Edge Unary Operator',
                    'type': 'array',
                    'items': {'type': 'string'},
                    'minItems': 1,
                }
              },
              'additionalProperties': False
            },
            'utahAgent':
            {
              'description': 'Utah Agent Edge Block',
              'type': 'object',
              'required': ['n', 's', 'o'],
              'properties':
              {
                'd': {'description': 'Edge SAID', 'type': 'string'},
                'u': {'description': 'Edge UE', 'type': 'string'},
                'n': {'description': 'Far Node SAID', 'type': 'string'},
                's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                'o': {'description': 'Edge Unary Operator', 'type': 'string'}
              },
              'additionalProperties': False
            },
          },
          'additionalProperties': False
        }
      ]
    },
    'r':
    {
      'description': 'Rule Section',
      'oneOf':
      [
        {'description': 'Rule Section SAID', 'type': 'string'},
        {
          'description': 'Rule Section Detail',
          'type': 'object',
          'required': ['d', 'l'],
          'properties':
          {
            'd': {'description': 'Rule Section SAID', 'type': 'string'},
            'l': {'description': 'Legal Language', 'type': 'string'}
          },
        'additionalProperties': False
        }
      ]
    }
  },
  'additionalProperties': False
}
```

#### Residence Schema Example



```python
{
  'v': 'ACDCCAACAAJSONAAZA.',
  't': 'acm',
  'd': 'EBmFqhgpZL_Dc6PLgZBuKtsYfl6ajXYihYLyp7qgcbPb',
  'u': '0ACbDaiZmKZcyZLHmfVLpJYQ',
  'i': 'EKBCU6u_xObNhFc9uuz1VdntNt99xmB2fA5qz7Li-Sl-',
  'rd': 'EFiRTnCtLIUpKItzniCeWq0Bpw7smsnEKvj98dd0fDSz',
  's': 'EH5hn7CbtDFxhS3IKioC8P-e4YyZoxqzhRpr4X_e32BD',
  'a': 
  {
    'd': 'EMpm7D9f-U9sV_s4xje1kuC3LclwftjFh_g2OvaK15Jd',
    'u': '0ABZHwn29_HgPlO7tRDEONUK',
    'i': 'EDB8gKNwzurf33pV2hsyGR9XFOmitDhc0LUzDamcU2JR',
    'street': 
    {
      'd': 'EP6pdEcxu4pJbVPZ40NDav1xN-8GjmvaOyHhf-J-20ht',
      'u': '0AA2UPIBJ6WKRMVk_xgeHpSd',
      'value': '157 E 300 N'
    },
    'city': 
    {
      'd': 'EIWG8UUJR7B59VrrUJiEJpz6cv2b4YEqqdqkHec-VK_d',
      'u': '0AC2TvvDIyT60xBlRG9CjFI9',
      'value': 'Beaver'
    },
    'county': 
    {
      'd': 'EESCQE9vhr8Ys6ZE7489Lp7H2k6tKhvKfEmJQ4c0I0zi',
      'u': '0AARGwC82gXbtRbQXs4cGBpC',
      'value': 'Beaver'
    },
    'state': 
    {
      'd': 'EDliUQapanK3jK74GC9FZF3Z5UbwXBs4bFptWlzOK6b6',
      'u': '0ACkFgtF3ee459teQu6OJBRc',
      'value': 'Utah'
    },
    'postcode': 
    {
      'd': 'EK26etqB0TxOkjmr71zgFMEC8eGLB9oPjkKlrJ73iqDz',
      'u': '0AC9HltcQYMY0fjn-9zQftZW',
      'value': '84713'
    },
    'country': 
    {
      'd': 'EOySpooOzGlATtv1jrqpnkZHzHexVGGSWvUHNdKXnixD',
      'u': '0AAkcR_gzffjPL-mrjiJg9rv',
      'value': 'United States'
    },
    'issuedDate': 
    {
      'd': 'EPJPv5WXViJqWg4E5szmgJOPM7qskg6qSsiYDeELzaxG',
      'u': '0AAuZ1oTBZ6jqFp8Z7UrpKJs',
      'value': '2020-08-22T00:00:00.000000+00:00'
    }
  },
  'e': 
  {
    'd': 'ECCjjL7-Rl7J5n0rZ_iu8OEvA1Vj5uaRIjtJ_9ZcIReV',
    'u': '0ABCJgPmvnyuDB_Bj34sb7ec',
    'coreIdentity': 
    {
      'd': 'EI3XY3ybKmPrCJ4DEfKRTHKz0fxJvzlKbmtEd7IF6yJR',
      'u': '0ADYpt736nc2ZcVYFJDImwuy',
      'n': 'EGsPyGCyHtDSWP61rVsRh3F8qooP_jrX6qPHqHn7-FOg',
      's': 'EN0JtdzBmFuTUzyJG9CXZhtdu-bW0V6L95bsVBzi-cY-',
      'o': ['E1E', 'DI1I', 'NI2I']
    }
  },
  'r': 
  {
    'd': 'EFPxq4WPl29szUqbrQIviOh_Ls_RlrYbp4L-fdQH0XrX', 
    'l': ''
  }
}

```

### Replacement AID Schema
This defines an ACDC that indicates a previously endorsed citizen's AID should be replaced with a new AID. Both AIDs are citizen-sourced. One use case is when a citizen loses control over their AID and has to start over with a new one. The replacement ACDC lets the citizen notify anyone they interacted with using the old AID that it is no longer the AID to use for that citizen. The other use case is for a ward whose guardian (parent) does not rotate the ward's keys to the ward's control upon emancipation. The Ward can start fresh with a new AID but notify any past interacting parties of the new AID.

```python
ReplaceSchemaSaid = 'EPVlX-S-eWERGiXJmb7FcW75I4J08ptQ-jGglq4VRwou'
ReplaceSchema = \
{
  '$id': 'EPVlX-S-eWERGiXJmb7FcW75I4J08ptQ-jGglq4VRwou',
  '$schema': 'https://json-schema.org/draft/2020-12/schema',
  'title': 'SEDI AID Replace Schema',
  'description': 'SEDI AID Replace JSON Schema for acm ACDC.',
  'credentialType': 'SEDI_Replace_ACDC_acm_message',
  'version': '0.1.0',
  'type': 'object',
  'required': ['v', 'd', 'i', 'rd', 's', 'a', 'e', 'r'],
  'properties':
  {
    'v': {'description': 'ACDC version string', 'type': 'string'},
    't': {'description': 'Message type', 'type': 'string'},
    'd': {'description': 'Message SAID', 'type': 'string'},
    'u': {'description': 'Message UE', 'type': 'string'},
    'i': {'description': 'Issuer AID', 'type': 'string'},
    'rd': {'description': 'Registry SAID', 'type': 'string'},
    's':
    {
      'description': 'Schema Section',
      'oneOf':
      [
        {'description': 'Schema Section SAID', 'type': 'string'},
        {'description': 'Schema Section Detail','type': 'object'}
      ]
    },
    'a':
    {
      'description': 'Attribute Section',
      'oneOf':
      [
        {'description': 'Attribute Section SAID','type': 'string'},
        {
          'description': 'Attribute Section Detail',
          'type': 'object',
          'required':
          [
            'd',
            'u',
            'i',
            'issuedDate',
            'obsolete',
          ],
          'properties':
          {
            'd': {'description': 'Attribute Section SAID', 'type': 'string'},
            'u': {'description': 'Attribute Section UE', 'type': 'string'},
            'i': {'description': 'Issuee Replacement AID', 'type': 'string'},
            'rd': {'description': 'Issuee Presentation Registry SAID', 'type': 'string'},
            'issuedDate': {'description': 'Issued Date as RFC-3339/ISO-8601 time MBZ', 'type': 'string'},
            'obsolete': {'description': 'Obsolete AID', 'type': 'string'},
          },
          'additionalProperties': False
        }
      ]
    },
        'e':
    {
      'description': 'Edge Section',
      'oneOf':
      [
        {'description': 'Edge Section SAID', 'type': 'string'},
        {
          'description': 'Edge Section Detail',
          'type': 'object',
          'required': ['d', 'u', 'utahAgent'],
          'properties':
          {
            'd': {'description': 'Edge Section SAID', 'type': 'string'},
            'u': {'description': 'Edge Section UE', 'type': 'string'},
            'utahAgent':
            {
              'description': 'Utah Agent Edge Block',
              'type': 'object',
              'required': ['d', 'u', 'n', 's', 'o'],
              'properties':
              {
                'd': {'description': 'Edge SAID', 'type': 'string'},
                'u': {'description': 'Edge UE', 'type': 'string'},
                'n': {'description': 'Far Node SAID', 'type': 'string'},
                's': {'description': 'Far Node Schema SAID', 'type': 'string'},
                'o': {'description': 'Edge Unary Operator', 'type': 'string'}
              },
              'additionalProperties': False
            }
          },
          'additionalProperties': False
        }
      ]
    },
    'r':
    {
      'description': 'Rule Section',
      'oneOf':
      [
        {'description': 'Rule Section SAID', 'type': 'string'},
        {
          'description': 'Rule Section Detail',
          'type': 'object',
          'required': ['d', 'l'],
          'properties':
          {
            'd': {'description': 'Rule Section SAID', 'type': 'string'},
            'l': {'description': 'Legal Language', 'type': 'string'}
          },
        'additionalProperties': False
        }
      ]
    }
  },
  'additionalProperties': False
}
```

#### Example Replacement ACDC


```python
{
  'v': 'ACDCCAACAAJSONAANu.',
  't': 'acm',
  'd': 'EPF4V1wuieZE15yOoWOsvCm0_1C-I0G7efTmIiUQ5sWa',
  'u': '0ACY9KcZQQ_FXQdCLG0vODyG',
  'i': 'EKBCU6u_xObNhFc9uuz1VdntNt99xmB2fA5qz7Li-Sl-',
  'rd': 'EH7Rrg0KDFmADQ4BMWp-JXL-HM4trbphW8b7bsbFxVBi',
  's': 'EPVlX-S-eWERGiXJmb7FcW75I4J08ptQ-jGglq4VRwou',
  'a': 
  {
    'd': 'EFAl_GXl1vxkCYb0vhBkDklrYkut-RePW1meTaib1ZdQ',
    'u': '0AB36SRUqDVDYq5UulmV5w1v',
    'i': 'EPvkhZTKfAte3QhfD-O3eKY2dwZqcLQ9OVIVGjT9edmR',
    'issuedDate': '2020-10-15T00:00:00.000000+00:00',
    'obsolete': 'EKr8JLtfqWCmHrxO3yu8ocS2n9o0Tlspeaqm9ZOf3FM1'
  },
  'e': 
  {
    'd': 'EBKQkzURVLRCnEE1OV63JGc88i8P2_f2L5jveyPSVwoB',
    'u': '0ABxzCU6Wz_mzY2KiB0u4Xgi',
    'utahAgent': 
    {
      'd': 'EF65dodygj3h3e_J2YKai-ILHkbTSo6S-L2BUCcXN2oi',
      'u': '0ADBRTatQM2Y5NzW9by_ko2L',
      'n': 'EOD4QiWo1oa6UtiEr456Lu0ba3v1Cmd6qOPoRbAdx2hD',
      's': 'EGx4BLclkjhaK1501guyBifxuTTJLwuK61InBTdkKF7v',
      'o': 'I2I'
    }
  },
  'r': 
  {
    'd': 'EFPxq4WPl29szUqbrQIviOh_Ls_RlrYbp4L-fdQH0XrX', 
    'l': ''
  }
}

```
## SEDI ACDC Usage Examples
