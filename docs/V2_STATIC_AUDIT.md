# V2 one-time static audit

## Scope

- Audit date: `2026-07-18` (Asia/Tokyo)
- Branch: `codex/v2-parity-hardening`
- Comparison base: `561ac3a`
- Purpose: one-time V2 consistency evidence
- Non-goal: reusable validator framework, Python validator, or CI system

This document records commands run against the V2 worktree. It does not prove live GPT, Canvas, image generation, web search, or private GitHub behavior.

## 1. JSON and embedded save-packet parsing

```powershell
$ErrorActionPreference = 'Stop'
Get-Content MANIFEST.json -Raw | ConvertFrom-Json | Out-Null
Get-Content baselines/state_transition_template.json -Raw | ConvertFrom-Json | Out-Null
$packet = Get-Content tests/fixtures/l01_pending_problem_save_packet.txt -Raw
$packetJson = $packet -replace '(?s)^\[AILEY_BAILEY_SAVE_PACKET_BEGIN\]\s*', '' -replace '\s*\[AILEY_BAILEY_SAVE_PACKET_END\]\s*$', ''
$packetJson | ConvertFrom-Json | Out-Null
```

Result: `PASS` — Manifest, baseline state template, and fixed L01 save packet parsed as JSON.

## 2. JSONL parsing and test counts

```powershell
$cases = @(Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })
$legacy = @($cases | Where-Object { $_.id -match '^[PRNQDXSIMTZ][0-9]{2}$' })
$added = @($cases | Where-Object { $_.id -in @('L01', 'J01') })
"total=$($cases.Count) legacy=$($legacy.Count) added=$($added.Count) critical=$(@($legacy | Where-Object critical).Count)"
```

Result: `PASS` — `total=79 legacy=77 added=2 critical=16`.

## 3. Markdown and JSONL test ID comparison

```powershell
$planIds = @(Select-String tests/EQUIVALENCE_TEST_PLAN.md -Pattern '^### ([A-Z][0-9]{2}) · ' | ForEach-Object { $_.Matches[0].Groups[1].Value })
$caseIds = @($cases.id)
$diff = @(Compare-Object $planIds $caseIds)
if ($diff.Count -ne 0) { throw "test ID mismatch: $($diff | Out-String)" }
```

Result: `PASS` — all 79 IDs matched with no duplicate or missing ID.

## 4. Existing 77-case field preservation

Z01 was explicitly required to receive concrete setup, prompt, must, and fail text. Validator labels were explicitly migrated from nonexistent Python names to manual methods. The command therefore protects identity, category, title, Critical flag, and repeat count for all 77 cases; it also requires setup/prompt/must/fail to remain byte-equivalent for the other 76 cases.

```powershell
$before = @(git show 561ac3a:tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })
$beforeById = @{}; $before | ForEach-Object { $beforeById[$_.id] = $_ }
$afterById = @{}; $cases | ForEach-Object { $afterById[$_.id] = $_ }
$protected = @('id', 'category', 'title', 'critical', 'repeat_count')
$contract = @('setup', 'prompt', 'must', 'fail')
foreach ($old in $before | Where-Object { $_.id -match '^[PRNQDXSIMTZ][0-9]{2}$' }) {
  $new = $afterById[$old.id]
  if (-not $new) { throw "missing legacy case $($old.id)" }
  foreach ($field in $protected) {
    if (($old.$field | ConvertTo-Json -Compress -Depth 20) -ne ($new.$field | ConvertTo-Json -Compress -Depth 20)) { throw "$($old.id) changed protected field $field" }
  }
  if ($old.id -ne 'Z01') {
    foreach ($field in $contract) {
      if (($old.$field | ConvertTo-Json -Compress -Depth 20) -ne ($new.$field | ConvertTo-Json -Compress -Depth 20)) { throw "$($old.id) changed contract field $field" }
    }
  }
}
```

Result: `PASS` — protected fields remained identical for 77/77 cases; setup/prompt/must/fail remained identical for 76/76 non-Z01 cases; Z01 was tightened as explicitly requested without changing its ID, category, title, Critical flag, or repeat count.

## 5. Critical Gate preservation

```powershell
$beforeCritical = @($before | Where-Object critical | ForEach-Object id | Sort-Object)
$afterCritical = @($cases | Where-Object critical | ForEach-Object id | Sort-Object)
$criticalDiff = @(Compare-Object $beforeCritical $afterCritical)
if ($beforeCritical.Count -ne 16 -or $afterCritical.Count -ne 16 -or $criticalDiff.Count -ne 0) { throw 'Critical Gate drift' }
```

Result: `PASS` — the same 16 Critical IDs were preserved.

## 6. Manual validator alignment

```powershell
$manifest = Get-Content MANIFEST.json -Raw | ConvertFrom-Json
$declared = @($manifest.manual_validation_methods | Sort-Object -Unique)
$used = @($cases.validator | Sort-Object -Unique)
if ($manifest.validators.Count -ne 0) { throw 'validators must remain empty' }
$validatorDiff = @(Compare-Object $declared $used)
if ($validatorDiff.Count -ne 0) { throw "manual validator mismatch: $($validatorDiff | Out-String)" }
```

Result: `PASS` — `validators` is empty and the five declared manual methods exactly match the five JSONL values.

## 7. Authority declarations

