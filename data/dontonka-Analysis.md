## Background
I'm new to Cosmos, not English native (so sorry for my imperfect English) but not totally either since I audited few hours on `pStake on Immunify` few weeks ago, but still beginner XD. On the other hand, I'm not bad at Golang and use this language in my day job as a Sr. Software Engineer.

## Approach taken in evaluating the codebase
1) I started at reading the whole README to get an initial idea.
2) Then I checked the 5 min. video explaining the current contest, which was nice to have.
3) Then I stepped into the code, opening the 4 files in scope, and started with the smaller file (x/onboarding/types/params.go). I quickly found many weakness for which I submitted a `Medium` issue.
4) Then I stepped into the main file (x/onboarding/keeper/ibc_callbacks.go) to get an overview again on the business logic.
5) Then I stepped into the swap logic (x/coinswap/keeper/pool.go and x/coinswap/keeper/swap.go), also cloned Coinswap (and checkout at tag v1.6.0), and compared with the code here, to see the changes the team mades. pool.go had nothing besides import changes, while swap.go had some core changes, which I have evaluated and `didn't spot anything bad`.
6) Then I read again the documentation from C4 (mainly for the Swap logic), and spotted that the `DefaultMaxSwapAmount for ETH` was wrong in the code (0.1 ETH instead of 0.01 ETH), so even if that specific file was not in scope I decided to submit a `Medium` for this since it could potentially be exploited.
7) Then I stepped again on the main file (x/onboarding/keeper/ibc_callbacks.go) trying to understand in details every possibility, also how it works with the other middlewares. I spotted that the recovery middleware is doing some checks that the onboarding was totally ignoring (sender == recipient, account exist, account's pubKey supported by canto). I started to dig into those and discussed with the sponsor in private (t_k__) to understand the expected result with those, to figure out at least one issue, for which I submitted a `High`.
8) During all those evaluations, I was always adding, modifying and running unit test to understand more the code and try to break things, good thing with golang is that you can step-by-step debug, so its much easier than EVM smart contract debugging.

## Anything that you learned from evaluating the codebase
I learn a bit more about the Cosmos ecosystem, IBC and discovered this (https://hub.mintscan.io/chains/ibc-network) which is pretty amazing tools. I think I need to invest more into the Cosmos ecosystem both financially and in terms of skills set.

## Any comments for the judge to contextualize your findings
The team used 2 testing framework, the usual standard testing framework for unit test and Ginkgo for integration test. I personally don't like Ginkgo, its also not well integrated in IDE to run a specific test, but you can always limit by changing to `FIt` instead of `It` for the specific test you want to run. Having 2 test framework make it harder to maintain in general, but maybe ginko give more flexibility for integration test, to be honest I'm not sure. Finally, the unit test in (`x/onboarding/keeper/ibc_callbacks_test.go`) are `very error-prone` as tests are relying on globals variables but since tests execute sequencially, they `are dependent on the previous` test values, so to understand the test case setup, you need to look at the previous tests setup, it's very messy and easy todo a mistake in such scenario, also much harder to maintain. For example, if you would change one test case and and move it in the list, most likely you would be breaking it as the setup might not be correct anymore. `I would highly recommend the team to change that`, so that each unit test setup express clearly their needs and doesn't depends on previous tests.

Another aspect that was new to me and I didn't like is the `fact of mixing receiver types` (pointer and value) for a specific struct, an example was in params.go
func (p *Params) ParamSetPairs()
func (p Params) Validate()

In my experience, `it's easier to use a type that has either pointer or value receivers`. When a type has mixed pointer and value receivers, the types that use it may need to be pointer or value receivers depending on the methods they use. This introduces a problem - if in the course of normal software engineering, you need to use a pointer receiver method, then the method you're using it in might also need to change from a value receiver to a pointer receiver. This introduces an unpredictable cascading effect when maintaining types. If the type has consistently used pointer or value receivers, then another type can use it and the receiver types of it's methods won't need to vary depending on which methods it's members use - it breaks the cascading effect. Overall, keeping receiver types consistent makes your code a lot easier to reason about from a user's perspective and that's a really good reason.

Besides, the production code seems well written to me.

### Time spent:
6 hours