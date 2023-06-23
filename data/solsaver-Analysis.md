# Analysis Report
The codebase to audit was relatively shorter than other audits, but as everything was in Go programming language, it was a bit of unchartered territory in terms of auditing. I have basic understand of Go programming language, but I still havent adapted to using Go for blockchain contracts and to identify all the possible flaws. With that said, below is my analysis and the effort that I put in.

## Audit Process
* I went through the Code4rena audit page first, as there was good enough details in there.
* Then I watched the explainer video that was posted in the Discord channel.
* After that, I went through method by method, verifying the assumptions based on the comments. I also looked at the usage of methods, to understand anything about the underlying usage, specially via the unit tests.

## Quality of the Codebase
* The codebase was well organized, had good variable names, comments and documentation references where ever applicable. That did make the audit process better.
* The methods were short, and a consistent pattern was followed on how to handle errors, define methods, and perform various checks.

## Others
* I could not find any centralization risks in the codebase that had to be audited.
* I will highly recommend to add unit tests for each file, as it makes testing much easier for any attack scenarios.
* This may not be the best recommendation, but please consider using Solidity contracts so that the codebased can be analyzed by seasoned wardens who might be able to find issues through Solidity. Its very easy to miss bugs in unfamiliar programming languages. But that said, the wardens (including me) need to up their games to be proficient in non-Solidity contracts.


### Time spent:
8 hours