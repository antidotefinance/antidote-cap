# Antidote supply cap

The Antidote cap program is the capped, rule-bound mint authority of the Antidote token on Solana.
The token's mint authority is handed to the program's PDA `["mint_auth", mint]` in the same
transaction that registers the mint, and from then on the cap program is the only thing that can
mint it, only under fixed rules:

- **No admin, no setters.** `register` binds the mint, the lifetime cap, the one minter and the
  per-day ceiling once; nothing can change them afterwards, and there is no way to hand the mint
  authority back. The program is deployed without IDL instructions (feature `no-idl`); its
  upgrade authority is shown on its Solscan page.
- **1B cap: supply plus owed.** `supply + owed <= lifetime_cap` always holds, where owed is what
  was reserved but not yet minted. Antidote registers with a cap of 1,000,000,000 tokens. Burned
  tokens leave the supply and free room again.
- **Per-day ceiling.** New commitments (a reservation, or a mint beyond what was reserved) in one
  UTC day are limited to the registered daily ceiling. Minting what was reserved earlier never
  counts again, so a reservation is always honored.
- **One minter.** Only the registered minter (the Antidote router's authority PDA) can call
  `reserve` and `mint_to`.
- The mint may have no freeze authority, and on Token-2022 only metadata pointer, token metadata
  and transfer hook extensions.

Instructions: `register`, `reserve`, `mint_to`, and `migrate_cap` (anyone, pays rent: grows a
cap account created before `owed` existed; changes nothing else).

| Program | Directory | Library name | Program id |
| --- | --- | --- | --- |
| levcap | programs/levcap | levcap | Gq9eEG2mnAefVGtcgnmZeutUitKvZoxErxL27RALJ7U |

The other Antidote programs (router, transfer hook, fee router) are verified from
https://github.com/antidotefinance/antidote-programs.

## Verify

Anchor 0.31.1, solana-cargo-build-sbf 3.1.12, platform-tools v1.52, built with solana-verify 0.5.2
in the image pinned by digest (never a tag):

```sh
cargo install solana-verify --version 0.5.2 --locked
solana-verify build --library-name levcap \
  --base-image solanafoundation/solana-verifiable-build@sha256:5d7b60cdda413d3a482b29f988e434606e206556590d1175cbee69a2728ddfc6
solana-verify get-executable-hash target/deploy/levcap.so
solana-verify -u https://api.mainnet-beta.solana.com get-program-hash Gq9eEG2mnAefVGtcgnmZeutUitKvZoxErxL27RALJ7U
```

The two hashes must be equal. Or in one step:

```sh
solana-verify verify-from-repo -u https://api.mainnet-beta.solana.com https://github.com/antidotefinance/antidote-cap \
  --program-id Gq9eEG2mnAefVGtcgnmZeutUitKvZoxErxL27RALJ7U --library-name levcap --commit-hash <commit> \
  --base-image solanafoundation/solana-verifiable-build@sha256:5d7b60cdda413d3a482b29f988e434606e206556590d1175cbee69a2728ddfc6
```

Security: report vulnerabilities privately, see SECURITY.md.

License: Business Source License 1.1 (BUSL-1.1), see LICENSE. Licensor Antidote; no Additional Use
Grant; each version converts to the Apache License, Version 2.0 on its Change Date (two years after
its release; LICENSE names the date).
