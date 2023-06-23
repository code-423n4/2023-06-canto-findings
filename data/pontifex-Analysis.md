# Approach
I'm familiar with golang and well knew such types of blockchains like Hyperledger Fabric.
Before the contest started I had researched previous Canto audit reports where there were interactions between contracts and sdk modules.
On the first day of the contest I read documentation for the onboarding module and researched the codebase for the first time.
On the second day I read documentation about IBC and ICS-20. Also I spent some time thinking about problems which can happen during the onboarding process.
The third day I looked up the codebase again to confirm my thoughts about possible problems. 
I wish I had more time for the contest.
# Codebase
I imagine the onboarding module as a robot which modifies users' messages. When something goes wrong the message shouldn't be modified. The module only reads state. The state isn't modified instantly in the module functions but only after transactions will be executed and included in a block.
# Findings contextualizing
I have two findings. Both of them are in the scope of a gap between state reading and transactions executing.
 

### Time spent:
16 hours