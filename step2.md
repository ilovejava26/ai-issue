# 라벨·마일스톤·예시 이슈 등록 작업 정리 (step 2)

> 실행 환경: **Windows PowerShell 5.1** (`C:\workspace\ai-issue`)
> 저장소: <https://github.com/ilovejava26/ai-issue> (step 1에서 생성)
> 전제: 프로젝트를 **Python 통계 라이브러리**로 가정하고 2주 스프린트 분량의 예시 데이터를 등록했습니다.

---

## 0. 작업 개요

| 항목 | 개수 | 비고 |
| --- | --- | --- |
| 라벨 | 13개 | 유형 6, 영역 4, 우선순위 3 |
| 마일스톤 | 2개 | Week 1 / Week 2 (2주 스프린트) |
| 이슈 | 12개 | 기존 `#1 배포됨` 제외, `#2`~`#13` |

사용한 GitHub REST API:

| 용도 | 메서드 / 엔드포인트 |
| --- | --- |
| 라벨 조회 | `GET /repos/{owner}/{repo}/labels` |
| 라벨 생성 | `POST /repos/{owner}/{repo}/labels` |
| 마일스톤 조회 | `GET /repos/{owner}/{repo}/milestones` |
| 마일스톤 생성 | `POST /repos/{owner}/{repo}/milestones` |
| 이슈 조회 | `GET /repos/{owner}/{repo}/issues` |
| 이슈 생성 | `POST /repos/{owner}/{repo}/issues` |

---

## 1. 사전 확인

```powershell
gh --version
python --version
gh label list --repo ilovejava26/ai-issue --limit 100
gh api repos/ilovejava26/ai-issue/milestones --jq 'length'
```

확인 결과:

- `gh` 2.103.3
- Python은 설치되어 있지 않음 → PowerShell로 작업
- 기존 라벨: GitHub 기본 라벨 10개
- 기존 마일스톤: 0개

---

## 2. 한글 입력을 위한 UTF-8 처리

Windows PowerShell 5.1에서 한글을 `gh` 명령줄 인자로 직접 넘기면 인코딩이 깨질 수 있습니다.
그래서 **UTF-8 JSON 파일을 만들고 `gh api --input`으로 전달**하는 방식을 사용했습니다.

공통 함수 (`seed.ps1`에 정의):

```powershell
function Invoke-GhApi {
    param(
        [string]$Method,
        [string]$Endpoint,
        $Payload
    )
    $json = $Payload | ConvertTo-Json -Depth 20 -Compress
    $file = Join-Path $tmpDir ([guid]::NewGuid().ToString('N') + '.json')
    [IO.File]::WriteAllText($file, $json, (New-Object System.Text.UTF8Encoding($false)))
    try {
        $output = & gh api -X $Method $Endpoint --input $file 2>&1
        if ($LASTEXITCODE -ne 0) { throw ($output | Out-String) }
        return ($output | Out-String | ConvertFrom-Json)
    }
    finally {
        Remove-Item $file -Force -ErrorAction SilentlyContinue
    }
}
```

핵심 포인트:

- `ConvertTo-Json`으로 본문 생성
- `[IO.File]::WriteAllText(..., UTF8Encoding($false))`로 **BOM 없는 UTF-8** 저장
- `gh api -X <METHOD> <ENDPOINT> --input <FILE>`로 전송
- 실패 시 `$LASTEXITCODE` 확인 후 예외 발생

---

## 3. 라벨 생성

기존 라벨을 먼저 조회해 중복을 건너뛰도록 했습니다.

```powershell
$existingLabels = @(& gh api "repos/$repo/labels?per_page=100" --paginate --jq '.[].name')

foreach ($label in $labels) {
    if ($existingLabels -contains $label.name) {
        Write-Host "[skip] label: $($label.name)"
        continue
    }
    Invoke-GhApi -Method POST -Endpoint "repos/$repo/labels" -Payload $label | Out-Null
    Write-Host "[new ] label: $($label.name)"
}
```

