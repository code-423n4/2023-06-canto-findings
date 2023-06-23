I'm beginner GOlang learner. So, first I try to read and understand of the project and the project files. Sadly, I try to investigate the codebase more but I have a day job and need to code some new features.

First, I didn't have any prior knowledge on IBC or Gravity Bridge. I still trying to figure somethings out. My main concerns were about OnRecvPacket and Middleware ordering. 

It wasn't clear to me where all the funds go when any step fails because it says: 'onboarding process is non-atomic,' and it also says: 'it does not revert IBC transfer and the asset transferred to the Canto network will still remain in the Canto network.'

So, I thought all failed transfers can't be reverted. 

And there are some rules on boarding to happen like: 10 USDT, 10 USDC and 0.01 ETH limit. It could also be go wrong if Canto prices goes up etc. 

### Time spent:
3 hours