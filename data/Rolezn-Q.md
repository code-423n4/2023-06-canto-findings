## QA Summary<a name="QA Summary">

| |Issue|Contexts|
|-|:-|:-:|
| [QA&#x2011;1](#QA&#x2011;1) | Loop can finish iterating earlier | 1 |
| [QA&#x2011;2](#QA&#x2011;2) | Multiple `GetParams` calls in `OnRecvPacket` | 2 |
| [QA&#x2011;3](#QA&#x2011;3) | Module account type check | 1 |


Total: 4 contexts over 3 issues


### <a href="#QA Summary">[QA&#x2011;1]</a><a name="QA&#x2011;1"> Loop can finish iterating earlier

In `OnRecvPacket` function, as soon as `s == packet.DestinationChannel` is `true`, it sets `found`, but still iterates the rest of the channels.

Instead when setting `found` to `true`, we can add `break` that ends the loop, skipping the rest of the `params.WhitelistedChannels` that didn't get checked yet.

#### <ins>Proof Of Concept</ins>

```go
	var found bool
	for _, s := range params.WhitelistedChannels {
		if s == packet.DestinationChannel {
			found = true
		}
	}

	if !found {
		return ack
	}
```

https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L45-L54

#### <ins>Recommended Mitigation Steps</ins> 

```go
	var found bool
	for _, s := range params.WhitelistedChannels {
		if s == packet.DestinationChannel {
			found = true
			break
		}
	}
	if !found {
		return ack
	}
```


### <a href="#QA Summary">[QA&#x2011;2]</a><a name="QA&#x2011;2"> Multiple `GetParams` calls in `OnRecvPacket`


The following calls `k.GetParams(ctx)` twice in the `OnRecvPacket` function. It may be more efficient to call it once at the start of the function and store the result in a variable for later use, in this case `params` can be reused.


#### <ins>Proof Of Concept</ins>

```go
39: params := k.GetParams(ctx)
```
https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L39

```go
87: autoSwapThreshold := k.GetParams(ctx).AutoSwapThreshold
```
https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L87


### <a href="#QA Summary">[QA&#x2011;3]</a><a name="QA&#x2011;3"> Module account type check

The `OnRecvPacket` function retrieves the account associated with the recipient from `k.accountKeeper.GetAccount(ctx, recipient)`. Then it checks if the account is a module account, which is a special type of account typically used by the system modules for specific functionalities (e.g., fee collection, staking pool).

If the recipient account is a module account, the function immediately returns the acknowledgment without proceeding with the rest of the logic. It is indicated that onboarding is not supported for module accounts, which is why the function returns `ack`.

While this check is a reasonable. It is possible that not all module accounts should be treated the same - some might actually need to go through the rest of the logic in the function. For instance, a staking module account might be required to participate in coin swaps and conversions. In this case, a more fine-grained check is advised. 

Instead of returnig `ack` on all module accounts, the project could maintain a whitelist of specific module accounts that should be allowed to proceed. The specific implementation would depend on the actual requirements of your application. This will also allow future module accounts that their onboarding to be supported simply by adding them to a whitelist.

#### <ins>Proof Of Concept</ins>

```go
	// onboarding is not supported for module accounts
	if _, isModuleAccount := account.(authtypes.ModuleAccountI); isModuleAccount {
		return ack
	}
```

