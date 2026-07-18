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

## 3차 수정 감사 (2026-07-18)

- Audit run: `2026-07-18 22:04:25 +09:00` (`Asia/Seoul`)
- Audit HEAD: `88d22d4c4512edb510d121aaeb44eec65ead0194`
- Scope: operational prompt/authority changes already pushed in the preceding commits, plus the L01 transfer-contract changes in the working tree.
- Working-tree scope at audit time: only `tests/cases/equivalence_cases.jsonl` and `tests/EQUIVALENCE_TEST_PLAN.md` were intentionally modified; no unrelated paths were present.
- The preceding V2 audit sections above are historical records and were not overwritten.

### Commands executed

```powershell
$manifest = Get-Content MANIFEST.json -Raw | ConvertFrom-Json
$cases = @(Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })
$packet = Get-Content tests/fixtures/l01_pending_problem_save_packet.txt -Raw
$packetJson = $packet -replace '(?s)^\[AILEY_BAILEY_SAVE_PACKET_BEGIN\]\s*', '' -replace '\s*\[AILEY_BAILEY_SAVE_PACKET_END\]\s*$', ''
$packetJson | ConvertFrom-Json | Out-Null
if ($cases.Count -ne 79) { throw "total=$($cases.Count)" }
$legacy = @($cases | Where-Object { $_.id -match '^[PRNQDXSIMTZ][0-9]{2}$' })
if ($legacy.Count -ne 77 -or @($legacy | Where-Object critical).Count -ne 16) { throw 'legacy/Critical drift' }
$before = @(git show 88d22d4:tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })
$after = @(Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })
foreach ($old in $before | Where-Object { $_.id -match '^[PRNQDXSIMTZ][0-9]{2}$' }) {
  $new = $after | Where-Object id -eq $old.id
  foreach ($field in @('id','category','title','critical','repeat_count','setup','prompt','must','fail')) {
    if (($old.$field | ConvertTo-Json -Compress -Depth 30) -ne ($new.$field | ConvertTo-Json -Compress -Depth 30)) { throw "$($old.id) changed $field" }
  }
}
$l01 = $cases | Where-Object id -eq 'L01'
$t8 = $l01.prompt | Where-Object turn -eq 8
if ($l01.prompt.Count -ne 21 -or (@($l01.prompt.turn) -join ',') -ne ((1..21) -join ',') -or $l01.prompt[6].input -ne '저장') { throw 'L01 sequence' }
if ($t8.transfer.source_thread -ne 'Thread A' -or $t8.transfer.source_turn -ne 7 -or $t8.transfer.destination_thread -ne 'Thread B' -or $t8.transfer.extraction.transform -ne 'none') { throw 'L01 transfer' }
$l01line = Get-Content tests/cases/equivalence_cases.jsonl | Where-Object { $_ -match '"id":"L01"' }
if ($l01line -match '"schema_version"|"packet_type"|"current_state"|"answer_submission_format"|l01_pending_problem_save_packet') { throw 'L01 packet duplication' }
if (@(Get-ChildItem knowledge_authoritative -File -Filter *.md).Count -ne 8) { throw 'authority file count' }
$used = @($cases.validator | Sort-Object -Unique); $declared = @($manifest.manual_validation_methods | Sort-Object -Unique)
if (@(Compare-Object $declared $used).Count -ne 0) { throw 'manual validator mismatch' }
git diff --check
$secretHits = Get-ChildItem -Recurse -File | Where-Object { $_.FullName -notmatch '\\.git\\|docs\\V2_STATIC_AUDIT\.md$' } | Select-String -Pattern '(?i)(api[_ -]?key|password|private key|BEGIN [A-Z ]*PRIVATE KEY|github_pat_|ghp_|sk-[A-Za-z0-9]{16,})'
if ($secretHits) { throw 'sensitive pattern' }
```

