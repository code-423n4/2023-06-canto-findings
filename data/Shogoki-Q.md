# QA Report Canto Protocol

## Non Critical 

### [NC-01] Unneccessary double call to getParams

#### Description 

In [ibc_callbacks.go:87](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L87) there is an additional call to `k.GetParams`. This is unnecessary, because all params are already in memory inside the local `params` variable from the call to `GetParams` in [Line 39](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L39)

#### Recommendation

Replace the call to `GetParams` with an access to the existing variable `params` 

### [NC-02] Too broad scope for variable swapCoins / unnecessary cal

#### Description

In [ibc_callbacks.go:88](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L88) the variable `swapCoins` gets declared and initialized with a call to `sdk.NewCoin`. However, this variable is only needed and used when the Canto balance is below the swapThreshold. Therefore it is a waste of compute resources and memory to declare it outside the corresponding if block scope [Line 92 - 108](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L92-L108). 

#### Recommendation

Move the declaration and initialization of `swapCoins` inside the if Block.

### [NC-03] Comments are not describing the actual actions of the code

#### Description

On multiple places inside the code base there are comments describing the function or a specific line of code. However in some places the description diverges from what the code actually does.
This is the case in the following places:

- [pool.go:30](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/coinswap/keeper/pool.go#L30-L31) The comment tells, that this function `return the liquidity pool by the specified anotherCoinDenom`, but this is not the case as it is returning the pool based on the poolId

- [pool.go:69](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/coinswap/keeper/pool.go#L69-L70) 
The comment here tells the same, `return the liquidity pool by the specified anotherCoinDenom`, but it is indeed returning the pool balances based on a given escrow address

- [swap.go:77-79](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/coinswap/keeper/swap.go#L77-L79) 
The comment says to check for the amount is `more than` the minimum. However, the code is checkinf for it to be `more than or equal` 


- [swap.go:169-171](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/coinswap/keeper/swap.go#L169-L171) 
The comment says to check for the amount is `less than` the max. However, the code is checking for it to be `less than or equal` 

## Low

### [L-01] unneccessary Loop iterations

#### Description

In [ibc_callbacks.go:45-50](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/keeper/ibc_callbacks.go#L45-L50) there is a loop, checking if the destinationChannel is in the whitelist. For this it is looping through all Whitelisted channels, and if it matches setting the `found` variable to true. However, if there was a match, the code still loops through the rest of all the whitelisted channels. Therefore, if the list of whitelisted channels is rather long, it can lead to a lot of unneccessary iterations.

#### Recommendation

If the Channel was found, there should be an early breakout of the loop using the `break` keyword.

