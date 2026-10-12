# clone-lin-gmx-funding

Experimental i64-scale LIN clone of [gmx-io/gmx-contracts](https://github.com/gmx-io/gmx-contracts) VaultUtils **getFundingFee** (`size * (cumulativeFundingRate - entryFundingRate) / 1e6`).

- **LIN clone:** [`src/lin_gmx_funding.lin`](src/lin_gmx_funding.lin)
- **C11 oracle:** [`test/oracles/gmx_funding_c11.c`](test/oracles/gmx_funding_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_GMX_FUNDING_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** a GMX Vault, **not** GLP, and does **not** claim Solidity uint256 / mainnet parity. The kernel is the GMX V1 **perpetual funding fee**: size times the delta of the cumulative funding rate versus the position entry rate, divided by `FUNDING_RATE_PRECISION=1e6`. Companion `getPositionFee` uses `/10000`. Confusing the two denominators overcharges **100x**. It is **not** GMX `getFeeBasisPoints` USDG tax/rebate, **not** Uniswap V2 `xy`, **not** Solidly cubic `x3y+y3x`, **not** Saddle/Curve `getD`, and **not** Maker `rpow`.

Upstream is MIT. This LIN kernel is an original uint64-scale implementation of the published `getFundingFee` formula, not a verbatim Solidity dump. `getFundingFee` / `getPositionFee` / `cumulativeFundingRates` / `entryFundingRate` / `FUNDING_RATE_PRECISION` are rejected by the Solidity transpiler (`REJ_SOLIDITY_GMX_FUNDING`).

Pinned upstream: commit `fe55c4ba8911ae2378b50bfb9bece5ee50cf2af8`, `contracts/core/VaultUtils.sol` sha256 `ad831f3c8d675c42665c854368bc06696e1cba23bd3294f91aa2ce4c0a8fc495` blob `8e29c3ff19258d191f6d2459c110a917ba1f9906`.

Canonical goldens (uint64-scale, 128-bit product):

- size=1e6, rate=1000, entry=0: funding **1000**
- same inputs with `/10000` (confused bps): **100000** (100x overcharge)
- size=1e6, cum=5000, entry=4000: **1000** vs ignore-entry **5000** (5x)
- size=1e6, entry>cum: **0** (fail-closed; Solidity SafeMath.sub reverts)
- position 10 bps of 1e6: **1000** (same number as funding 1000, different denominator)
- Uniswap 30 bps one-shot of 1e6: **3000** (swap fee, not accrued funding)
- naive i64 `(5e9*5e9)/1e6` wraps to **6553255926290**; 128-bit muldiv = **25000000000000**
