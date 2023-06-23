# QA Report

8 low issues and 2 non-critical found.

| Id  | Issue                                                                                                  |
|-----|--------------------------------------------------------------------------------------------------------|
| L-1 | [Params has both value and reference receivers](#l-1-params-has-both-value-and-reference-receivers)    |
| L-2 | [Misleading method GoDoc calculateWithExactInput](#l-2-misleading-method-godoc-calculatewithexactinput) |
| L-3 | [Misleading method GoDoc GetPoolBalances](#l-3-misleading-method-godoc-getpoolbalances)                |
| L-4 | [Deprecated method on EmitEvent used](#l-4-deprecated-method-on-emitevent-used)                        |
| L-5 | [Silent fail on no whitelisting](#l-5-silent-fail-on-no-whitelisting)                                  |
| L-6 | [Silent fail on recipient account being a module](#l-6-silent-fail-on-recipient-account-being-a-module) |
| L-7 | [Silent fail on missing ERC20 mapping](#l-7-silent-fail-on-missing-erc20-mapping)                      |
| L-8 | [Silent fail on trading pair disable](#l-8-silent-fail-on-trading-pair-disable)                        |
| NC-1 | [Dead GoDoc params for TradeExactInputForOutput](#nc-1-dead-godoc-params-for-tradeexactinputforoutput) |
| NC-2 | [Dead GoDoc params for TradeInputForExactOutput](#nc-2-dead-godoc-params-for-tradeinputforexactoutput) |


## Low Risk

### L-1 Params has both value and reference receivers

Params has two methods, one uses pointer and the other a value reciver. GoLang documentation recommends choosing either one or the other.
(Arugments being consistency, readability and peformance)
https://go.dev/doc/faq#methods_on_values_or_pointers

The Validate method is currently using a value receiever, this can be changed to a pointer without side affect (avoiding copying the struct on each call).

#### Reommended mitigation
Change the `Params` from value to reference.

```
x/onboarding/keeper/params.go:L89

- func (p Params) Validate() error {
+ func (p *Params) Validate() error {
```

### L-2 Misleading method GoDoc calculateWithExactInput

GoDoc paramater of `soldTokenDenom` on `swap:calculateWithExactInput` is not a function parameter, however there is a boughtTokenDenom parameter (which would be an inversion of the GoDoc description).

#### Reommended mitigation

```
x/coinswap/keeper/swap.go:L33

- @param soldTokenDenom : received token's denom
+ @param boughtTokenDenom : demonination of token brought
```


### L-3 Misleading method GoDoc GetPoolBalances

GoDoc mehod comment for `pool:GetPoolBalances` does not describe the method correctly (it may be a copy, paste & edit error from the proceeding method in the file).

#### Reommended mitigation

```
x/coinswap/keeper/pool.go:L69

- // GetPoolBalances return the liquidity pool by the specified anotherCoinDenom
+ // GetPoolBalances returns the coin balances of the liquidity pool by the specified address
```


### L-4 Deprecated method on EmitEvent used

EmitEvent is deprecated in favor of EmitTypedEvent
https://pkg.go.dev/github.com/cosmos/cosmos-sdk/types@v0.45.9#EventManager.EmitEvent

```
x/onboarding/kepper/ibc_callbacks.go:L97-105
			ctx.EventManager().EmitEvent(
				sdk.NewEvent(
					coinswaptypes.EventTypeSwap,
					sdk.NewAttribute(coinswaptypes.AttributeValueAmount, swappedAmount.String()),
					sdk.NewAttribute(coinswaptypes.AttributeValueSender, recipient.String()),
					sdk.NewAttribute(coinswaptypes.AttributeValueRecipient, recipient.String()),
					sdk.NewAttribute(coinswaptypes.AttributeValueIsBuyOrder, strconv.FormatBool(true)),
					sdk.NewAttribute(coinswaptypes.AttributeValueTokenPair, coinswaptypes.GetTokenPairByDenom(transferredCoin.Denom, swapCoins.Denom)),
				),
```

#### Reommended mitigation

Use `EmitTypedEvent`


### L-5 Silent fail on no whitelisting

There is no logging on the early return condition of no whitelisted channels found for the detaination channel.

Logging this return condition would provide a data point for observability, suporting system monitoring an issue diagnosis.

#### Reommended mitigation

```
x/onboarding/kepper/ibc_callbacks.go:L56
+logger.Error("no whitelist channel found for %s",packet.DestinationChannel)
```


### L-6 Silent fail on recipient account being a module

There is no logging on the early return condition when the recipient account is a module.

Logging this return condition would provide a data point for observability, suporting system monitoring an issue diagnosis.

#### Reommended mitigation

```
x/onboarding/kepper/ibc_callbacks.go:L71
+logger.Error("recipient account is a module %s",account.GetAddress())
```


### L-7 Silent fail on missing ERC20 mapping

There is no logging on the early return condition when no ERC20 mapping is found for the denom.

Logging this return condition would provide a data point for observability, suporting system monitoring an issue diagnosis.

#### Reommended mitigation

```
x/onboarding/kepper/ibc_callbacks.go:L120
+logger.Error("no ERC20 mapping found for %s",transferredCoin.Denom)
```

### L-8 Silent fail on trading pair disable

There is no logging on the early return condition when the trading pair is disabled.

Logging this return condition would provide a data point for observability, suporting system monitoring an issue diagnosis.

#### Reommended mitigation

```
x/onboarding/kepper/ibc_callbacks.go:L127
+logger.Error("disabled trading pair with id %s",pairID)
```



## Non-Critical

### NC-1 Dead GoDoc params for TradeExactInputForOutput

The GoDoc for `swap:TradeExactInputForOutput` contain two parameters are not in the method signature and there are no likely matching candidates, these lines should be deleted as they'll only serve to sow confusion.

swap.go L68-69
@param sender: address of the sender
@param receipt: address of the receiver

#### Reommended mitigation

```
x/coinswap/keeper/swap.go:L68-69

-@param sender: address of the sender
-@param receipt: address of the receiver
```

### NC-2 Dead GoDoc params for TradeInputForExactOutput

The GoDoc for `swap:TradeInputForExactOutput` contain two parameters are not in the method signature and there are no likely matching candidates, these lines should be deleted as they'll only serve to sow confusion.

swap.go L68-69
@param sender: address of the sender
@param receipt: address of the receiver

#### Reommended mitigation

```
x/coinswap/keeper/swap.go:L159-160

-@param sender : address of the sender
-@param receipt : address of the receiver
```
