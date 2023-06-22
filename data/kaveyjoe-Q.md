Target : https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/onboarding/keeper/ibc_callbacks.go

1 . In the line transferredCoin := ibc.GetReceivedCoin(...), the ibc.GetReceivedCoin function is used to construct the transferredCoin object. However, the function arguments passed to ibc.GetReceivedCoin seem incorrect. The function expects the following arguments: (sourcePort, sourceChannel, destinationPort, destinationChannel, denomination, amount). However, in the code, the arguments are passed in the order: (packet.SourcePort, packet.SourceChannel, packet.DestinationPort, packet.DestinationChannel, data.Denom, data.Amount). This mismatch in argument order can lead to incorrect values being assigned to transferredCoin.

2 . In the subsequent code block, the standardCoinBalance is checked against autoSwapThreshold using the LessThan function (standardCoinBalance.LT(autoSwapThreshold)). However, the LessThan function compares the values strictly, excluding the case where the values are equal. If the standardCoinBalance is equal to autoSwapThreshold, the swapping logic will be incorrectly executed, leading to unexpected behavior.

3 . The line swappedAmount, err = k.coinswapKeeper.TradeInputForExactOutput(...) attempts to swap the coins using the coinswapKeeper. If the swapping operation fails, the error is logged but the execution continues without returning an acknowledgement with an error. This behavior may lead to inconsistent state if the swapping operation encounters an error.

4 . The code block for converting coins to ERC20 tokens seems to be missing a check for the balance of the standardDenom before performing the conversion. It is important to ensure that the balance is sufficient for the conversion to avoid errors or inconsistencies.



TARGET : https://github.com/code-423n4/2023-06-canto/blob/main/Canto/x/coinswap/keeper/pool.go

1 . In the CreatePool function, there is no error handling for the GetStandardDenom function call. If GetStandardDenom returns an error, it should be handled appropriately.

2 . In the GetPoolByLptDenom function, there is no error handling for the GetPool function call. If GetPool returns an error, it should be handled appropriately.

3 . In the GetPoolBalances function, if the account retrieved using k.ak.GetAccount is nil, it returns nil instead of coins, which may cause unexpected behavior. It should return coins instead of nil and handle the ErrReservePoolNotExists error separately.

4 . In the GetPoolBalancesByLptDenom function, if the account retrieved using k.ak.GetAccount is nil, it returns nil instead of coins, which may cause unexpected behavior. It should return coins instead of nil and handle the ErrReservePoolNotExists error separately.

5 . In the GetLptDenomFromDenoms function, if denom1 and denom2 are equal, it returns an error of type types.ErrEqualDenom. However, this error is not handled in the calling code. It should be handled appropriately.

6 . In the GetLptDenomFromDenoms function, if both denom1 and denom2 are not equal to the standard denom, it returns an error of type types.ErrNotContainStandardDenom. However, this error is not handled in the calling code. It should be handled appropriately.

7 . In the GetLptDenomFromDenoms function, if the pool retrieved using k.GetPool does not exist, it returns an error of type types.ErrReservePoolNotExists. However, this error is not handled in the calling code. It should be handled appropriately.

8 . In the ValidatePool function, if the pool retrieved using k.GetPoolByLptDenom does not exist, it returns an error of type types.ErrReservePoolNotExists. However, this error is not handled in the calling code. It should be handled appropriately.

9 . In the setPool function, there is no error handling for the store.Set function calls. If store.Set returns an error, it should be handled appropriately.

10 . In the getSequence function, there is no error handling for the store.Get function call. If store.Get returns an error, it should be handled appropriately.

11 . In the setSequence function, there is no error handling for the store.Set function call. If store.Set returns an error, it should be handled appropriately.