Result: `PASS` — `JSON_AND_FIXTURE`, `COUNT_AND_PROTECTED_FIELDS`, `L01_TRANSFER`, `AUTHORITY_AND_MANIFEST`, and `SENSITIVE_AND_DIFF` all passed. The L01 setup now requires Turn 7's actual model response; the fixed fixture remains available only for standalone restore/drift coverage. No legacy protected field or Critical Gate changed.

The following remain `not_evaluated`: live current-GPT response comparison, Canvas runtime execution, image-generation ordering/content, live web behavior, and private GitHub authorization/runtime reads.

## 구간 A 보완 감사 (2026-07-18)

- Audit run: `2026-07-18 22:57:11 +09:00` (`Asia/Seoul`)
- Audit HEAD before this follow-up commit: `8fd894a0b9039721ec70664ece61fe23a3414718`
- Scope: `05_deep_learning_method.md` 일반화, 프로젝트 지침 절 참조 수정, 그리고 이 보완 감사 계약
- 기존 3차 감사 절과 과거 결과는 덮어쓰지 않았다.

### 실제 실행한 보완 검사

```powershell
$ErrorActionPreference = 'Stop'
$manifest = Get-Content MANIFEST.json -Raw | ConvertFrom-Json
$cases = @(Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json })

$ids = @($cases.id)
if ($ids.Count -ne @($ids | Sort-Object -Unique).Count) { throw 'duplicate test id' }
$planIds = @(Select-String tests/EQUIVALENCE_TEST_PLAN.md -Pattern '^### ([A-Z][0-9]{2}) · ' | ForEach-Object { $_.Matches[0].Groups[1].Value })
if (@(Compare-Object $planIds $ids).Count -ne 0) { throw 'test catalog mismatch' }
$legacy = @($cases | Where-Object { $_.id -match '^[PRNQDXSIMTZ][0-9]{2}$' })
if ($cases.Count -ne 79 -or $legacy.Count -ne 77 -or @($legacy | Where-Object critical).Count -ne 16) { throw 'test count drift' }
if (@($cases | Where-Object id -eq 'L01').Count -ne 1 -or @($cases | Where-Object id -eq 'J01').Count -ne 1) { throw 'L01/J01 identity drift' }
$used = @($cases.validator | Sort-Object -Unique)
$declared = @($manifest.manual_validation_methods | Sort-Object -Unique)
if (@(Compare-Object $declared $used).Count -ne 0 -or $manifest.validators.Count -ne 0) { throw 'manual validator mismatch' }

$authorityFiles = @(Get-ChildItem knowledge_authoritative -File -Filter *.md)
if ($authorityFiles.Count -ne 8) { throw 'authority file count' }
$scopes = foreach ($file in $authorityFiles) {
  $text = Get-Content $file.FullName -Raw
  foreach ($token in @('status: authoritative', 'authority_scope:', 'integration_rule:')) {
    if ([regex]::Matches($text, [regex]::Escape($token)).Count -ne 1) { throw "$($file.Name): $token" }
  }
  $scope = (Select-String -LiteralPath $file.FullName -Pattern '^- authority_scope: ' | Select-Object -First 1).Line -replace '^- authority_scope: ', ''
  if ([string]::IsNullOrWhiteSpace($scope)) { throw "$($file.Name): empty authority scope" }
  $scope
}
if (@($scopes | Sort-Object -Unique).Count -ne 8) { throw 'duplicate authority scope' }

$operational = @('00_PROJECT_INSTRUCTIONS.md') + @($authorityFiles | ForEach-Object FullName)
$specificDependencyHits = Select-String -LiteralPath $operational -Pattern '(?i)Dwarkesh|interview-prep|interview theater|archive machinery|archive paths'
if ($specificDependencyHits) { throw "specific deep-learning dependency remains: $($specificDependencyHits | Out-String)" }
if (Select-String -LiteralPath 00_PROJECT_INSTRUCTIONS.md -Pattern '\[0\. 권한과 우선순위\]') { throw 'stale section reference' }
if (-not (Select-String -LiteralPath 00_PROJECT_INSTRUCTIONS.md -Pattern '\[0\. 적용 순서와 실행 경계\]')) { throw 'current section reference missing' }

$l01 = $cases | Where-Object id -eq 'L01'
$t8 = $l01.prompt | Where-Object turn -eq 8
if ($l01.prompt.Count -ne 21 -or (@($l01.prompt.turn) -join ',') -ne ((1..21) -join ',') -or $l01.prompt[6].input -ne '저장') { throw 'L01 sequence' }
if ($t8.transfer.source_thread -ne 'Thread A' -or $t8.transfer.source_turn -ne 7 -or $t8.transfer.destination_thread -ne 'Thread B' -or $t8.transfer.destination -ne '첫 메시지' -or $t8.transfer.extraction.transform -ne 'none') { throw 'L01 transfer' }
if ($t8.transfer.extraction.start_marker -ne '[AILEY_BAILEY_SAVE_PACKET_BEGIN]' -or $t8.transfer.extraction.end_marker -ne '[AILEY_BAILEY_SAVE_PACKET_END]') { throw 'L01 markers' }
$l01line = Get-Content tests/cases/equivalence_cases.jsonl | Where-Object { $_ -match '"id":"L01"' }
if ($l01line -match '"schema_version"|"packet_type"|"current_state"|"answer_submission_format"|l01_pending_problem_save_packet') { throw 'L01 packet duplication' }
if ($t8.transfer.fixture_use -notmatch 'L01에서는 고정 fixture를 입력으로 사용하지 않음') { throw 'L01 fixture rule' }

git diff --check
if ($LASTEXITCODE -ne 0) { throw 'git diff --check failed' }
$secretHits = Get-ChildItem -Recurse -File | Where-Object { $_.FullName -notmatch '\\.git\\|docs\\V2_STATIC_AUDIT\.md$' } | Select-String -Pattern '(?i)(api[_ -]?key|password|private key|BEGIN [A-Z ]*PRIVATE KEY|github_pat_|ghp_|sk-[A-Za-z0-9]{16,})'
if ($secretHits) { throw "potential secret pattern: $($secretHits | Out-String)" }
$branch = (git branch --show-current).Trim()
if ($branch -ne 'codex/v2-parity-hardening') { throw "unexpected branch=$branch" }
$head = (git rev-parse HEAD).Trim()
$remoteHead = (git rev-parse origin/codex/v2-parity-hardening).Trim()
if ($head -ne $remoteHead) { throw "local/remote mismatch: $head / $remoteHead" }
```