생성한 라벨:

| 라벨 | 색상 | 설명 |
| --- | --- | --- |
| `type: feature` | `#1D76DB` | 새 기능 구현 |
| `type: bug` | `#D73A4A` | 버그 수정 |
| `type: docs` | `#0075CA` | 문서 작업 |
| `type: test` | `#5319E7` | 테스트 작성 및 보강 |
| `type: chore` | `#C5DEF5` | 빌드, CI, 설정 작업 |
| `type: perf` | `#E99695` | 성능 개선 |
| `area: descriptive` | `#0E8A16` | 기술통계 |
| `area: inference` | `#B60205` | 확률분포 및 가설검정 |
| `area: regression` | `#FBCA04` | 상관분석 및 회귀 |
| `area: data` | `#006B75` | 데이터 로딩 및 전처리 |
| `priority: high` | `#B60205` | 우선순위 높음 |
| `priority: medium` | `#FBCA04` | 우선순위 보통 |
| `priority: low` | `#0E8A16` | 우선순위 낮음 |

라벨 하나를 직접 만드는 명령은 다음과 같습니다.

```powershell
Invoke-GhApi -Method POST -Endpoint "repos/ilovejava26/ai-issue/labels" -Payload @{
    name        = 'type: feature'
    color       = '1D76DB'
    description = '새 기능 구현'
}
```

---

## 4. 마일스톤 생성

2주 스프린트를 주차별 마일스톤 2개로 나눴습니다.

```powershell
$milestones = @(
    @{
        key         = 'w1'
        title       = 'Sprint 1 · Week 1 — 기반 구축'
        description = '패키지 기반, 데이터 로딩, 기술통계, 테스트/CI를 완성해 분석 기능의 토대를 만든다.'
        due_on      = '2026-11-13T23:59:59Z'
    },
    @{
        key         = 'w2'
        title       = 'Sprint 1 · Week 2 — 분석 기능 확장'
        description = '확률분포, 가설검정, 상관/회귀, 문서화를 완성하고 0.1.0 릴리스 후보를 만든다.'
        due_on      = '2026-11-20T23:59:59Z'
    }
)
```

기존 마일스톤을 조회한 뒤 없는 것만 생성:

```powershell
$existingMilestones = @{}
$msRaw = & gh api "repos/$repo/milestones?state=all&per_page=100"
if ($msRaw) {
    $parsedMilestones = $msRaw | ConvertFrom-Json
    if ($parsedMilestones) {
        foreach ($ms in @($parsedMilestones)) {
            if ($ms.title) { $existingMilestones[$ms.title] = $ms.number }
        }
    }
}

foreach ($m in $milestones) {
    if ($existingMilestones.ContainsKey($m.title)) {
        $milestoneNumbers[$m.key] = $existingMilestones[$m.title]
        continue
    }
    $created = Invoke-GhApi -Method POST -Endpoint "repos/$repo/milestones" -Payload @{
        title       = $m.title
        description = $m.description
        due_on      = $m.due_on
    }
    $milestoneNumbers[$m.key] = $created.number
}
```

생성 결과:

| # | 마일스톤 | 마감 |
| --- | --- | --- |
| 1 | Sprint 1 · Week 1 — 기반 구축 | 2026-11-13 |
| 2 | Sprint 1 · Week 2 — 분석 기능 확장 | 2026-11-20 |

> `due_on`은 날짜만 저장되어 GitHub에서는 `00:00:00Z`로 표시됩니다.

---

## 5. 이슈 생성

기존 이슈 제목을 조회해 중복 생성을 방지했습니다.

```powershell
$existingIssues = @(& gh api "repos/$repo/issues?state=all&per_page=100" --paginate --jq '.[].title')

foreach ($issue in $issues) {
    if ($existingIssues -contains $issue.title) {
        Write-Host "[skip] issue: $($issue.title)"
        continue
    }
    $payload = @{
        title     = $issue.title
        body      = $issue.body
        labels    = $issue.labels
        milestone = $milestoneNumbers[$issue.milestone]
    }
    $created = Invoke-GhApi -Method POST -Endpoint "repos/$repo/issues" -Payload $payload
    Write-Host "[new ] issue #$($created.number): $($issue.title)"
    Start-Sleep -Milliseconds 300
}
```

