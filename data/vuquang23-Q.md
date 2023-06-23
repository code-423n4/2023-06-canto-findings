# Low Risk Issues

**1. Should not allow creating a pool if `counterpartyDenom` param is standard denomination**

The file `pool.go` in the `Canto/x/coinswap/keeper` package contains a function called `CreatePool`, which takes a parameter named `counterpartyDenom`. This function is responsible for creating a pool that consists of two tokens: one with the standard denomination and another with a counterparty denomination. It is important to ensure that we do not create a liquidity pool with two identical tokens. Therefore, we need to verify whether the counterpartyDenom is the standard denomination or not. If it is, we should return an error to indicate that creating a pool with identical tokens is not allowed.

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

When flag variable `found` is set to `true`, it should break the loop to reduce computation resource.

https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L48

**2. Should validate `WhitelistedChannels`**

In the function `Validate` at line 89 of the file `Canto/x/onboarding/types/params.go`, it currently only validates the fields `EnableOnboarding` and `AutoSwapThreshold` using the `validateBool` and `validateAutoSwapThreshold` functions, respectively, but it does not validate the `WhitelistedChannels` field. Although this does not cause a bug in the case, we should validate the `WhitelistedChannels` field using the `validateWhitelistedChannels` function to ensure more consistency in the source code.

https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/types/params.go#L89


Function change proposal
```golang
// Validate checks that the fields have valid values
func (p Params) Validate() error {
	if err := validateBool(p.EnableOnboarding); err != nil {
		return err
	}
	if err := validateAutoSwapThreshold(p.AutoSwapThreshold); err != nil {
		return err
	}
	if err := validateWhitelistedChannels(p.WhitelistedChannels); err != nil {
		return err
	}
	return nil
}
```
