After auditing the new `onboarding` module, together with the modified `coinswap` module, I believe the code is in good shape both architecturally and security-wise.
A few potential issues were found, one that I considered high, but the rest are easily fixable and their impact are low (as long as the codebase doesn't change).

The onboarding feature seems very needed for onboarding new users and will be very helpful in making the first transactions in the Canto network.

I analyzed the code mostly in two ways - manual auditing the code and via tweaking existing tests. It's very noticeable that the developers put a lot of effort in the rich test suite. It helped writing the POCs and the code exploration much easier. Most edge cases were caught by the existing test suite.

While not fully reviewing it, the `recovery` module seems to contradict with the new `onboarding` module - as new users might be "recovered" before being onboarded. Because currently the recovery module is disabled, it has no security risks.

I would like to raise two concerns in the architecture, one of them raised as a low/QA finding:
1. In my opinion, users should be able to opt out of the recovery mechanism. Some applications might assume that the entirety of the funds (either in vouchers or ERC20 format) will be kept. It means that automation code might find it difficult for integration. Opting out should be easy to implement. Though, as long as the only integration of the onboarding module is with the Gravity Bridge chain, this is probably impossible.
2. The use of a dedicated and small AMM for the onboarding swaps might lead to loss of funds to users. While it's limited to a loss of +-$10 per user, this can sum up if the chain will onboard many users. IMO first converting the vouchers to ERC20 format and then using one of the available DEXs on chain would have better UX, as those DEXes will be bigger and steadier.

### Time spent:
20 hours