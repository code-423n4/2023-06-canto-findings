Should check that the WhitelistedChannels field of Params of the onboarding module has valid value

```
// Validate checks that the fields have valid values
func (p Params) Validate() error {
	if err := validateBool(p.EnableOnboarding); err != nil {
		return err
	}
	if err := validateAutoSwapThreshold(p.AutoSwapThreshold); err != nil {
		return err
	}
	return nil
}
```
[Permalink][Permalink]

## Recommended Mitigation Steps

The team might consider adding validateWhitelistedChannels() in the Validate() method at x/onboarding/types/params.go


[Permalink]: https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/onboarding/types/params.go#L88C1-L97C2