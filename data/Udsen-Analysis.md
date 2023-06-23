1. Any comments for the judge to contextualize your findings

My findings are mainly based on the `IBC input token` swap for `4 Canto` tokens. This is conducted to fulfill the `autoSwapThreshold` requirement. I recommend that it is not ideal to keep a hardcoded threshold. The swap amount of `Canto` tokens should be based on the canto threshold and recipient's balance difference. Because more tokens are converted to `canto` even if the recipient's existing balance is closer to the `autoSwapThreshold` threshold.
I believe this will have more impact as the `Canto` price increase in the future. If Canto price goes to 10 USD then 4 Canto = 40 USD. Which could be considerable amount swapped in the perspective of the recipient irrespective of his `Canto` balance (if the balance is less than 4 Canto).

And further I have given another finding related to the `packet.SourceChain`. In my opinion the `WhitelistedChannels` list should be checked against the `SourceChain` of the channel. But in the `ibc_callbacks.OnRecvPacket` function the `DestinationChain` is used instead. This could be problematic since now the execution is independent of the `SourceChain`.

2. Approach taken in evaluating the codebase

I initially started auditing from the codebase with the most SLOC which is the `swap.go`. Then moved into other files in the descending order of the SLOC. After the initial audit, I drew relationships among the 4 code files and their logics to derive possible attack vectors. 

Ones I understood the underlying system better I focused more on the `ibc_callbacks` file, since it included the core logic of the protocol.

3. Architecture recommendations

The hardcoding of the `autoSwapThreshold` to be swapped, should be removed and more flexibility should be given to the recipient in this regard. Even though `4 Canto` results in a small USD amount as of now, it might not be the case in the future. The swap amount should be considered as the difference between the recipient's balance and the `autoSwapThreshold` amount.

4. Codebase quality analysis

Codebase was comprehensively written with proper commenting to understand the logic behind the implementation. Since the codebase is written in `Golang` it is challenging to understand compared to `solidity`. 

The code base is clearly divided into main components of the protocol logic. Separate code files are included for `swap`, `pool`, `ibc_callback` and `params` which are the major components of the protocol. This provides a greater understanding and readability of the protocol to the auditor.  

There were few Typos in the comments of the code which can be corrected for better readability.

5. Centralization risks

The Centralization risk is associated with the following areas:

Whitelisting of `Source Chain's` of the Channels
Whitelisting of the `Canto` liquidity pools for the swap.
Deciding on the Canto token limit for the liquidity pool (Currently 10,000 Canto tokens).
Deciding on the maximum input amount of the `USDC`, `USDT` and `ETH` as the IBC transfer token amounts.

The admins have control over above decisions which could impact the users of the protocol in the future.

6. Mechanism review

Since Canto is a Layer 1 blcokchain which is built using the cosmos-SDK and uses an EVM for execution, it is a highly versatile protocol. The protocol uses tendermint consensus, the attack vectors such as front-running are limited. Because the transactions are not processed strictly in order of gas price. 

The protocol uses `IRISNET's Coinswap module v1.6` as the AMM liquidity pool for the swaps which is similar in nature to the Uniswap liquidity pools in ethereum. The protocol is providing an efficient method to onboard non-Canto users to the Canto network via this middleware layer. 

Hence the attack vectors considered for this protocol is slightly different from other solidity based decentralized applications.

7. Systemic risks

Since the system uses liquidity pools similar to the ones used in Uniswap, there could be risks associated with manipulation of these pools by high networth individuals. 

Since the swaps and conversions of tokens are dependent on these pool reserves, manipulation could effect the entire swap and conversion logic of the protocol thus putting the users at risk of fund loss, in extreme cases

Further more when this protocol expands and more liquidity pools are added with different token pairs, it is recommended to consider the decimal precision as well while price and token amount calculations in the `swap.go` file. Else this could lead to accounting errors which could hamper the proper system execution.

### Time spent:
9 hours