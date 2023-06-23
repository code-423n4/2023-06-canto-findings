# Introduction

On Canto network, the x/onboarding module implements IBC middleware for users who don’t have native tokens for initial gas spending. Additionally, this module will also convert IBC assets to their ERC20 equivalent if the mapping is already registered using the x/coinswap module.

# Protocol Flow

1. Firstly set params and create three lp pools for USDC, USDT and ETH
2. User transfers assets to the Canto network through Gravity Bridge
3. Check whitelisted channel && recipient type && recipient Canto balance
4. If the balance is less than AutoSwapThreshold, swap the assets to Canto using the coinswap module
5. If the remaining assets are registered as a ERC20, convert them to ERC20
6. Errors that occur during onboarding are only logged and transactions are not revert; Assets remain on the Canto network with the recipient address

# Observations

## Architecture

The core swap logic forks [IRISNET's Coinswap module v1.6](https://github.com/irisnet/irismod/tree/v1.6.0/modules/coinswap), is the implementation of uniswapv2's AMM invariants.

## Centralization risks

None

## Systemic risks

System depends on IBC permissionless callbacks, it is necessary to carefully check whether the channel is trusted. Canto has whitelist channels check, which is handled well.   
However, the port type is not checked. If the IBC message is not a transfer port, the swap logic may be mistakenly executed.

## Mechanism review

### Expiration check

One of the key guarantees provided by core IBC is that a packet will not be received on the destination chain if the timeout has passed. Timeouts are specified within the packet interface by a timeoutHeight and timoutTimestamp of the counterparty (receiving) chain. So callbacks don't need to check if messages timeout, by the way, callback has slippage protection.

### AutoSwapThreshold

Amount of the swapped Canto is always equal to the AutoSwapThreshold, if user's assets is less than the AutoSwapThreshold, the swap logic fails.
Currently there is not much impact based on the value of Canto token, but if the price of Canto tokens rises, it is better to support the exchange of small funds.  

### MaximumSwapAmount

For risk management purposes, the input coin amount has a pre-defined limit.    
When the price of Canto tokens increase huge, the limit amount of assets may not be fully exchanged for one
Canto token, causing middleware functionality to be unavailable.   

### Uniswap k

The processing logic in swap is to calculate the input and output, without considering the check k value, which may affect the balance of the pool:
- Considering the boundary value, the division method has precision error, input amount needs add 1, which is handled well
- Considering the fee-on-transfer tokens, the number of tokens transferred to the pool is lower than expected, resulting in a decrease in k value and affecting the pool balance.  

### Revert

OnRecvPacket consists of two logical: `swap Canto` and `convert ERC20`, which are executed in sequence. Need ensure that the `swap Canto` failure doesn't affect subsequent `convert ERC20` execution.
So you need to make sure that errors are handled through the log and don't revert subsequent logic, where has a special `Sub` calculation.
When an overflow occurs, instead of returning an error, Sub will revert directly, so you need to make sure that `Sub` overflow doesn't occur.  
Especially `outputReserve.Sub(outputAmt)`, through `exactBoughtCoin.Amount.GTE(outputReserve)` short-circuit return to deal with, which is handled well.   


### Time spent:
12 hours