```powershell
$authorityFiles = @(Get-ChildItem knowledge_authoritative -File -Filter *.md)
if ($authorityFiles.Count -ne 8) { throw "authority file count=$($authorityFiles.Count)" }
foreach ($file in $authorityFiles) {
  $text = Get-Content $file.FullName -Raw
  foreach ($token in @('status: authoritative', 'authority_scope:', 'integration_rule:')) {
    if ([regex]::Matches($text, [regex]::Escape($token)).Count -ne 1) { throw "$($file.Name): $token" }
  }
}
$scopes = @($authorityFiles | ForEach-Object {
  $scopeLine = Select-String -LiteralPath $_.FullName -Pattern '^- authority_scope: ' | Select-Object -First 1
  [pscustomobject]@{ file = $_.Name; scope = $scopeLine.Line -replace '^- authority_scope: ', '' }
})
if (@($scopes.scope | Sort-Object -Unique).Count -ne 8) { throw 'duplicate authority scope' }
$routingOwners = @(Select-String knowledge_authoritative/*.md -Pattern '모드 간 우선순위는 `00_PROJECT_INSTRUCTIONS.md`|통합 라우팅과 모드 간 출력 우선순위')
if ($routingOwners.Count -ne 5) { throw "routing authority declaration count=$($routingOwners.Count)" }
```

Result: `PASS` — all eight authoritative Markdown files retain one status, scope, and integration declaration; all eight scopes are unique. The manifest owns conflict resolution, `00_PROJECT_INSTRUCTIONS.md` owns integrated routing and cross-mode output priority, and each feature file owns only its activated feature contract. The `.cc`, save/load, image, and diagnostic files consistently delegate cross-mode priority to the project instructions; no feature module independently overrides that order.

## 8. Manifest paths and counts

```powershell
$paths = @($manifest.project_instruction_path, $manifest.license_path, $manifest.notice_path, $manifest.provenance_path) + @($manifest.authoritative_files) + @($manifest.qa_only_files)
foreach ($path in $paths) { if (-not (Test-Path -LiteralPath $path)) { throw "missing Manifest path: $path" } }
if ($manifest.authoritative_file_count -ne $manifest.authoritative_files.Count) { throw 'authoritative count mismatch' }
if ($manifest.qa_only_file_count -ne $manifest.qa_only_files.Count) { throw 'QA-only count mismatch' }
if ($manifest.legacy_test_count -ne 77 -or $manifest.added_test_count -ne 2 -or $manifest.total_test_count -ne 79) { throw 'test count mismatch' }
```

Result: `PASS` — all declared paths exist; authoritative count is 8, QA-only count is 18, and test counts are 77 + 2 = 79.

## 9. Save ZIP conflict and onboarding ownership

```powershell
$zipHits = @(Select-String 00_PROJECT_INSTRUCTIONS.md,knowledge_authoritative/*.md -Pattern 'Provide Save data|ZIP File')
if ($zipHits.Count -ne 0) { throw 'save ZIP conflict remains' }
$onboardingHits = @(rg -n "첫 실질 답변 안내|새 프로젝트 학습 세션의 첫 실질 답변" 00_PROJECT_INSTRUCTIONS.md knowledge_authoritative)
if ($LASTEXITCODE -notin @(0, 1)) { throw 'rg onboarding scan failed' }
$onboardingHits
```

Result: `PASS` — no ZIP-output instruction remains; first-session onboarding ownership appears only in `00_PROJECT_INSTRUCTIONS.md`.

## 10. L01 observable-state and 21-turn checks

```powershell
$l01 = $cases | Where-Object id -eq 'L01'
if ($l01.prompt.Count -ne 21) { throw "L01 turn count=$($l01.prompt.Count)" }
if (@($l01.prompt.turn) -join ',' -ne ((1..21) -join ',')) { throw 'L01 turn sequence mismatch' }
$observableContract = "$($l01.must) $($l01.fail) $($l01.prompt.expect -join ' ')"
if ($observableContract -match 'thread_session_id|navigation_id|curriculum_id|active_source_id|active_problem_set_id') { throw 'hidden state used as success condition' }
if (($l01.prompt | Where-Object turn -in 3,4,5).input -join ',' -ne 'b,e,a') { throw 'L01 compass inputs mismatch' }
```

Result: `PASS` — L01 has sequential turns 1–21, uses the explicit `b,e,a` latest-compass sequence, and evaluates only observable state.

## 11. Provenance checksums

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath 'reference_only/Ailey & Bailey X대화형_260323.md','reference_only/Ailey & Bailey_Gemini_260514.md' | Select-Object Path,Hash
```

Result: `PASS` — hashes matched `437959EF66D17A91C882547A93AA7875D1C55E30A0EC916C37AB277084B27406` and `1873A0A648FF84DCF55B6ED9109D045D03AC64950C6FDA28556B5DB924E3FBFB` recorded in `REFERENCE_PROVENANCE.md`.

## 12. Sensitive-pattern and diff checks

```powershell
$secretHits = Get-ChildItem -Recurse -File | Where-Object { $_.FullName -notmatch '\\.git\\|docs\\V2_STATIC_AUDIT\.md$' } | Select-String -Pattern '(?i)(api[_ -]?key|password|private key|BEGIN [A-Z ]*PRIVATE KEY|github_pat_|ghp_|sk-[A-Za-z0-9]{16,})'
if ($secretHits) { throw "potential secret pattern: $($secretHits | Out-String)" }
git diff --check
if ($LASTEXITCODE -ne 0) { throw 'git diff --check failed' }
```

Result: `PASS` — no credential-like pattern was found and `git diff --check` reported no whitespace errors.

## Not executed

- Live current-GPT response comparison
- ChatGPT Canvas execution and external `.cc` asset loading
- Image-generation ordering and content checks
- Live web-search behavior
- Private GitHub authorization and repository reads

Those remain `not_evaluated` until executed on the corresponding live surfaces with captured baselines.
