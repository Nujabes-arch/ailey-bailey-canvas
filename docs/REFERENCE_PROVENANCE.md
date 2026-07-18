# Reference provenance

This document records the public upstreams and exact file evidence used when shaping the local study-teacher contracts. It does not claim access to, recovery of, or possession of a private GPT prompt.

## Review record

- Reviewed at: `2026-07-18 21:33:11 +09:00` (`Asia/Seoul`)
- Method: fetched the public repositories, inspected the reviewed commit trees and commit metadata, resolved Git blob IDs for the cited files, and read each repository's `LICENSE`.
- Local repository: `Nujabes-arch/ailey-bailey-canvas`, branch `codex/v2-parity-hardening`

## Upstream A — `lemos999/ailey-bailey-canvas`

| Field | Verified value |
|---|---|
| Public URL | https://github.com/lemos999/ailey-bailey-canvas |
| Reviewed commit | `8a36e77d025bb9c258bfeaf8587424783140b185` (`Update`) |
| Commit author | `Fewweekslater` (`lemos999`) |
| Commit date | `2026-04-02 08:02:58 +09:00` |
| Repository role | Persona definitions, learning-flow/prompt architecture, Codex and Canvas trigger material, and bundled Canvas assets |
| License path / blob | `LICENSE` / `5cf6d90d47f938431496f6a33096edec7d6d303c` |
| License | CC BY-NC-SA 4.0 |
| License URL | https://creativecommons.org/licenses/by-nc-sa/4.0/ |

### Reviewed paths and local relationships

| Upstream path | Git blob SHA-1 | Local relationship |
|---|---|---|
| `Ailey & Bailey X대화형_260323.md` | `adf2005e1f6097fde69b686d320e467c9308be10` | `reference_only/Ailey & Bailey X대화형_260323.md` has the same Git blob and local SHA-256 `437959EF66D17A91C882547A93AA7875D1C55E30A0EC916C37AB277084B27406`; byte-identical at the reviewed revision |
| `Ailey & Bailey X_251023(Stable)_신.txt` | `c576716208533a06ba455ecb45db84abd9f20fb4` | Stable prompt comparison source; not represented as a byte-identical operational file |
| `prompt_src/00_core/A_system_core.prompt.txt` | `f508494460b2fab102fbd878ac6f314b333bd64f` | Core routing and output-structure comparison source |
| `prompt_src/01_personas/P_personas_core.prompt.txt` | `8d5aacd98d7d2d49bb73890e3c305d54e0039466` | Persona comparison source |
| `prompt_src/03_modules/M_Ailey_Bailey_Codex_Engine.prompt.txt` | `2af16ba36de8a8eea6220f90686973b6fb387249` | Learning/Codex module comparison source |
| `prompt_src/04_canvas_engine/M_C_canvas_engine.prompt.txt` | `6dccc69e67a69cb746b9d765e77e37d85e004425` | Canvas contract comparison source |
| `bundle/main.js` | `308f7cb6c92e3bd7157ce82cf6ffbde6edec39d2` | Public Canvas runtime reference; not copied into the local bundle |
| `bundle/main.css` | `137dda605aa54e08c75d286739a3135fa9977cc4` | Public Canvas style reference; not copied into the local bundle |

The local `00_PROJECT_INSTRUCTIONS.md` and `knowledge_authoritative/*.md` are independently organized adaptations of the observed behavior and cited public material. They are not claimed byte-identical to any upstream path.

## Upstream B — `lemos999/Singulari-Tea-Codex-Canvas`

| Field | Verified value |
|---|---|
| Public URL | https://github.com/lemos999/Singulari-Tea-Codex-Canvas |
| Reviewed commit | `04f55932c7eb6a90ff194f66565f2db7e01d79c1` (`Sync live Ailey bundle output`) |
| Commit author and committer | `Fewweekslater` (`lemos999`) |
| Commit date | `2026-04-17 21:29:07 +09:00` |
| Repository role | Stable prompt, external `.cc` bundle, CSS/JavaScript runtime, and KaTeX asset reference |
| License path / blob | `LICENSE` / `8cc0449cb12587110fbd4def2cc9abdf24e5210a` |
| License | CC BY-NC-SA 4.0 |
| License URL | https://creativecommons.org/licenses/by-nc-sa/4.0/ |

### Reviewed paths and local relationships

| Upstream path | Git blob SHA-1 | Local relationship |
|---|---|---|
| `Ailey & Bailey X_251023(Stable)_신.txt` | `f9d07a4e7e9713a713ace66bb5d922ef4f125754` | Stable prompt comparison source; not represented as a byte-identical operational file |
| `bundle/main.js` | `78169b38bfcfbc116dba9ca9dde4d4e0783ad894` | The `.cc` contract in `knowledge_authoritative/01_cc_canvas.md` points to the corresponding public runtime URL |
| `bundle/main.css` | `862f844e7c47fc60b330fc5d9b5d820f45b42a04` | The `.cc` contract's public shell/style reference |
| `katex.min.css` | `73a8c0a2fc51ba349e34adc4108d9611311b5a3c` | The `.cc` contract's KaTeX asset reference |

The local Canvas contract records the runtime URLs as an external dependency; it does not claim that the local Markdown contract is a byte copy of the upstream prompt or bundle.

## Relationship assessment

Both repositories are public repositories owned under `lemos999` and their reviewed license files identify `fewweekslater (Ray You)` under CC BY-NC-SA 4.0. Their reviewed stable-prompt blob IDs differ (`c576716…` versus `f9d07a…`), and their complete Git histories had no common commit. No fork, clone, transfer, or other derivation relationship was verified from the inspected public metadata. They are therefore recorded as separate upstream provenance nodes, not as an asserted double-original or an unverified fork chain.

## Other local comparison material

- `reference_only/Ailey & Bailey_Gemini_260514.md` has local SHA-256 `1873A0A648FF84DCF55B6ED9109D045D03AC64950C6FDA28556B5DB924E3FBFB` and no matching path was verified in either reviewed commit. It remains a local comparison artifact.
- `reference_only/` is not an operational source. The authoritative modules and project instructions contain local adaptations and state contracts, not private prompt material.
- Local changes include rewritten wording, a distinct authority hierarchy, save/restore and thread-isolation contracts, normalized navigation and diagnostic rules, and manual test material. These are adaptations, not official upstream releases.

For attribution and redistribution conditions, see `LICENSE` and `NOTICE.md`.
