Ensure that all functions consistently handle errors by returning the error values to the calling functions
https://github.com/code-423n4/2023-06-canto/blob/a4ff2fd2e67e77e36528fad99f9d88149a5e8532/Canto/x/coinswap/keeper/pool.go#L43-L54
// GetAllPools return all the liquidity pools
func (k Keeper) GetAllPools(ctx sdk.Context) (pools []types.Pool) {
        store := ctx.KVStore(k.storeKey)
        iterator := sdk.KVStorePrefixIterator(store, []byte(types.KeyPool))
        defer iterator.Close()
        for ; iterator.Valid(); iterator.Next() {
                var pool types.Pool
                k.cdc.MustUnmarshal(iterator.Value(), &pool)
                pools = append(pools, pool)
        }
        return
}
Recommendation 
func (k Keeper) GetAllPools(ctx sdk.Context) ([]types.Pool, error) {
	store := ctx.KVStore(k.storeKey)
	iterator := sdk.KVStorePrefixIterator(store, []byte(types.KeyPool))
	defer iterator.Close()

	var pools []types.Pool
	for ; iterator.Valid(); iterator.Next() {
		var pool types.Pool
		err := k.cdc.Unmarshal(iterator.Value(), &pool)
		if err != nil {
			return nil, fmt.Errorf("failed to unmarshal pool: %w", err)
		}
		pools = append(pools, pool)
	}

	return pools, nil
}