이슈 본문은 PowerShell here-string(`@' ... '@`)으로 작성했습니다.

```powershell
@{
    title     = 'feat: 기술통계 API 구현 (mean, median, variance, std)'
    labels    = @('type: feature', 'area: descriptive', 'priority: high', 'good first issue')
    milestone = 'w1'
    body      = @'
## 배경
라이브러리의 가장 기본이 되는 기술통계 API를 제공합니다.

## 작업 내용
- [ ] `mean`, `median`, `variance`, `std` 구현
- [ ] 표본/모분산 구분을 위한 `ddof` 옵션 지원
- [ ] `NaN` 처리 정책 결정 및 문서화
- [ ] 입력 검증(빈 배열, 1개 값, 상수 배열)

## 완료 조건
- SciPy/NumPy 결과와 오차 범위 내에서 일치한다
- 경계값에 대한 테스트가 포함된다
- 공개 API에 타입 힌트와 docstring이 작성된다
'@
}
```

생성된 이슈:

| # | 제목 | 마일스톤 |
| --- | --- | --- |
| 2 | chore: 패키지 구조와 개발 환경 구성 | Week 1 |
| 3 | feat: 기술통계 API 구현 (mean, median, variance, std) | Week 1 |
| 4 | feat: CSV 데이터 로더와 결측치 처리 | Week 1 |
| 5 | test: 기술통계 함수 단위 테스트 및 SciPy 대조 검증 | Week 1 |
| 6 | chore: GitHub Actions CI 구축 (ruff + mypy + pytest) | Week 1 |
| 7 | feat: 정규분포 PDF/CDF와 난수 샘플링 구현 | Week 2 |
| 8 | feat: t-검정 구현 (one-sample, two-sample) | Week 2 |
| 9 | feat: Pearson/Spearman 상관계수 계산 | Week 2 |
| 10 | feat: 단순 선형 회귀 구현 | Week 2 |
| 11 | perf: 대용량 데이터 기술통계 벡터화 및 메모리 최적화 | Week 2 |
| 12 | fix: 분산 계산의 부동소수점 오차 개선 (Welford) | Week 2 |
| 13 | docs: API 레퍼런스와 2주차 튜토리얼 작성 | Week 2 |

---

## 6. 스크립트 실행

작성한 `seed.ps1`을 PowerShell 5.1이 UTF-8로 읽도록 **BOM을 추가**한 뒤 실행했습니다.

```powershell
$p = 'C:\Users\user\AppData\Local\Temp\opencode\seed.ps1'
$c = [IO.File]::ReadAllText($p, [Text.Encoding]::UTF8)
[IO.File]::WriteAllText($p, $c, (New-Object Text.UTF8Encoding($true)))

powershell -NoProfile -ExecutionPolicy Bypass -File $p
```

실행 결과(요약):

```text
[new ] label: type: feature
...
[new ] milestone: Sprint 1 · Week 1 — 기반 구축 (#1)
[new ] milestone: Sprint 1 · Week 2 — 분석 기능 확장 (#2)
[new ] issue #2: chore: 패키지 구조와 개발 환경 구성
...
[new ] issue #13: docs: API 레퍼런스와 2주차 튜토리얼 작성

DONE
labels=13 milestones=2 issues=12
```

> `seed.ps1`은 멱등성을 고려해 라벨·마일스톤·이슈의 기존 항목을 확인하고 건너뜁니다.
> 재실행하면 `[skip]`만 출력되고 중복 생성되지 않습니다.

---

## 7. 트러블슈팅

### 7-1. 마일스톤이 없을 때 `ConvertFrom-Json` 오류

