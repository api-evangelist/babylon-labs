---
name: Check Babylon BTC staking status
description: Look up a staker's Bitcoin staking delegations and network context on the
  public Babylon Staking API — verify an address has an active delegation, list its
  delegations, and enrich with finality-provider and network info.
api: openapi/babylon-labs-staking-api-openapi-original.yml
base_url: https://staking-api.babylonlabs.io
auth: none
operations:
- GET /v1/staker/delegation/check
- GET /v2/delegations
- GET /v2/finality-providers
- GET /v2/network-info
---

# Check Babylon BTC staking status

The Babylon Staking API is a **public, unauthenticated, read-only** REST API over
on-chain Bitcoin staking state. Base URL: `https://staking-api.babylonlabs.io`.
Responses use a shared envelope `{ err, errorCode, statusCode }` on errors (see
`errors/babylon-labs-problem-types.yml`). List endpoints are **cursor-paginated**:
pass `pagination_key` and follow `pagination.next_key`.

## Steps

1. **Confirm an address is staking.** Call `GET /v1/staker/delegation/check?address=<btc_address>`
   (Taproot or Native Segwit). Optionally add `timeframe=today`. The response reports
   whether the address has an active delegation.

2. **List the staker's delegations.** Call `GET /v2/delegations` for the staker to
   retrieve delegation records (state, staking tx, finality provider, amounts). If
   `pagination.next_key` is non-empty, repeat with `pagination_key=<next_key>` until
   exhausted.

3. **Resolve finality providers.** For each delegation's finality-provider BTC public
   key, call `GET /v2/finality-providers` to enrich with moniker and active TVL.

4. **Add network context.** Call `GET /v2/network-info` for current global staking
   parameters when interpreting states, unbonding periods, or eligibility.

## Notes

- Prefer **v2** endpoints; the `v1/*` equivalents (`/v1/delegation`, `/v1/global-params`,
  `/v1/stats`, `/v1/finality-providers`, `/v1/stats/staker`) are **deprecated** — see
  `lifecycle/babylon-labs-lifecycle.yml`.
- On `NOT_FOUND` / `BAD_REQUEST` inspect `errorCode`; the string enum is the stable
  machine-readable classifier.
