---
arc: <to be assigned>
title: ABI Versioning
description: Add a versioning scheme to the ABI
author: Joe Polny (@joe-p)
discussions-to: <URL>
status: Draft
type: Standards Track
category (*only required for Standards Track): ARC
created: 2023-07-15
requires (*optional): [ARC-0004](./arc-0004.md)
---

## Abstract
The Algorand ABI is defined in [ARC-0004](./arc-0004.md). As the protocol and desires of the ABI changes, there will be changes to the ABI. As such, there should be a versioning number to make it clear what versions of the ABI smart contracts adhere to and what versions of the ABI off-chain code supports.

## Motivation
Without any versioning scheme, it will be very hard to follow the evolution of the Algorand ABI.

## Specification
The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in <a href="https://www.ietf.org/rfc/rfc2119.txt">RFC-2119</a>.

Every ABI change is versioned `MAJOR.MINOR`. `MAJOR` is incremeneted when there are breaking changes and `MINOR` is incremented for the addition of backwards-compatible features.

Any ABI Interface or Contract JSON description that adheres to ABI standards beyond [ARC-0004](./arc-0004.md) **MUST** inlcude a `version` key in the top-level of the interface. Interface or Contract JSON descriptions that only adhere to [ARC-0004](./arc-0004.md) **SHOULD** have `"version": "1.0"` at the top level, but descriptions without a `version` key **MUST** be interepreted to implcitly be version 1.0.

## Rationale
The `MAJOR.MINOR` scheme is used to make it clear what features a given description supports and when clients will become incompatible with  the respective contract.

Since `version` is a new key being added, all existing descriptions without will not have the `version` key, thus the omission of the key is implying version 1.0 ([ARC-0004](./arc-0004.md)).

## Backwards Compatibility
This ARC is fully backwards compatible with [ARC-0004](./arc-0004.md) and all clients. There will, however, need to be additional effort for clients to parse the `version` key for future evolutions of the ABI.

## Test Cases
N/A

## Reference Implementation

An [ARC-0004](./arc-0004.md) contract description witht the `version` key
```json
{
  "version": "1.0",
  "name": "Calculator",
  "desc": "Contract of a basic calculator supporting additions and multiplications. Implements the Calculator interface.",
  "networks": {
    "wGHE2Pwdvd7S12BL5FaOP20EGYesN73ktiC1qzkkit8=": { "appID": 1234 },
    "SGO1GKSzyE7IEPItTxCByw9x8FmnrCDexi9/cOUJOiI=": { "appID": 5678 },
  },
  "methods": [
    {
      "name": "add",
      "desc": "Calculate the sum of two 64-bit integers",
      "args": [
        { "type": "uint64", "name": "a", "desc": "The first term to add" },
        { "type": "uint64", "name": "b", "desc": "The second term to add" }
      ],
      "returns": { "type": "uint128", "desc": "The sum of a and b" }
    },
    {
      "name": "multiply",
      "desc": "Calculate the product of two 64-bit integers",
      "args": [
        { "type": "uint64", "name": "a", "desc": "The first factor to multiply" },
        { "type": "uint64", "name": "b", "desc": "The second factor to multiply" }
      ],
      "returns": { "type": "uint128", "desc": "The product of a and b" }
    }
  ]
}
```

## Security Considerations
None

## Copyright
Copyright and related rights waived via <a href="https://creativecommons.org/publicdomain/zero/1.0/">CCO</a>.