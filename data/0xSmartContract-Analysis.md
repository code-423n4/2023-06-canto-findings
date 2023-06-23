# 🛠️ Analysis - Canto Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Competition analysis| What are similar projects? |
|i) |Security Approach of the Project | Audit approach of the Project |
|j) |Other recommendations | What is unique? How are the existing patterns used? |
|k) |New insights and learning from this audit | Things learned from the project |
|l) |Resources - Frameworks | Details of the sources, documents and frameworks used in the above analysis |
|m) |Time spent on analysis | Time detail of individual stages in my code review and analysis |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-06-canto

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-06-canto#tests)|Test and installation structure is simple, cleanly designed with Go|
|2|Architecture Review| [Canto](https://github.com/code-423n4/2023-06-canto#canto-audit-details)|Mechanism that automatically sells a certain amount of CANTO token to the user account during Bridge|
|3|Test Suits|[Tests](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/testutil)|In this section, the scope and content of the tests of the project are analyzed.|
|4|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-06-canto#scope)||
|5|Special focus on Areas of  Concern|[Areas of Concern-Swap](https://github.com/code-423n4/2023-06-canto#swap)|IBC transfers and swap function|

## b) Analysis of the code base

| Contract | SLOC | Purpose | Modules used |  
| ----------- | ----------- | ----------- | ----------- |
| [x/onboarding/keeper/ibc_callbacks.go](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/onboarding/keeper/ibc_callbacks.go) | 55 | Contains core logic | [`x/coinswap`](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/coinswap) |
| [x/onboarding/types/params.go](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/onboarding/types/params.go) | 47 | Contains params for onboarding module | [`x/coinswap`](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/coinswap) |
| [x/coinswap/keeper/pool.go](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/coinswap/keeper/pool.go) | 85 | Contains logic for dex pools |  |
| [x/coinswap/keeper/swap.go](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/coinswap/keeper/swap.go) | 102 | Contains logic for dex swaps |  |

### [ibc_callbacks.go](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/onboarding/keeper/ibc_callbacks.go)

At this stage, we read the codes from top to bottom and try to understand what purpose they serve;
I'm code reviewing the `ibc_callbacks.go` contract piece by piece;
In the code analysis, only `ibc_callbacks.go` is shown in detail, the codes in other scopes are examined during the code audit with the same technique.

```golang
func (k Keeper) OnRecvPacket(
	ctx sdk.Context,
	packet channeltypes.Packet,
	ack exported.Acknowledgement,
) exported.Acknowledgement {
	logger := k.Logger(ctx)
```

1. The function begins by initializing a logger variable using the `Logger` method of the `Keeper` object (`k`). The logger provides access to a specific SDK context's logging capabilities.

```golang
params := k.GetParams(ctx)
if !params.EnableOnboarding {
	return ack
}
```

2. The function retrieves the parameters using the `GetParams` method of the `Keeper` object. If the `EnableOnboarding` flag in the parameters is set to `false`, the function returns the original acknowledgement (`ack`) without further processing.

```golang
var found bool
for _, s := range params.WhitelistedChannels {
	if s == packet.DestinationChannel {
		found = true
	}
}

if !found {
	return ack
}
```

3. The function checks if the `DestinationChannel` of the received packet is present in the `WhitelistedChannels` list defined in the parameters. If the channel is not found in the whitelist, the function returns the original acknowledgement without further processing.

```golang
_, recipient, senderBech32, recipientBech32, err := ibc.GetTransferSenderRecipient(packet)
if err != nil {
	return channeltypes.NewErrorAcknowledgement(err.Error())
}
```

4. The function uses the `GetTransferSenderRecipient` function from the `ibc` package to extract the sender and recipient addresses from the packet. If an error occurs during the extraction, an error acknowledgement is created and returned.

```golang
account := k.accountKeeper.GetAccount(ctx, recipient)
if _, isModuleAccount := account.(authtypes.ModuleAccountI); isModuleAccount {
	return ack
}
```

5. The function retrieves the recipient's account using the `GetAccount` method of the `accountKeeper` object. If the account is determined to be a module account, the function returns the original acknowledgement without further processing.

```golang
standardDenom := k.coinswapKeeper.GetStandardDenom(ctx)
```

6. The function retrieves the standard denomination using the `GetStandardDenom` method of the `coinswapKeeper` object.

```golang
var data transfertypes.FungibleTokenPacketData
if err = transfertypes.ModuleCdc.UnmarshalJSON(packet.GetData(), &data); err != nil {
	err = errorsmod.Wrapf(types.ErrInvalidType, "cannot unmarshal ICS-20 transfer packet data")
	return channeltypes.NewErrorAcknowledgement(err.Error())
}
```

7. The function unmarshals the packet data into a `transfertypes.FungibleTokenPacketData` object. If an error occurs during unmarshaling, an error acknowledgement is created and returned.

```golang
transferredCoin := ibc.GetReceivedCoin(
	packet.SourcePort, packet.SourceChannel,
	packet.DestinationPort, packet.DestinationChannel,
	data.Denom, data.Amount,
)
```

8. The function uses the `GetReceivedCoin` function from the `ibc` package to construct a `transferredCoin` object representing the transferred coin based on the packet's source and destination information, as well as the denomination and amount from the packet data.

```golang
autoSwapThreshold := k.GetParams(ctx).AutoSwapThreshold
swapCoins := sdk.NewCoin(standardDenom, autoSwapThreshold)
standardCoinBalance := k

.bankKeeper.SpendableCoins(ctx, recipient).AmountOf(standardDenom)
swappedAmount := sdk.ZeroInt()

if standardCoinBalance.LT(autoSwapThreshold) {
	swappedAmount, err = k.coinswapKeeper.TradeInputForExactOutput(ctx, coinswaptypes.Input{Coin: transferredCoin, Address: recipient.String()}, coinswaptypes.Output{Coin: swapCoins, Address: recipient.String()})
	if err != nil {
		logger.Error("failed to swap coins", "error", err)
	} else {
		ctx.EventManager().EmitEvent(
			sdk.NewEvent(
				coinswaptypes.EventTypeSwap,
				sdk.NewAttribute(coinswaptypes.AttributeValueAmount, swappedAmount.String()),
				sdk.NewAttribute(coinswaptypes.AttributeValueSender, recipient.String()),
				sdk.NewAttribute(coinswaptypes.AttributeValueRecipient, recipient.String()),
				sdk.NewAttribute(coinswaptypes.AttributeValueIsBuyOrder, strconv.FormatBool(true)),
				sdk.NewAttribute(coinswaptypes.AttributeValueTokenPair, coinswaptypes.GetTokenPairByDenom(transferredCoin.Denom, swapCoins.Denom)),
			),
		)
	}
}
```

9. The function checks if the balance of the standard coin (defined by `standardDenom`) held by the recipient is less than the `autoSwapThreshold`. If it is, a coin swap operation is performed using the `TradeInputForExactOutput` method of the `coinswapKeeper` object. The swapped amount and any error encountered during the swap operation are logged, and an event is emitted.

```golang
pairID := k.erc20Keeper.GetTokenPairID(ctx, transferredCoin.Denom)
if len(pairID) == 0 {
	return ack
}

pair, _ := k.erc20Keeper.GetTokenPair(ctx, pairID)
if !pair.Enabled {
	return ack
}
```

10. The function retrieves the ERC20 token pair ID corresponding to the transferred coin's denomination using the `GetTokenPairID` method of the `erc20Keeper` object. If the pair ID is empty, the function returns the original acknowledgement. Otherwise, it checks if the token pair is enabled. If it's not enabled, the function also returns the original acknowledgement.

```golang
convertCoin := sdk.NewCoin(transferredCoin.Denom, transferredCoin.Amount.Sub(swappedAmount))
convertMsg := erc20types.NewMsgConvertCoin(convertCoin, common.BytesToAddress(recipient.Bytes()), recipient)
if _, err = k.erc20Keeper.ConvertCoin(sdk.WrapSDKContext(ctx), convertMsg); err != nil {
	logger.Error("failed to convert coins", "error", err)
	return ack
}
```

11. The function constructs an ERC20 conversion message (`convertMsg`) using the remaining amount of the transferred coin (`convertCoin`) and the recipient's address. It then calls the `ConvertCoin` method of the `erc20Keeper` object to convert the Cosmos Coin to an ERC20 token. Any error encountered during the conversion is logged, and if an error occurs, the original acknowledgement is returned.

```golang
logger.Info(
	"coinswap and erc20 conversion completed",
	"sender", senderBech32,
	"receiver", recipientBech32,
	"source-port", packet.SourcePort,
	"source-channel", packet.SourceChannel,
	"dest-port", packet.DestinationPort,
	"dest-channel", packet.DestinationChannel,
	"swap amount", swappedAmount,
	"convert amount", convertCoin.Amount,
)
```

12. The function logs an informational message indicating that the coinswap and ERC20 conversion have been completed. It includes details such as the sender

 and receiver addresses, source and destination ports and channels, swapped amount, and convert amount.

```golang
ctx.EventManager().EmitEvent(
	sdk.NewEvent(
		types.EventTypeOnboarding,
		sdk.NewAttribute(sdk.AttributeKeySender, senderBech32),
		sdk.NewAttribute(transfertypes.AttributeKeyReceiver, recipientBech32),
		sdk.NewAttribute(channeltypes.AttributeKeySrcChannel, packet.SourceChannel),
		sdk.NewAttribute(channeltypes.AttributeKeySrcPort, packet.SourcePort),
		sdk.NewAttribute(channeltypes.AttributeKeyDstPort, packet.DestinationPort),
		sdk.NewAttribute(channeltypes.AttributeKeyDstChannel, packet.DestinationChannel),
		sdk.NewAttribute(types.AttributeKeySwapAmount, swappedAmount.String()),
		sdk.NewAttribute(types.AttributeKeyConvertAmount, convertCoin.Amount.String()),
	),
)
```

13. The function emits an event using the SDK's event manager. The event contains attributes such as the event type (`EventTypeOnboarding`), sender and receiver addresses, source and destination ports and channels, swapped amount, and convert amount.

```golang
return ack
```

14. Finally, the function returns the original acknowledgement.


After reading the code, I extract the architectural structure with the infographic (ibc_callbacks.go);

<img width="1035" alt="image" src="https://github.com/code-423n4/2023-06-canto/assets/104318932/2f2c71ea-173e-4282-8925-8cdeea5195a2">



## c) Test analysis


```js

README.md:

  254: - What is the overall line coverage percentage provided by your tests?:  100 for business logic

```

Test coverage is generally of good quality and based on Unit Tests, more Integration Tests can be added


The following parts should be added to the fund.go test file;

Test Cases: You should define various test cases to cover different scenarios and extreme cases. For example, you can test fund an account with different amounts of tokens, test with multiple accounts and module accounts, and test error conditions such as insufficient balance or invalid addresses.

Installation and Disassembly: If your tests require a specific initial state or involve changing the blockchain state, you should set up the necessary prerequisites before running each one.
The test here is an out-of-scope test and is shared for information about the test approach.
```solidity
Canto/testutil/fund.go:
   1: package testutil
   2: 
   3: import (
   4: 	inflationtypes "github.com/Canto-Network/Canto/v6/x/inflation/types"
   5: 	sdk "github.com/cosmos/cosmos-sdk/types"
   6: 	bankkeeper "github.com/cosmos/cosmos-sdk/x/bank/keeper"
   7: )
   8: 
   9: // FundAccount is a utility function that funds an account by minting and
  10: // sending the coins to the address. This should be used for testing purposes
  11: // only!
  12: func FundAccount(bankKeeper bankkeeper.Keeper, ctx sdk.Context, addr sdk.AccAddress, amounts sdk.Coins) error {
  13: 	if err := bankKeeper.MintCoins(ctx, inflationtypes.ModuleName, amounts); err != nil {
  14: 		return err
  15: 	}
  16: 
  17: 	return bankKeeper.SendCoinsFromModuleToAccount(ctx, inflationtypes.ModuleName, addr, amounts)
  18: }
  19: 
  20: // FundModuleAccount is a utility function that funds a module account by
  21: // minting and sending the coins to the address. This should be used for testing
  22: // purposes only!
  23: func FundModuleAccount(bankKeeper bankkeeper.Keeper, ctx sdk.Context, recipientMod string, amounts sdk.Coins) error {
  24: 	if err := bankKeeper.MintCoins(ctx, inflationtypes.ModuleName, amounts); err != nil {
  25: 		return err
  26: 	}
  27: 
  28: 	return bankKeeper.SendCoinsFromModuleToModule(ctx, inflationtypes.ModuleName, recipientMod, amounts)
  29: }
```





## d) Architectural 

The audited design solves an important problem. You usually make the first entry to new blockchains like Canto by sending a token from another chain with a bridge. For example, you sent 100 USDC to Canto with a bridge, but we do not have a fee, that is, CANTO, to make any transaction with this 100 USDC, so it is a 100 USDC transaction. we can't, the only solution is to find CANTO token from another chain and send it again with bridge, which increases the cost, this problem is solved by the project team with the codes subject to the audit.

But without auditing this code, it is necessary to know the architecture of CANTO network in general, Cosmos SDKs and smart contract with go,Let's take a brief look at them;

Although auditors usually audit in Solidity language, Canto wrote their contracts in go language, if you are not familiar with smart contract programming with go language, below references and code repositories will make you familiar with blockchain programming with go language before auditing;

1. Cosmos SDK:
    - Cosmos SDK provides a framework that allows you to build your own blockchain applications. Smart contracts written in Go can be used in this project.
    - Related source: https://github.com/cosmos/cosmos-sdk

2. Canto Network:
    - Canto Network is a blockchain platform written in Go language. Smart contracts on Canto Network are also written in Go language.
    - Related source: https://github.com/Canto-Network/Canto

3. Terra:
    - Terra is a blockchain platform using the Go language. Smart contracts can also be written in the Go language in Terra.
    - Related source: https://github.com/terra-money/core

4. Binance Chain:
    - Binance Chain is a blockchain platform powered by the Binance exchange. On Binance Chain, smart contracts can be written in the Go language.
    - Related source: https://github.com/binance-chain

5. Polkadot:
    - Polkadot is a platform that enables the merging of different blockchains. While smart contracts are written in Rust in Polkadot, smart contracts written in Go can be used in projects running on Polkadot's Parachain feature.
    - Related source: https://github.com/paritytech/polkadot



It is important to know the operation of Cosmos SDKs in order to understand the architecture of the part of the project to be audited;
https://v1.cosmos.network/sdk


What is Cosmos SDK?

“Cosmos itself is not a blockchain. It is a network of blockchains built to be compatible with each other.”

Called the "internet of blockchains" by the developers, Cosmos is a decentralized ecosystem that enables connections between blockchain systems. The goal of the project is to enable separate blockchains to communicate with each other seamlessly.

Cosmos offers a different perspective on the Layer-1 blockchain, enabling any blockchain to communicate with others, share data and perform transactions. It offers developers a next-generation technology that improves blockchain creation efficiency and gives them access to powerful tools. Developers can only use some parts offered by the Cosmos SDK. Because it is modular, developers can create different combinations to meet various demands. Blockchains like Bitcoin, Ethereum, and Solana represent a single/main blockchain network, while Cosmos represents a blockchain universe.

For `swap()` function , project use a forked version of IRISNET's Coinswap module v1.6, which includes some modifications.
During the audit, v1.6 should be examined architecturally, and especially the modified parts should be audited in detail.
https://github.com/irisnet/irismod/tree/v1.6.0/modules/coinswap

## e) Documents 
The document related to the code and the project to be audited can be increased further, and the operation of the codes can be added to the documents with an infographic, this will facilitate readability and the auditors' understanding.

https://github.com/code-423n4/2023-06-canto#overview-of-onboarding-middleware


##  f) Centralization risks 
No risk of centralization


##  g) Systemic risks 

The fact that the Canto blockchain network is relatively new and is naturally less war-tested is the most important system risk in the project, this is natural in such innovative projects and projects should be built with this risk in mind.


What is Canto?
Canto is a permissionless Layer 1 blockchain compatible with the Ethereum Virtual Machine (EVM). It’s designed to deliver on the decentralized finance (DeFi) promise – that new models will be easily accessible, completely transparent, decentralized, and free via a post-traditional financial drive. Canto launched with no core foundation, presale, vesting, or venture backers to be completely decentralized. 


At its core, Canto leverages the Tendermint Consensus and the Cosmos Software Development Kit (SDK) and is secured by Canto validators. It achieves EVM compatibility through the Ethermint system, which enables an EVM execution environment, further facilitating the deployment of Ethereum smart contracts. These tools include the Canto Decentralized Exchange (DEX), Canto Lending Market (CLM), and NOTE. Canto aims to become the “best execution layer for original work” through the execution of:

Zero fees for liquidity providers (LPs) – protocol users, traders, and arbitrageurs can access liquidity freely. 

Rent extraction resistance – Canto believes primary decentralized applications (dApps) like DEXs and lending protocols should have no governance tokens to offer FPI.  

Minimally reasonable customer capture – Primary public primitives lack customer interfaces to enable customer acquisition for Canto protocols. 

The Team Behind Canto
Scott Lewis, co-founder of DeFi Pulse and Slingshot Crypto is a contributor and leading force in Canto. The Plex team is another group of contributors, responsible for the development of the network, its Free Public Infrastructure, and the canto.io frontend.  A DAO proposal was also recently passed to bring on the B-Harvest team, an established team within the Cosmos ecosystem, as core developers for Canto.

Another notable contributor is NeoBase, who steward the Canto EVM blockchain explorer, and have developed a Canto analytics dashboard featuring details of the protocol including the lending market, as well as covering the Canto DEX and Forteswap.




## h) Competition analysis

There has been no previous project that automatically sells native tokens as much as the fee fee during Bridge.





## i) Security Approach of the Project



Successful current security understanding of the project;

1-They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.



What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Secondly audit from a reputable auditing organization and resolved all the security concerns in the report






## j) Other recommendations


An important detail in the design: Checking the existence of min 4 Canto in the account. The amount here should not be hardcode, it would be more advantageous to design it in a more flexible structure, in a variable structure, taking into account the salary base that may increase in the future.








## k) New insights and learning from this audit 

The most important learning I got from this project; My understanding of the workings of a new blockchain like Canto and the design of integration with the Cosmos SDK

In addition, I understood how new blockchain projects can become user-friendly with an architecture, especially if there is no gas fee in the tokens that come to them with the bridge.

## l) Resources - Frameworks

The resources and frameworks I use while doing code audit and analysis;

Ide: VsCode and VsCode Go Extension
Document: https://go.dev/
Document: https://github.com/cosmos/cosmos-sdk
Document: https://github.com/irisnet/irismod/tree/v1.6.0/modules/coinswap
Document: https://v1.cosmos.network/sdk
https://www.loom.com/share/632db169981740b2b149411c9249def5?sid=75e7de98-e5ab-4e17-a67d-dfe642f15b78




## m) Time spent on analysis

A total of 11 hours was spent

### Time spent:
11 hours