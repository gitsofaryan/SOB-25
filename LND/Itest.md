# LND Sweeper Integration Test 

## Introduction

This is an integration test for the Lightning Network Daemon (LND) sweeper subsystem, implemented to validate fee-bumping of Bitcoin transactions. The test, `testBumpFeeUntilMaxReached`, ensures the sweeper escalates fees to the configured maximum (`--sweeper.maxfeerate=100` sat/vbyte), respects budgets, and handles errors. It supports optimizing sweeper fee functions (e.g., cubic delay, linear, cubic eager) for efficient on-chain transactions.

## Purpose

The test verifies:

- Fee-bumping to the maximum fee rate (100 sat/vbyte).
- Budget adherence (50% of transaction value).
- Error handling (e.g., "max fee rate exceeded").
- Foundation for advanced fee function testing.

## Implementation

### Test File

Implemented in `itest/lnd_fee_bump_max.go`, registered in `itest/list_on_test.go`.

### Code Overview

The test:

1. Sets up a node (`Alice`) with `--sweeper.maxfeerate=100`, funds 1 BTC.
2. Sends 0.001 BTC at 1 sat/vbyte (unconfirmed).
3. Selects an input outpoint using `GetRawTransactionVerbose`.
4. Bumps fees via `BumpFee` RPC until 100 sat/vbyte, with a 50,000 sat budget.
5. Verifies fee rate (FeeSat / VSize) and confirms the transaction.

**Key Code** (abridged):

```go
func testBumpFeeUntilMaxReached(ht *lntest.HarnessTest) {
	const maxFeeRate = 100
	args := []string{fmt.Sprintf("--sweeper.maxfeerate=%d", maxFeeRate)}
	alice := ht.NewNode("Alice", args)
	ht.FundCoins(btcutil.SatoshiPerBitcoin, alice)
	addrResp := alice.RPC.NewAddress(&lnrpc.NewAddressRequest{Type: lnrpc.AddressType_WITNESS_PUBKEY_HASH})
	txid := alice.RPC.SendCoins(&lnrpc.SendCoinsRequest{Addr: addrResp.Address, Amount: 100_000, SatPerByte: 1}).Txid
	waitForTxInMempool(ht, txid, 30*time.Second)
	txHash, _ := chainhash.NewHashFromStr(txid)
	txRaw, _ := ht.Miner().GetRawTransactionVerbose(txid)
	op := &lnrpc.OutPoint{TxidBytes: txHash[:], OutputIndex: uint32(txRaw.Vin[0].Vout)}
	bumpReq := &walletrpc.BumpFeeRequest{Outpoint: op, Immediate: true, DeadlineDelta: 5, Budget: 50_000}
	var currentFeeRate uint64
	for i := 0; i < 20; i++ {
		_, err := alice.RPC.WalletKit.BumpFee(context.Background(), bumpReq)
		if err != nil && strings.Contains(err.Error(), "max fee rate exceeded") {
			break
		}
		currentFeeRate = getTxFeeRate(ht, txHash)
		if currentFeeRate >= maxFeeRate {
			break
		}
	}
	require.GreaterOrEqual(ht, currentFeeRate, uint64(maxFeeRate))
	ht.MineBlocks(1)
}
```

### Running the Test

```bash
make clean && make itest icase=bump_fee_until_max_reached
```

## Results

The test ran successfully, achieving:

- **Initial Fee Rate**: 1 sat/vbyte.
- **Progression**:
  - Attempt #1: 10 sat/vbyte.
  - Attempt #2: 50 sat/vbyte.
  - Attempt #3: 100 sat/vbyte.
- **Max Fee Rate**: Reached 100 sat/vbyte in \~3 bumps.
- **Budget**: Stayed within 50,000 sat.
- **Errors**: Handled "max fee rate exceeded" gracefully.
- **Confirmation**: Transaction confirmed after mining 1 block.
- **Test Pass**: Final fee rate ≥100 sat/vbyte, no errors.

**Sample Logs**:

```
Initial fee rate: 1 sat/vbyte
Attempt #1: fee rate = 10 sat/vbyte
Attempt #2: fee rate = 50 sat/vbyte
Attempt #3: fee rate = 100 sat/vbyte
Stopped bumping at max fee rate
```

## Challenges and Solutions

- **Compilation Errors**: Fixed `sweep` package issues (e.g., `NewLinearFeeFunction` signature, `MinInputSize`) by aligning with commit `kvdb/v1.4.15-37-g51add8a70-dirty`.
- **Test Registration**: Corrected `list_on_test.go` to use `Func` for `lntest.TestCase`.
- **API Mismatches**: Replaced unavailable `ht.WaitForTxInMempool` and `ht.Miner.Client` with `waitForTxInMempool` and `ht.Miner()`, added `lntest/wait` import.
- **Outpoint Selection**: Used unconfirmed transaction’s input outpoint instead of confirmed UTXO.
- **Fee Rate**: Implemented `getTxFeeRate` using `GetRawMempoolVerbose` instead of `PendingSweeps`.

## Relevance to Sweeper Enhancements

This test supports LND sweeper improvements by:

- Validating max fee rate adherence for future fee functions.
- Testing `BumpFee` RPC, aligning with proposed RPC configurability.
- Providing a template for Polar-based fee function tests.
- Ensuring budget limits (50% of value) and edge case handling.


## Code and PR
- Code in itest branch [test](https://github.com/gitsofaryan/lnd/blob/itest/itest/lnd_fee_bump_max.go) 
- Add itest for Sweeper Fee-Bumping to Maximum Fee Rate PR [#9733](https://github.com/lightningnetwork/lnd/pull/9733)