### 결과

- PASS — JSON/JSONL 파싱, 전체 ID 유일성, 79개 카탈로그와 77개 legacy/16개 Critical 보존
- PASS — `L01`·`J01`의 단일 존재, 다섯 manual validator 일치
- PASS — authoritative 8개 파일, 각 `status`·`authority_scope`·`integration_rule` 정확히 1회, 비어 있지 않은 scope 8개 고유
- PASS — `05_deep_learning_method.md`와 다른 운영 파일에서 Dwarkesh/interview-prep 전용 의존성 및 오래된 절 참조 제거
- PASS — L01 21턴, Turn 7의 정확한 `저장`, Turn 8의 실제 marker·Thread A→Thread B transfer 계약, fixture 비사용 계약
- PASS — `git diff --check`와 민감정보 검사
- PASS — 감사 시점의 브랜치가 `codex/v2-parity-hardening`이고 로컬 HEAD와 원격 HEAD가 `8fd894a0b9039721ec70664ece61fe23a3414718`로 일치

다음 항목은 이 보완 감사에서도 `not_evaluated`로 유지한다.

- live current-GPT 응답 비교 및 실제 21턴 L01 수동 실행
- ChatGPT Canvas/외부 `.cc` 런타임
- 이미지 생성 순서와 결과
- live 웹 검색 동작
- private GitHub 런타임 권한과 실제 저장소 동작
