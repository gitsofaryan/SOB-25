# Current Sweeper Fee Function

## Overview

*How are the `startFeeRate` and `endFeeRate` of the fee function currently defined, and can they be adjusted?* My name is Aryan Jain, and I’m submitting this as part of my application to contribute to LND.

The response is based on my analysis of the `LinearFeeFunction` in `sweep/fee_function.go` and the sweeper configuration in `lncfg/sweeper.go`. It provides a clear explanation of how `startFeeRate` and `endFeeRate` are defined and their adjustability, ensuring alignment with LND’s technical requirements.

## Definition and Adjustability of `startFeeRate` and `endFeeRate`

The `LinearFeeFunction` in `sweep/fee_function.go` defines `startFeeRate` and `endFeeRate` as key parameters for calculating fee rates during transaction sweeping. Below, I detail their definitions and adjustability based on the provided code.

### `startFeeRate`
- **Definition**:
  - The initial fee rate used when broadcasting a transaction, set in the `NewLinearFeeFunction` constructor.
  - Determined in one of two ways:
    - If an explicit `startingFeeRate` is provided via the `fn.Option[chainfee.SatPerKWeight]` parameter, that value is used.
    - Otherwise, it is estimated by calling `estimateFeeRate(confTarget)`, which queries the fee estimator (e.g., Bitcoin Core’s `estimatesmartfee`) for the given confirmation target (`confTarget`).
  - If `confTarget >= 1,008` (maximum block target defined by `chainfee.MaxBlockTarget`), the minimum relay fee (`estimator.RelayFeePerKW()`) is used instead.
  - The estimated fee rate is capped at `endingFeeRate` to prevent overpayment.
  - If `confTarget <= 1` (imminent deadline), `startFeeRate` is set to `maxFeeRate` to prioritize immediate confirmation.
- **Units**: `chainfee.SatPerKWeight` (sats per 1,000 weight units, approximately equivalent to sats/vbyte for standard transactions).
- **Example**: For `confTarget = 40`, the estimator might return ~1 sat/vbyte (minimum relay fee) in a low-fee environment, or a higher rate based on mempool conditions.
- **Adjustability**:
  - **Configuration**:
    - Explicitly set by passing a specific `startingFeeRate` to `NewLinearFeeFunction`, allowing precise control.
    - Influenced by configuring the fee estimator, such as adjusting Bitcoin Core’s fee estimation parameters (e.g., `estimatesmartfee` settings).
    - The minimum relay fee can be adjusted via Bitcoin Core’s `--minrelaytxfee`, which LND’s estimator respects.
  - **Runtime**: Fixed after the `LinearFeeFunction` is initialized. Changing it requires creating a new `LinearFeeFunction` instance, which is not dynamic during an active sweep.
  - **Practical**: Operators can set `startFeeRate` to a higher value (e.g., 10 sats/vbyte) in high-fee environments by passing a custom value or tuning the estimator to reflect current market conditions.

### `endFeeRate`
- **Definition**:
  - The maximum fee rate used as the deadline approaches, set in `NewLinearFeeFunction`.
  - Provided as the `maxFeeRate` parameter, typically derived from the budget (default 50% of HTLC value) divided by the estimated transaction weight (e.g., 1,142 weight units for incoming HTLCs).
  - Capped at the configured `maxFeeRate`, which is set via the `Sweeper.MaxFeeRate` configuration (default 1,000 sats/vbyte, as defined in `lncfg/sweeper.go`).
  - If `confTarget <= 1`, both `startFeeRate` and `endFeeRate` are set to `maxFeeRate`.
  - The fee rate is capped at `MaxAllowedFeeRate` (10,000 sats/vbyte) and must be at least `MaxFeeRateFloor` (100 sats/vbyte), as enforced by `Sweeper.Validate()`.
- **Units**: `chainfee.SatPerKWeight`.
- **Example**: For a 300,000-sat HTLC with a 50% budget and 1,142 wu, `endFeeRate = (300,000 * 0.5) / 1,142 ≈ 131 sats/vbyte`. For a 600,000-sat HTLC, `endFeeRate` is capped at `maxFeeRate = 1,000 sats/vbyte`.
- **Adjustability**:
  - **Configuration**:
    - Set via the `--sweeper.maxfeerate` flag or `lnd.conf` (default 1,000 sats/vbyte, validated to be between 100 and 10,000 sats/vbyte by `Sweeper.Validate()`).
    - Influenced by the budget configuration (`Sweeper.Budget`, default 50% of HTLC value), adjustable via `--sweeper.budget.deadlinehtlc`. For example, a 25% budget reduces `endFeeRate` to ~65 sats/vbyte for a 300,000-sat HTLC.
    - The transaction weight, calculated internally based on HTLC type, indirectly affects `endFeeRate` but is not directly configurable.
  - **Runtime**: Fixed after initialization. Changing it requires reinitializing the `LinearFeeFunction`, which disrupts active sweeps.
  - **Practical**: Operators can lower `maxFeeRate` to 500 sats/vbyte or adjust the budget to 25% using `lncli` or `lnd.conf` to reduce fees, but setting it too low risks confirmation delays in high-fee environments.

## Summary
- **startFeeRate**:
  - **Defined**: Explicitly via `startingFeeRate` or estimated by the fee estimator for `confTarget`, capped at `endFeeRate`. Uses minimum relay fee for `confTarget >= 1,008` or `maxFeeRate` for `confTarget <= 1`.
  - **Adjustable**: Yes, via explicit setting, fee estimator configuration, or minimum relay fee adjustments. Fixed after initialization.
- **endFeeRate**:
  - **Defined**: Derived from budget divided by transaction weight, capped at `--sweeper.maxfeerate` (default 1,000 sats/vbyte). Set to `maxFeeRate` for `confTarget <= 1`.
  - **Adjustable**: Yes, via `--sweeper.maxfeerate` (100 to 10,000 sats/vbyte) or `--sweeper.budget.deadlinehtlc`. Fixed after initialization.
- **Limitations**: Both rates are static during a sweep, requiring a new `LinearFeeFunction` for changes, which highlights the need for more flexible fee strategies.

## References
- LND source code: `sweep/fee_function.go`, `lncfg/sweeper.go`



