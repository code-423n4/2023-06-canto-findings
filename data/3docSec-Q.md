- the swap limit `MaxSwapAmount` is easily circumvented: one could chain multiple `types.MsgSwapOrder` in the same transaction and achieve atomically the same result as swapping a larger amount with a single message

- x/onboarding/keeper/ibc_callbacks.go function OnRecvPacket:
  - the onboarding function moves on with ERC20 conversion even if there are no coins left (transferredCoin - swappedAmount = 0). This can lead to sending 0 coins to the erc20 module for swapping, which is inconsistent with the erc20 module expectations (the MsgConvertCoin.validateBasic() would fail on zero amounts)