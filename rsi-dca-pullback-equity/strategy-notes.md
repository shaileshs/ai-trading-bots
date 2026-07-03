# Strategy: RSI Pullback + DCA Ladder (Long Equity)

Source: flowchart screenshot (bot automation logic tree), captured 2026-07-02.

## Summary

This is a long-only equity strategy that (1) waits for a pullback/oversold setup to open an initial position, then (2) averages down in up to 4 additional tagged legs as the position loses value, subject to a stack of on/off gates.

## Entry gating (all must pass, in order)

1. **Timing** — Either "Open at EOD" is off, or it's on and current time is after 3:30pm.
2. **Cooldown** — Bot did NOT open a position exactly 0 days ago (i.e., not same-day re-entry).
3. **PDT rule** — Either "Allow PDT" is on, or it's off and the bot did NOT close a position exactly 0 days ago (avoids pattern-day-trade violations).
4. **Capacity** — Bot's general "can open a new position" check passes.
5. **RSI filter** (optional) — Either RSI Check is off, or it's on and the symbol's intraday RSI(14) is below a configured RSI value (oversold trigger).
6. **Earnings filter (stocks only)** — Either the symbol isn't a stock, or it is a stock and does NOT report earnings within 3 market days (avoids earnings-related volatility).
7. **VIX filter** (optional) — Either VIX minimum is off, or it's on and VIX is above the configured minimum (only trade when volatility regime is elevated enough).

## Two branches after gating: initial entry vs. DCA add-on

**Branch A — no position yet (0 Long Equity positions held):**
- Trend/dip filter: either "USE 10EMA" is on and price is below the 10-day EMA, or it's off and price is below the 5-day SMA.
- Optional "Open on UP Days Only": either that setting is off, or price has already increased $0.01+ intraday since open.
- If all pass → **Open Symbol Long Equity** (initial position).

**Branch B — bot already holds 1–4 Long Equity positions (the initial trend/dip filter is skipped since a position exists):**
- Same "Open on UP Days Only" check applies.
- Then loops through position counts 1 → 2 → 3 → 4, and for whichever count currently matches, checks: has the position's premium dropped by an APTR (a multiple of ATR — Average/Adjusted Price True Range) amount since it was opened, AND does the position carry the matching sequence tag ("initial position" / "two" / "three" / "four")?
- If both true → **Open Symbol Long Equity** (adds another leg, effectively martingale/DCA-style averaging down).
- Caps out at 4 positions — no 5th add-on is defined.

## Net effect

This is a scaled dollar-cost-averaging (DCA) strategy: enter long on an oversold/pullback signal (RSI low, price below a moving average), then add to the position in up to 4 further tranches each time it drops by another ATR-based increment, as long as timing/PDT/earnings/VIX gates all stay green. There's no visible stop-loss or profit-taking exit in this screenshot — it only shows the entry/add-on logic, not exits.

## Open questions before building this

- What's the exit/take-profit and stop-loss logic (not shown here)?
- What RSI threshold, VIX minimum, and APTR multiplier values are configured?
- Position sizing per leg — fixed dollar amount, or increasing/decreasing per tranche?
- Which symbols/watchlist is this intended to run against?

Small change
