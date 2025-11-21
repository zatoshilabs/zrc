# ZRC-721: Zordinals NFT Inscription Standard

ZRC-721 is a lightweight, open NFT inscription standard for Zordinals (Ordinals on Zcash). It borrows the minimal envelope style of BRC-20 and the metadata conventions of ERC-721. Each mint inscription _is_ the NFT (supply is always 1), so there is no transfer operation beyond moving the inscription itself.

## Goals
- Minimal ops: only `deploy` and `mint`
- Deterministic supply and token IDs
- Off-chain metadata and imagery anchored by IPFS CIDs for provenance
- Optional royalty hint (basis points) for secondary markets to enforce
- Open, permissionless, wallet/indexer friendly

## Inscription Envelopes
All inscriptions are UTF-8 JSON. Recommended content type: `text/plain;charset=utf-8` (or `application/json` if your tooling prefers).

### Deploy inscription
Declares a collection and its metadata root.

```json
{
  "p": "zrc-721",
  "op": "deploy",
  "collection": "ZGODS",
  "supply": "10000",
  "meta": "bafybeicqjqzixdtawkbcuyaagrmk3vyfweidwzb6hwbucadhoxoe2pd3qm",
  "royalty": "100"
}
```

**Fields**
- `p`: Protocol id. Must be the literal string `zrc-721`.
- `op`: Operation. Must be `deploy`.
- `collection`: Case-sensitive collection slug/name (keep it short and unique).
- `supply`: Total tokens to be minted (stringified integer, `>= 1`).
- `meta`: IPFS CID that points to the metadata folder root (each token’s JSON lives under this root).
- `royalty`: Optional secondary-sale royalty in basis points (stringified integer, e.g., `100` = 1%). Intended as a hint for marketplaces to optionally enforce and pay to the transparent address that inscribed the `deploy`.

### Mint inscription
Creates a single NFT for a given collection.

```json
{
  "p": "zrc-721",
  "op": "mint",
  "collection": "ZGODS",
  "id": "0"
}
```

**Fields**
- `p`: Protocol id. Must be `zrc-721`.
- `op`: Operation. Must be `mint`.
- `collection`: Must match an existing `deploy` inscription.
- `id`: Token identifier (stringified integer, typically 0-indexed). Each `id` can be minted once and must be `< supply`.

**Transfer model**: No `transfer` inscription is specified. The mint inscription itself is the NFT; moving the inscription UTXO moves the NFT.

## Metadata and Media (IPFS)
- Store metadata as individual JSON files at `ipfs://<meta_cid>/<id>.json`.
- Within each metadata file, include an IPFS link to the image: `img` (or `image`) should be an `ipfs://` URI; gateways like `https://ipfs.io/ipfs/<cid>/...` are acceptable fallbacks.
- Images can be any format supported by your ecosystem (PNG, JPG, GIF, etc.) and should live under the same or a separate IPFS CID.

### Recommended metadata schema
```json
{
  "name": "ZGODS 0",
  "collection": "ZGODS",
  "description": "The first ZRC-721 Inscription Collection on the ZCash Privacy Blockchain. We are the ZGODS, expect us.",
  "website": "https://example.com",
  "twitter": "https://x.com/example",
  "img": "ipfs://<image_cid>/0.png",
  "attributes": [
    { "trait_type": "Background", "value": "metro fiber junction" }
  ]
}
```
- `name`, `collection`, `description`, `img` are strongly recommended.
- `attributes` follows the familiar ERC-721/1155 metadata array of trait objects.
- Extra fields (links, creator info, animation_url, etc.) are allowed and should remain stable once published.

## End-to-End Flow
1) **Prepare metadata and media**: Upload token JSON files and images to IPFS. Keep file names aligned with token IDs (e.g., `0.json`, `0.png`).
2) **Deploy**: Inscribe the `deploy` JSON to establish the collection, supply, and metadata root CID.
3) **Mint**: For each token ID, inscribe a `mint` JSON. Ensure each `id` is unique and within the declared `supply`.
4) **Transfer/ownership**: Ownership changes by transferring the inscription UTXO that contains the mint inscription.

## Example set (from this repo)
- Deploy envelope: `deploy.json`
- Mint envelope: `mint.json`
- Token metadata example: `0.json` (references an IPFS-hosted image at `bafybeiaqmceddfi4y3dyqwepjs6go477x35ypaojwgegcsee2vgy63yobq/0.png`)

## Openness
This specification is offered as an open, public standard. Use, fork, and implement freely in wallets, indexers, and minting tools.
