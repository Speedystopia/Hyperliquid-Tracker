# Hyperliquid Portfolio Monitor

A single HTML file that reads any public Hyperliquid address and shows what that
account actually owns, what it owes, and what would liquidate it.

No install, no build, no keys. Open the file in a browser, paste an address,
press **Run**.

---

## Quick start

1. Open `hl-portfolio.html` in any modern browser.
2. Paste a public address (`0x…`, 40 hex characters).
3. Press **Run**.

The page opens blank — no address is preloaded, and nothing is stored between
sessions.

---

## What it shows

**Risk banner** — the first thing on the page. If a short is over-hedged against
its own collateral, it says so, gives the price at which the position breaks, and
sizes the fix. Green when there is nothing to do.

**Tiles** — total equity, perpetuals, spot, 24h P&L, staking and vaults, health
factor. Consolidated across the master account and every sub-account by default;
one button switches to master-only.

**Accounts** — one row per account, colour-coded, with equity, positions, gross
notional, maintenance margin and utilisation. Click any row to filter the whole
page to that account.

**Equity curve** — account value or P&L, over 24h / 7d / 30d / all time, with the
perpetuals-only series overlaid.

**Performance** — P&L and volume by period, split between perpetuals and spot.

**Holdings** — every balance the address owns, wherever it sits. Hyperliquid keeps
capital in four separate places and only the first is a spot balance:

| Tag | Meaning |
|---|---|
| *(none)* | spot inventory — the only balances eligible as portfolio-margin collateral |
| `perp collateral` | USDC posted against the clearinghouse to margin open positions |
| `staking` | delegated or undelegated HYPE, subject to an unstaking queue |
| `vault` | equity deposited in a vault, subject to its lock-up |

**Perpetual positions** — size, entry, mark, unrealised P&L, funding received,
liquidation prices, and the next predicted funding on Binance and Bybit for
comparison. Each account gets a header row with its USDC collateral, maintenance
margin, buffer and utilisation.

**Resting orders** — open limit and trigger orders, with distance to mark and the
notional that would be added if every order filled.

**Vault positions** — vault name, leader, APR, your equity, your P&L, days
following, and the lock-up expiry.

**Net delta** — spot quantity plus perpetual size per asset, staking included. At
zero the book is delta-neutral and only funding accrues.

**Borrowing and margin** — outstanding borrow, rate, interest cost, collateral
credit, liquidation value, your actual fee tier, net capital deposited and lifetime
return on it, and any API agent authorised on the account.

**Debt repayment** — appears when there is a borrow. Pick an asset and a share of
the debt; it computes how much to sell, the execution cost at your real fee rate,
interest saved, collateral credit forgone, and the net effect on margin.

---

## The risk calculation

This is the part no exchange screen gives you.

When spot is posted as collateral against a short, **a rally is what liquidates the
position, not a decline**. Two separate limits apply, and they are not the same
number:

- **Borrow capacity** — quantity × LTV (0.65 for HYPE). Past this, a rally forces
  the protocol to borrow USDC against your collateral, and at some price that
  borrowing hits its ceiling.
- **Portfolio-margin liquidation** — quantity × liquidation threshold ÷ (1 +
  maintenance margin). The threshold is `0.5 + 0.5 × LTV`, so 0.825 at an LTV of
  0.65. Below this hedge ratio the account *gains* liquidation value as the price
  rises and no rally can liquidate it.

Both trigger prices are computed from the live position and shown in the perpetual
positions table, alongside Hyperliquid's own liquidation price — which is
calculated on the perpetual account in isolation, ignores spot collateral, and is
therefore far too conservative for a hedged book.

---

## What it does not do

- **No order history.** Positions and balances come from state endpoints and are
  always current. Nothing is reconstructed from fills.
- **No keys, no signing, no trading.** Read-only against public endpoints. It
  cannot place an order, move funds, or revoke anything.
- **No storage.** Nothing is written to disk or to the browser.

---

## Security

- **No secrets in the file.** No addresses, keys, tokens or credentials are
  embedded. The only external host it contacts is `api.hyperliquid.xyz`.
- **No storage.** No `localStorage`, no cookies, no IndexedDB. Nothing persists
  between sessions.
- **No `eval`, no dynamic code.** One `fetch` call, to the public info endpoint.
- **API data is escaped before rendering.** Token tickers, HIP-3 market names,
  vault names and descriptions, sub-account names and API-agent labels are free
  text chosen by third parties. Every one is HTML-escaped, so a hostile name
  renders as text instead of executing. This is verified against an injection
  test suite.

## Endpoints used

All public, all on `api.hyperliquid.xyz/info`:

`clearinghouseState` · `spotClearinghouseState` · `portfolio` · `subAccounts` ·
`perpDexs` · `metaAndAssetCtxs` · `spotMetaAndAssetCtxs` · `allMids` ·
`allBorrowLendReserveStates` · `borrowLendUserState` · `delegatorSummary` ·
`userVaultEquities` · `vaultDetails` · `frontendOpenOrders` · `userFees` ·
`webData2` · `extraAgents` · `predictedFundings`

---

## Limits worth knowing

- **Sub-accounts are separate margin accounts.** Collateral in one does not support
  a position in another, and a liquidation in one leaves the others untouched. The
  risk calculation therefore covers the master account only — the only one that
  uses portfolio margin.
- **HIP-3 venues** (xyz, vntl, flx and the rest) are separate margin accounts too,
  and are excluded from the same calculation.
- **Consolidated P&L costs one request per sub-account**, so it is capped at 20.
  Beyond that, equity still consolidates — it ships inside the `subAccounts`
  response — but P&L stays master-only and the tile says so.
- **Trigger prices hold other positions at their current mark** and exclude future
  funding. Positive funding accrual pushes the constraints further out.
- **Equity is not notional.** Perpetual equity is posted collateral plus unrealised
  P&L. The gross notional of the positions is leverage and is never added to
  equity; it appears as a memo line only.

---

## Licence

MIT. See `LICENSE`.

This is an analytics tool, not financial advice. Trigger prices are derived from
public data and documented formulas; verify them against the exchange before
acting on them.
