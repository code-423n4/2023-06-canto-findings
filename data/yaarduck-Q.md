Users have no way to opt out of the swap/conversion. Users might want to have no `canto` tokens at all or keep the IBC vouchers, but they can't.
This is problematic for a few reasons - tax tracking, automation code that expects to receive a fixed amount on the other side and evading potential loss of funds as demonstrated by other submitted issues.

One possible way to add an opt out mechanism is to add that info to the `memo` field of the IBC packet.