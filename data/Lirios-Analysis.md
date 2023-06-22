For me this was the first audit of a cosmos/IBC/GoLang project.
I have experience in Solidity and in the past have looked at the geth code to try to better understand EVM internals. Also have looked at IBC before out of interest, but never really in depth.

The documentation of the project is good and understandable. Very nice to have specs folder per module with specs and info in code.
Big part of time was spent in better understanding GoLang and reading documentation which was very valuable.
After doing that, only had time for manual code review and did not setup a test environment to do real testing, assuming that would take too much time.

Scope of the audit is small with only 4 files in scope. While small is good for a first audit and keeps the focus on the part to audit, it would be nice if there was a "onboarding uses [erc20](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/erc20) and [coinswap](https://github.com/code-423n4/2023-06-canto/tree/main/Canto/x/coinswap) modules. Any issues found in those modules that influence onboarding will be considered as in scope" to better audit the complete onboarding process.

Overall the project looks solid



### Time spent:
8 hours