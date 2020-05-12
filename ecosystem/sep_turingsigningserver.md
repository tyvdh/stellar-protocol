## Preamble

```
SEP: TBD
Title: Turing Signing Servers: Turing complete smart contracts via decentralized multisig machines
Author: [Tyler van der Hoeven, @tyvdh, tyler@stellar.org](https://github.com/tyvdh)
Track: Standard
Status: Draft
Created: 2020-05-08
Updated: 2020-05-08
Version: 0.0.1
Discussion: TBD
```

## Summary
"If you can't explain it simply, you don't understand it well enough." Provide a simplified and
layman-accessible explanation of the SEP.

## Motivation
Should clearly explain why the existing protocol specification is inadequate to address the problem
that the SEP solves. SEP submissions without sufficient motivation may be rejected outright.

## Abstract
A short (~200 word) description of the technical issue being addressed.

## Use Cases

## Specification

### Authentication

All endpoints are publically accessible and require no authentication with the exception of the `UPDATE /contract/:hash` which requires the contents to be signed by the contract secret key as part of the body request.

### Content Type

All endpoints accept `Content-Type: application/json` with the exception of `POST|UPDATE /contract/:hash` which accepts `Content-Type: multipart/form`.

All endpoints respond with `Content-Type: application/json` with the exception of `GET /contract/:hash/run/collate` which responds with `Content-Type: text/plain`.

### Errors

If an error occurs when calling any endpoint, an appropriate HTTP status code
will be returned along with an error response.

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
`error` | string | A description of the error.

##### Example

```json
{
   "error": "..."
}
```

### Endpoints

- [`POST /contract/<hash>`](#post-contract-hash)
<!-- - [`POST /contract/<hash>/collate`](#post-contract-hash-collate) -->
- [`GET /contract/<hash>`](#get-contract-hash)
<!-- - [`GET /contract/<hash>/collate`](#get-contract-hash-collate) -->
- [`PUT /contract/<hash>`](#put-contract-hash)
<!-- - [`PUT /contract/<hash>/collate`](#put-contract-hash-collate) -->
- [`POST /contract/<hash>/run`](#post-contract-hash-run)
<!-- - [`POST /contract/<hash>/run/collate`](#post-contract-hash-run-collate) -->

#### `POST /contract/<hash>`

This endpoint uploads a smart contract.


##### Authentication
[`NONE`]

##### Request

###### Fields

Name | Type | Description
-----|------|------------
`contract` | file | The contract function file the contract creator is uploading to the turing server.
`turrets` | string | base64 encoded comma-separated list of urls where this contract will be hosted.

###### Example

```json5
{
  "contract": {
    ...
    "path": "/path/to/contract.js"
    ...
  },
  "turrets": "aHR0c...J1eno="
}
```

##### Response

###### Fields

Name | Type | Description
-----|------|------------
`vault` | string | Turing signing server account where signing fees must be paid to
`signer` | string | The signing account for this contract on this turing server. This is what you'll add to contract or user accounts for multisig protection.
`fee` | string | The XLM fee required by this turing server to sign for contracts.

##### Example

```json
{
    "vault": "GD6J...",
    "signer": "GDLZ...",
    "fee": "0.5"
}
```

## Design Rationale
The rationale fleshes out the specification by describing what motivated this particular design and
why certain design decisions were made. It should describe alternate designs that were
considered and related work, e.g. how the feature is supported in other protocols. The rationale
may also provide evidence of consensus within the community, and should discuss important
objections or concerns raised during discussion.

## Implementations

## Limitations

## Security Concerns
All SEPs should carefully consider areas where security may be a concern, and document them
accordingly. If a change does not have security implications, write "N/A".

## Legal