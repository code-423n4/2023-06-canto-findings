## Approach to evaluating the codebase
I use my familiar IDE (vscode) to evaluating the codebase, which allows me to quickly see the overall project architecture, easily jump between function calls and implementations, quickly find the code modules I want to analyse in detail, etc.
I use a top-down, coarse-to-fine approach to my evaluation, first getting a sense of the overall project architecture and then diving into the details for a more detailed evaluation.

## Centralisation risks
The whitelists, thresholds and other parameters (e.g. [onboarding params](https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/types/params.go#L12-L16)) are currently set by the team that controls the code (the official canto team), which is actually related to the security of users' money, and it would be better if they could be set in a more decentralised way.

## Codebase Quality Analysis
I think the overall architecture and code quality of the project is quite good, the architecture is reasonable, the logic is clear, the code is clean, and the comments are clear.

And I think it could be more detailed for the docs.
After all, this project is very different from other contests on c4, its underlying architecture, cosmos ibc mechanism, etc. may be unfamiliar to many wardens.
Therefore, it maybe better to provid more technical introduction (or more provide related links) in the docs, to help the wardens start up more quickly and easily, and it will make the contents more effective.

## What did you learn from the codebase evaluation?
This is a Go-based project, which is very different from most of the other projects on c4 (which are mostly Solidity language implementations).
And canto is a cosmos-based blockchain, and its architecture is very different from ethereum.

Although I have some prior knowledge of go, cosmos, and the canto project, it do reinforced my knowledge by auditing this codebase, especially cosmos-sdk architecture.

## Advice for C4
I think C4 should have more of these differentiated competitions.
We can choose to collaborate with more different blockchain projects, and we can also choose more projects that involve different technical fields and programming languages.
Because blockchain is an open and rich ecology, but security is always very important, any project, any technology, any implementation needs a secure audit to guarantee, and it needs a decentralised audit like C4 to strengthen.

At the same time, c4's decentralised community needs more types of projects to attract more diverse technical people and grow the community. Of course, a variety of competitions will also allow members to learn more and gain more experience.


### Time spent:
6 hours