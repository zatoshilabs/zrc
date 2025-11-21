# ZRC-20: Zcash Fungible Token Standard

ZRC-20 is a minimal, BRC-20–style fungible token standard for Zcash. Token state is tracked by JSON inscriptions: `deploy`, `mint`, and `transfer`. Indexers/wallets determine balances by following inscription ownership (no smart contracts required).

## Envelope Format
- UTF-8 JSON payloads. Suggested content type: `text/plain;charset=utf-8` (or `application/json`).
- Common fields: `p` must be `zrc-20`; `tick` is the token ticker (commonly 4 uppercase letters).
- Numeric fields are stringified integers.
- Canonicality: Only the first valid `deploy` for a given `tick` is recognized; later deploy attempts with the same `tick` should be ignored by indexers.

### Deploy
Declares a token, its max supply, and per-mint cap.

```json
{
  "p": "zrc-20",
  "op": "deploy",
  "tick": "ZERO",
  "max": "21000000",
  "lim": "1000"
}
```

Fields:
- `op`: Must be `deploy`.
- `tick`: Token ticker; treat as case-sensitive (recommend uppercase, 4 chars).
- `max`: Maximum total supply that can ever be minted.
- `lim`: Maximum amount per `mint` inscription.

### Mint
Creates fungible units up to the per-mint limit and until `max` is reached.

```json
{
  "p": "zrc-20",
  "op": "mint",
  "tick": "ZERO",
  "amt": "1000"
}
```

Fields:
- `op`: Must be `mint`.
- `tick`: Must match an existing deploy (case-sensitive).
- `amt`: Amount to mint (must be `<= lim`; cumulative mints cannot exceed `max`).

Ownership: The holder of the mint inscription UTXO owns the minted amount.

### Transfer
Moves balances by inscribing a transfer and then sending that inscription UTXO to the recipient.

```json
{
  "p": "zrc-20",
  "op": "transfer",
  "tick": "ZERO",
  "amt": "500"
}
```

Fields:
- `op`: Must be `transfer`.
- `tick`: Token ticker.
- `amt`: Amount to move.

Transfer settlement: Balances update when the transfer inscription is sent to the recipient’s address (the inscription UTXO movement finalizes the transfer).

## End-to-End Flow
1) **Deploy** a token (`deploy.json`).
2) **Mint** inscriptions (`mint.json`), respecting `lim` and `max`.
3) **Transfer** by inscribing `transfer` and sending that inscription to the recipient.
4) **Indexing**: Wallets/marketplaces sum mints and transfers by current inscription ownership to compute balances.

## Permissionless & Open Source
This specification is open and permissionless. Implement freely in wallets, indexers, bridges, and marketplaces on Zcash.
