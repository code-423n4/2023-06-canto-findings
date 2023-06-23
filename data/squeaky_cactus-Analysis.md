**Any comments for the judge to contextualize your findings**
This was my first experience with IBC code, having previously solely dealt with Ethereum. There were a few gotcha, including the usage of the term denom to mean the label for coins, rather than the decimal places.

**Approach taken in evaluating the codebase**
Manual review, starting with a bottom approach, then after reviewing the video and various design docs a top down.

Tools used included GoLand IDE and Qodana.

**Architecture recommendations**
For the desired use case, the architecture/design is perfectly fine. 
Chaining middleware modules together is a common approach, and while it can mean either assumptions get made (e.g. with regard to input validation) or a duplication of processing can occur, but when viewed in it's entirety as whole system it'll operate as expected.

There's good reuse of existing libraries, such as the Cosmos types that provide an excellent safety net in terms of meth operations.

**Codebase quality analysis**
The code is functional, with a battery of test covering it.

The codebase has a mix of receiver usage (pointers & value), which could be in most cases (if not all, maybe due to interfaces) be converted to pointer usage.

The defensive coding approach taken (early returns in functions/methods), is better used when you perform the precondition testing before side effecting. 

There are some GoDoc comment, however they're applied inconsistently and those at the method level, rarely convey more then can be inferred from the function/method name and signature.

**Centralization risks**
The code will need to run on nodes, which is the point of centralization. 

**Mechanism review**
Simple mechanism of checking if the sender has a configurable amount of acanto (default 4 acanto), if not then a swap is performed to get them that amount (failing silently).

**Systemic risks**
Configuration/initialization risk, any problems are likely to arise from misconfiguration of the settings & pools or altering of their state.




### Time spent:
12 hours