# QA Report
## Summary
Total 05 Low
### Low Risk Issues
- [L-01] Type mismatch for amount packet field between code and spec
- [L-02] Wrong emit event
- [L-03] Update to the latest version of idc
- [L-04] Ensure propagated errors in fmt.Errorf errors are wrapped with %w and not %s nor %v 
- [L-05] Update Denom regex to support more DID characters 

## [L-01] Type mismatch for amount packet field between code and spec
### Description:
```
File: x/onboarding/keeper/ibc_callbacks.go
72:        var data transfertypes.FungibleTokenPacketData
>>> Amount string `protobuf:"bytes,2,opt,name=amount,proto3" json:"amount,omitempty"` 
```
[Link to code](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L72)
According [specifications](https://github.com/cosmos/ibc/tree/main/spec/app/ics-020-fungible-token-transfer#data-structures) the ICS20 specification uses uint256 for FungibleTokenPacketData.Amount. The code uses string
### Recommendation:
Change the spec

## [L-02] Wrong emit event
### Description:
```
File: x/onboarding/keeper/ibc_callbacks.go
101:                                        sdk.NewAttribute(coinswaptypes.AttributeValueSender, recipient.String()), // @audit must be sender.String()),
```
[This line](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L101) emits the wrong event
### Recommendation:
Change from `recipient.String` to `sender.String`

## [L-03] Update to the latest version of idc
[Canto is using idc version: v6.0.0](https://pkg.go.dev/github.com/Canto-Network/Canto/v6/ibc), [consider updating to idc version: v11.0.2](https://pkg.go.dev/github.com/evmos/evmos/v11/ibc)

## [L-04] Ensure propagated errors in fmt.Errorf errors are wrapped with %w and not %s nor %v 
File: https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/types/params.go
[References](https://github.com/cosmos/cosmos-sdk/issues/16536)

## [L-05] Update Denom regex to support more DID characters 
```
File: cosmos-sdk@v.0.45.9/types/coin.go
763:	// a letter, a number or a separator ('/').
764:	reDnmString = `[a-zA-Z][a-zA-Z0-9/-]{2,127}`
```
According [references](https://github.com/cosmos/cosmos-sdk/pull/9699) the project should add the missing pieces in coin.go
