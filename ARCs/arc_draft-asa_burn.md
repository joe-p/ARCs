---
arc: <to be assigned>
title: ASA Burning App
description: Standardized Application for Burning ASAs
author: Joe Polny (@joe-p)
discussions-to: <URL>
status: Draft
type: Standards Track
category (*only required for Standards Track): ARC
created: 2023-09-15
requires (*optional): <ARC number(s)>
---

## Abstract
This ARC provides TEAL which would deploy a application that can be used for burning Algorand Standard Assets. The goal is to have the apps deployed on the public networks using this TEAL to provide a standardized burn address and app ID.

## Motivation
Currently there is no official way to burn ASAs. While one can currently deploy their own app or rekey an account holding the asset to a vanity address, having a standardized address for burned assets allows explorers and dapps to display burnt supply for every ASA.

## Specification

### ARC4 JSON Description
```json
{
  "name": "ASABurnApp",
  "desc": "",
  "methods": [
    {
      "name": "optIntoASA",
      "args": [
        {
          "name": "mbrPayment",
          "type": "pay",
          "desc": "The payment that covers the opt-in MBR for the contract"
        },
        {
          "name": "asa",
          "type": "asset",
          "desc": "The ASA to opt in to"
        }
      ],
      "desc": "Sends an inner transaction to opt the contract account into an ASA. The fee forthe inner transaction must be covered by the sender. Will fail if the contractis already opted in to the asset or if the asset has a clawback address set.",
      "returns": {
        "type": "void",
        "desc": ""
      }
    }
  ]
}
```

## Rationale
This simple application is only able to opt in to ASAs but not send them. As such, once an ASA has been sent to the app address it is effectively burnt.

The app prevents itself from opting into an asset that has a clawback address set to prevent tokens that were once considered to be burnt to go back into circulation. This means any token with a clawback address set cannot be burnt via this app.

## Backwards Compatibility
N/A

## Test Cases
TODO

## Reference Implementation
```
#pragma version 9

// This TEAL was generated by TEALScript v0.42.0
// https://github.com/algorand-devrel/TEALScript

// The following ten lines of TEAL handle initial program flow
// This pattern is used to make it easy for anyone to parse the start of the program and determine if a specific action is allowed
// Here, action refers to the OnComplete in combination with whether the app is being created or called
// Every possible action for this contract is represented in the switch statement
// If the action is not implmented in the contract, its repsective branch will be "NOT_IMPLMENTED" which just contains "err"
txn ApplicationID
int 0
>
int 6
*
txn OnCompletion
+
switch create_NoOp NOT_IMPLEMENTED NOT_IMPLEMENTED NOT_IMPLEMENTED NOT_IMPLEMENTED NOT_IMPLEMENTED call_NoOp

NOT_IMPLEMENTED:
	err

// optIntoASA(asset,pay)void
//
// Sends an inner transaction to opt the contract account into an ASA. The fee for
// the inner transaction must be covered by the sender. Will fail if the contract
// is already opted in to the asset or if the asset has a clawback address set.
// 
// @param mbrPayment The payment that covers the opt-in MBR for the contract
// @param asa The ASA to opt in to
abi_route_optIntoASA:
	byte 0x // push empty bytes to fill the stack frame for this subroutine's local variables

	// asa: asset
	txna ApplicationArgs 1
	btoi
	txnas Assets

	// mbrPayment: pay
	txn GroupIndex
	int 1
	-
	dup
	gtxns TypeEnum
	int pay
	==
	assert

	// execute optIntoASA(asset,pay)void
	callsub optIntoASA
	int 1
	return

optIntoASA:
	proto 3 0

	// contracts/asa-burn-app.algo.ts:14
	// assert(!globals.currentApplicationAddress.hasAsset(asa))
	global CurrentApplicationAddress
	frame_dig -2 // asa: asset
	asset_holding_get AssetBalance
	swap
	pop
	!
	assert

	// contracts/asa-burn-app.algo.ts:15
	// assert(asa.clawback === globals.zeroAddress)
	frame_dig -2 // asa: asset
	asset_params_get AssetClawback
	assert
	global ZeroAddress
	==
	assert

	// contracts/asa-burn-app.algo.ts:17
	// preMBR = globals.currentApplicationAddress.minBalance
	global CurrentApplicationAddress
	acct_params_get AcctMinBalance
	assert
	frame_bury -3 // preMBR: uint64

	// contracts/asa-burn-app.algo.ts:19
	// sendAssetTransfer({
	//       assetReceiver: globals.currentApplicationAddress,
	//       xferAsset: asa,
	//       assetAmount: 0,
	//     })
	itxn_begin
	int axfer
	itxn_field TypeEnum

	// contracts/asa-burn-app.algo.ts:20
	// assetReceiver: globals.currentApplicationAddress
	global CurrentApplicationAddress
	itxn_field AssetReceiver

	// contracts/asa-burn-app.algo.ts:21
	// xferAsset: asa
	frame_dig -2 // asa: asset
	itxn_field XferAsset

	// contracts/asa-burn-app.algo.ts:22
	// assetAmount: 0
	int 0
	itxn_field AssetAmount

	// Fee field not set, defaulting to 0
	int 0
	itxn_field Fee

	// Submit inner transaction
	itxn_submit

	// contracts/asa-burn-app.algo.ts:25
	// verifyTxn(mbrPayment, {
	//       receiver: globals.currentApplicationAddress,
	//       amount: { greaterThanEqualTo: globals.currentApplicationAddress.minBalance - preMBR },
	//     })
	// verify receiver
	frame_dig -1 // mbrPayment: pay
	gtxns Receiver
	global CurrentApplicationAddress
	==
	assert

	// verify amount
	frame_dig -1 // mbrPayment: pay
	gtxns Amount
	global CurrentApplicationAddress
	acct_params_get AcctMinBalance
	assert
	frame_dig -3 // preMBR: uint64
	-
	>=
	assert
	retsub

abi_route_defaultTEALScriptCreate:
	int 1
	return

create_NoOp:
	txn NumAppArgs
	bz abi_route_defaultTEALScriptCreate
	err

call_NoOp:
	method "optIntoASA(pay,asset)void"
	txna ApplicationArgs 0
	match abi_route_optIntoASA
	err
```

## Security Considerations
It should be noted that once an asset is sent to the contract there will be no way to recover the asset.

To do the simplicity of a TEAL, an audit is not needed. The contract has no code paths which can send tokens, thus there is no concern of an exploit that undoes burning.

## Copyright
Copyright and related rights waived via <a href="https://creativecommons.org/publicdomain/zero/1.0/">CCO</a>.