처음에는 아래처럼 작성했지만, 마일스톤이 0개일 때 `$null` 인덱싱 오류가 발생했습니다.

```powershell
# 실패한 코드
$msRaw = & gh api "repos/$repo/milestones?state=all&per_page=100"
if ($msRaw) {
    $msRaw | ConvertFrom-Json | ForEach-Object {
        $existingMilestones[$_.title] = $_.number
    }
}
```

수정: `ConvertFrom-Json` 결과를 변수에 담고, 값이 있을 때만 순회합니다.

```powershell
$parsedMilestones = $msRaw | ConvertFrom-Json
if ($parsedMilestones) {
    foreach ($ms in @($parsedMilestones)) {
        if ($ms.title) { $existingMilestones[$ms.title] = $ms.number }
    }
}
```

### 7-2. `jq` 표현식의 따옴표가 사라지는 문제

PowerShell이 네이티브 인자에서 큰따옴표를 제거해 아래 명령이 실패했습니다.

```powershell
# 실패: join(,) 로 전달되어 jq 파싱 오류
gh issue list --repo ilovejava26/ai-issue `
  --json number,title,labels,milestone `
  --jq '.[] | [.number, .title, (.labels|map(.name)|join(",")), .milestone.title] | @tsv'
```

수정: jq 식 안에서 큰따옴표를 쓰지 않고 `tostring`을 사용합니다.

```powershell
gh issue list --repo ilovejava26/ai-issue --limit 20 `
  --json number,title,labels,milestone `
  --jq '.[] | [.number, .title, (.labels|map(.name)|tostring), .milestone.title] | @tsv'
```

### 7-3. PowerShell의 `NativeCommandError`

`gh`/`git`이 stderr로 진행 메시지를 출력하면 PowerShell이 빨간 `NativeCommandError`로 표시합니다.
실제 성공 여부는 종료 코드나 아래 검증 명령으로 확인하는 것이 안전합니다.

### 7-4. UTF-8 BOM

`write` 도구로 만든 `.ps1`은 BOM 없는 UTF-8이라 PowerShell 5.1이 한글을 잘못 읽습니다.
`seed.ps1` 실행 전에 BOM을 추가했습니다.

```powershell
$c = [IO.File]::ReadAllText($p, [Text.Encoding]::UTF8)
[IO.File]::WriteAllText($p, $c, (New-Object Text.UTF8Encoding($true)))
```

---

## 8. 검증 명령과 결과

마일스톤:

```powershell
gh api repos/ilovejava26/ai-issue/milestones \
  --jq '.[] | [.number, .title, .due_on, .open_issues] | @tsv'
```

```text
1  Sprint 1 · Week 1 — 기반 구축      2026-11-13T00:00:00Z  5
2  Sprint 1 · Week 2 — 분석 기능 확장  2026-11-20T00:00:00Z  7
```

이슈:

```powershell
gh issue list --repo ilovejava26/ai-issue --limit 20 \
  --json number,title,labels,milestone \
  --jq '.[] | [.number, .title, (.labels|map(.name)|tostring), .milestone.title] | @tsv'
```

```text
13  docs: API 레퍼런스와 2주차 튜토리얼 작성  ["type: docs","priority: medium"]  Sprint 1 · Week 2 — 분석 기능 확장
...
2   chore: 패키지 구조와 개발 환경 구성       ["type: chore","priority: high"]   Sprint 1 · Week 1 — 기반 구축
1   배포됨
```

라벨:

```powershell
gh label list --repo ilovejava26/ai-issue --limit 100
```

---

## 9. 최종 상태

- 라벨 13개 추가 (기존 기본 라벨 유지)
- 마일스톤 2개 생성 (Week 1: 5개 이슈, Week 2: 7개 이슈)
- 예시 이슈 12개 생성 (`#2`~`#13`)
- 기존 이슈 `#1 배포됨`은 라벨/마일스톤 없이 그대로 유지

원격 저장소: <https://github.com/ilovejava26/ai-issue/issues>
