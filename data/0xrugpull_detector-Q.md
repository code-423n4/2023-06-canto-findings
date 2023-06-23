If transferredCoin is all swapped and nothing left,there's no need to convert coin.

https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/onboarding/keeper/ibc_callbacks.go#L124-#L136
```
	convertCoin := sdk.NewCoin(transferredCoin.Denom, transferredCoin.Amount.Sub(swappedAmount))

+	if convertCoin.Amount.IsPositive() {
		// Build MsgConvertCoin, from recipient to recipient since IBC transfer already occurred
		convertMsg := erc20types.NewMsgConvertCoin(convertCoin, common.BytesToAddress(recipient.Bytes()), recipient)


		// NOTE: we don't use ValidateBasic the msg since we've already validated
		// the ICS20 packet data

		// Use MsgConvertCoin to convert the Cosmos Coin to an ERC20
		if _, err = k.erc20Keeper.ConvertCoin(sdk.WrapSDKContext(ctx), convertMsg); err != nil {
			logger.Error("failed to convert coins", "error", err)
			return ack
		}
+	}
```