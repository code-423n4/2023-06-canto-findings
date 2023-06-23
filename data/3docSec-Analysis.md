This is my first contest in C4. However I have more experience coming from Immunefi where I was awarded a handful of reports, including my first High severity this very week.

About my findings:

- I am not completely sure about the severity of the first report (the off-by-one mantissa). It may be low (“software not working as expected”) as well as higher because, as mentioned, it’s an incorrect limit on the trading volume. Even though, as mentioned in the QA report, it can be easily avoided

- the second “high” report may actually be two separate issues: “LP limits can be circumvented by simply sending tokens to the pool” and “the first LP liquidity provider can claim exclusive ownership by exploiting the deposit limits & sending tokens”. I reported them together because when used together they can maximize damage

- one thing I did not say about maximizing damage and limiting risk: the above mentioned combined attack can be best executed by keeping permanent exclusive ownership of the pool, and only temporarily manipulating the pool swap price when front-running transactions coming from the gravity bridge. These are executed after being confirmed on 2 chains, so should be VERY easy to front-run. Making the price manipulation temporary prevents anybody to profit from the “favorable” side of the swap while the price is off

Please do let me know if there’s anything I can improve on. Thank you

### Time spent:
10 hours