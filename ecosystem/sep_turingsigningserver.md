## Preamble

```
SEP: TBD
Title: Turing Signing Servers: Decentralized turing complete transaction creation.
Author: [Tyler van der Hoeven, @tyvdh, tyler@stellar.org](https://github.com/tyvdh)
Track: Standard
Status: Draft
Created: 2020-05-08
Updated: 2020-05-12
Version: 0.0.1
Discussion: TBD
```

## Summary
<!-- "If you can't explain it simply, you don't understand it well enough." Provide a simplified and
layman-accessible explanation of the SEP. -->

## Motivation
<!-- Should clearly explain why the existing protocol specification is inadequate to address the problem
that the SEP solves. SEP submissions without sufficient motivation may be rejected outright. -->

## Abstract
<!-- A short (~200 word) description of the technical issue being addressed. -->

## Use Cases

## Specification

### Content Type

All endpoints accept `Content-Type: application/json` with the exception of `POST|UPDATE /contract/<hash>` which accepts `Content-Type: multipart/form-data`.

All endpoints respond with `Content-Type: application/json` with the exception of `GET /contract/<hash>/run/collate` and `PUT /contract/<hash>` which respond with `Content-Type: text/plain`.

### Errors

If an error occurs when calling any endpoint, an appropriate HTTP status code
will be returned along with an error message.

#### Status Code

Common HTTP status codes may be returned for a server. In particular the following are expected:

Status Code | Name | Reason
-----|------|------------
`400` | Bad Request | The request is invalid in anyway.
`404` | Not Found | The path is not recognized by the server, or the address does not have a contract.

#### Response

##### Fields

Name | Type | Description
-----|------|------------
`message` | string | A description of the error.

##### Example

```json
{
  "message": "..."
}
```

### Endpoints

