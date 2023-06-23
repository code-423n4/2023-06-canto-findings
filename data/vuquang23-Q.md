# Low Risk Issues

**1. Should not allow creating a pool if `counterpartyDenom` param is standard denomination**

The file `pool.go` in the `Canto/x/coinswap/keeper` package contains a function called `CreatePool`, which takes a parameter named `counterpartyDenom`. This function is responsible for creating a pool that consists of two tokens: one with the standard denomination and another with a counterparty denomination. It is important to ensure that we do not create a liquidity pool with two identical tokens. Therefore, we need to verify whether the `counterpartyDenom` is the standard denomination or not. If it is, we should return an error to indicate that creating a pool with identical tokens is not allowed.


https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/coinswap/keeper/pool.go#L15


Function change proposal

```golang
// CreatePool create a liquidity that saves relevant information about popular pool tokens
func (k Keeper) CreatePool(ctx sdk.Context, counterpartyDenom string) (types.Pool, error) {
	standardDenom := k.GetStandardDenom(ctx)
	if counterpartyDenom == standardDenom {
		return types.Pool{}, fmt.Errorf("unaccepted counterparty denomination")
	}
	sequence := k.getSequence(ctx)
	lptDenom := types.GetLptDenom(sequence)
	pool := &types.Pool{
		Id:                types.GetPoolId(counterpartyDenom),
		StandardDenom:     standardDenom,
		CounterpartyDenom: counterpartyDenom,
		EscrowAddress:     types.GetReservePoolAddr(lptDenom).String(),
		LptDenom:          lptDenom,
	}
	k.setSequence(ctx, sequence+1)
	k.setPool(ctx, pool)
	return *pool, nil
}
```

# Non-critical Issues

**1. Should break the loop when data is found**

In file `Canto/x/onboarding/keeper/ibc_callbacks.go`, there is a loop with comment:

```
// check source channel is in the whitelist channels
```

When flag variable `found` is set to `true`, it should break the loop to save computation resource.

https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L48

