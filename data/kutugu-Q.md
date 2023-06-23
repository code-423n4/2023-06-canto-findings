# Findings Summary

| ID     | Title                                                        | Severity |
| ------ | ------------------------------------------------------------ | -------- |
| [L-01] | Should check port type before swap Canto                     | Low      |
| [L-02] | MaximumSwapAmount may be insufficient When Canto price rises | Low      |

# [L-01] Should check port type before swap Canto

## Description

According to the docs:
> When users transfer assets to the Canto network through Gravity Bridge, the IBC transfer automatically triggers swap and conversion to Canto via IBC middleware. These actions are triggered only when transferred through a whitelisted channel.

Canto swap depends on IBC permissionless callbacks, it is necessary to carefully check whether the channel is trusted.   
Canto has whitelist channels check, which is handled well. However, the port type is not checked.   
If the IBC message is not a transfer port, the swap logic may be mistakenly executed.   

## Recommendations

Check port is transfer type

# [L-02] MaximumSwapAmount may be insufficient When Canto price rises

## Description

According to the docs:
> For risk management purposes, a swap will fail if the input coin amount exceeds a pre-defined limit (10 USDC, 10 USDT, 0.01 ETH) or if the swap amount limit is not defined.

When the price of Canto tokens increase huge, the limit amount of assets may not be fully exchanged for one Canto token, causing middleware functionality to be unavailable.   

## Recommendations

Add a function to modify MaximumSwapAmount