- [`POST /contract/<hash>`](#post-contract-hash)
- [`POST /contract/<hash>/collate`](#post-contract-hash-collate)
<br><br>
- [`GET /contract/<hash>`](#get-contract-hash)
- [`GET /contract/<hash>/collate`](#get-contract-hash-collate)
<br><br>
- [`PUT /contract/<hash>`](#put-contract-hash)
- [`PUT /contract/<hash>/collate`](#put-contract-hash-collate)
<br><br>
- [`POST /contract/<hash>/run`](#post-contract-hash-run)
- [`POST /contract/<hash>/run/collate`](#post-contract-hash-run-collate)

#### `POST /contract/<hash>`

This endpoint uploads a smart contract.

##### Request

###### Fields

Name | Type | Description
-----|------|------------
`contract` | file | The contract function file the contract creator is uploading to the turing server.
`turrets` | string | base64 encoded comma-separated list of urls where this contract will be hosted.
`fields` | array\<\{name: string, type: string\|object\|array, description: string, rules: string\}\> | Array of objects detailing the expected incoming request parameters. `name` is the key for the outgoing request. `type` is the key value type, e.g. `"string"` or `"number"`. `description` is the human readable display explanation for the intention behind the given field. `rules` is a service provider explanation for any restrictions which could cause the request to be rejected. Intended to help build regex flows or help explain incoming errors from a given contract.

###### Example

```json5
// TODO: include example for both array and object types

{
  "contract": {
    ...
    "path": "/path/to/contract.js",
    ...
  },
  "turrets": "aHR0c...J1eno=",
  "fields": [
    {
      "name": "to",
      "type": "string",
      "description": "Where should we send TYLERCOIN to?",
      "rules": "Must be a valid Stellar address"
    },
    {
      "name": "source",
      "type": "string",
      "description": "What's the source account for this transaction?",
      "rules": "Must be a valid Stellar address, often the same as the `to` address"
    },
    {
      "name": "amount",
      "type": "string",
      "description": "TYLERCOIN is purchased 1:1 for XLM. How much do you want to pay & receive?",
      "rules": "Must be a valid numerical amount above any TSS signing fee for this contract"
    }
  ]
}
```

##### Response

###### Fields

Name | Type | Description
-----|------|------------
`vault` | string | Turing signing server account where signing fees must be paid to
`signer` | string | The signing account for this contract on this turing server. This is what you'll add to contract or user accounts for multisig protection.
`fee` | string | The XLM fee required by this turing server to sign for contracts.

###### Example

```json
{
  "vault": "GD6J...",
  "signer": "GDLZ...",
  "fee": "0.5"
}
```

#### `GET /contract/<hash>`

This endpoint gets the details surrounding a smart contract.

##### Response

###### Fields

Name | Type | Description
-----|------|------------
`vault` | string | Turing signing server account where signing fees must be paid to
`signer` | string | The signing account for this contract on this turing server. This is what you'll add to contract or user accounts for multisig protection.
`fee` | string | The XLM fee required by this turing server to sign for contracts.
`fields` | array\<\{name: string, type: string\|object\|array, description: string, rules: string\}\> | Array of objects detailing the expected incoming request parameters. `name` is the key for the outgoing request. `type` is the key value type, e.g. `"string"` or `"number"`. `description` is the human readable display explanation for the intention behind the given field. `rules` is a service provider explanation for any restrictions which could cause the request to be rejected. Intended to help build regex flows or help explain incoming errors from a given contract.

###### Example

```json
{
  "vault": "GD6J...",
  "signer": "GDLZ...",
  "fee": "0.5",
  "fields": [
    {
      "name": "to",
      "type": "string",
      "description": "Where should we send TYLERCOIN to?",
      "rules": "Must be a valid Stellar address"
    },
    {
      "name": "source",
      "type": "string",
      "description": "What's the source account for this transaction?",
      "rules": "Must be a valid Stellar address, often the same as the `to` address"
    },
    {
      "name": "amount",
      "type": "string",
      "description": "TYLERCOIN is purchased 1:1 for XLM. How much do you want to pay & receive?",
      "rules": "Must be a valid numerical amount above any TSS signing fee for this contract"
    }
  ]
}
```

#### `PUT /contract/<hash>`

This endpoint updates an existing contract's turret list.

##### Request

###### Fields

Name | Type | Description
-----|------|------------
`turrets` | string | base64 encoded comma-separated list of urls where this contract will be hosted.
`signature` | string | base64 encoded contract keypair signature of the `turrets` string.

###### Example

```json
{
  "turrets": "aHR0c...J1eno=",
  "signature": "u5wYk...9HVAA=="
}
```

##### Response

###### Fields

Status Code | Name | Reason
-----|------|------------
`200` | OK | Contract turret list was updated

###### Example

```text
OK
```

#### `POST /contract/<hash>/run`

This endpoint runs a smart contract.

##### Request

###### Fields

Fields for contract runs will be determined by the contract and discoverable by requesting `GET /contract/<hash>`. Will be confined to a normal JSON object and should be relatively simple.

###### Example

```json
{
  "to": "GAWS...",
  "source": "GAWS...",
  "amount": "500"
}
```

##### Response

###### Fields

Name | Type | Description
-----|------|------------
`xdr` | string | base64 transaction envelope produced by the contract
`signer` | string | The signer account for the signature for the contract xdr
`signature` | string | The signature intended to be attached to the xdr should it be needed

###### Example

```json
{
  "xdr": "AAAAAC0mu...EtAAAAAAAAAAAA=",
  "signer": "GDLZ...",
  "signature": "UA6NqXp...qsHVJBAQ=="
}
```

#### `POST /contract/<hash>/run/collate`

This endpoint runs a smart contract collation which will gather all the signatures where this contract is stored via the turrets list and respond with a single signed XDR.

##### Request

###### Body Fields

Fields for contract runs will be determined by the contract and discoverable by requesting `GET /contract/<hash>`. Will be confined to a normal JSON object and should be relatively simple.

###### Query Params

Name | Type | Description
-----|------|------------
`signatures` | number | number of signatures the collation should attach to the returned XDR

###### Example

```json
{
  "to": "GAWS...",
  "source": "GAWS...",
  "amount": "500"
}
```

##### Response

###### Fields

Status Code | Type | Description
-----|------|------------
`200` | string | base64 signed transaction envelope produced by the collated contract

###### Example

```text
AAAAAC0mu...EtAAAAAAAAAAAA=
```

## Design Rationale
<!-- The rationale fleshes out the specification by describing what motivated this particular design and why certain design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other protocols. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion. -->

## Implementations

## Limitations

## Security Concerns
<!-- All SEPs should carefully consider areas where security may be a concern, and document them accordingly. If a change does not have security implications, write "N/A". -->

## Legal
