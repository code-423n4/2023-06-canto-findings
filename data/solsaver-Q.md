## Return nil to be consistent

If the `err` is no tnil, then `coins` is returned which is uninitialized, and will use the default value.

All the other error handling return a `nil` instead of uninitialized variables. This instance can be updated to follow the same pattern to avoid any confusion and incorrect usage by the caller methods.

```
// GetPoolBalances return the liquidity pool by the specified anotherCoinDenom
func (k Keeper) GetPoolBalances(ctx sdk.Context, escrowAddress string) (coins sdk.Coins, err error) {
	address, err := sdk.AccAddressFromBech32(escrowAddress)
	if err != nil {
		return coins, err
	}
    ...
```

Code Link: https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/coinswap/keeper/pool.go#L73

## Break early from loop

In the loop, if the `if` condition matches, the loop can be broken from earlier by using the `break` statement right after with `found = true`.

```
    ...
	// check source channel is in the whitelist channels
	var found bool
	for _, s := range params.WhitelistedChannels {
		if s == packet.DestinationChannel {
			found = true
		}
	}
    ...
```

Code Link: https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L48