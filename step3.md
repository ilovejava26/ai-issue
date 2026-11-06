# docs 문서를 이슈로 등록하고 커밋·푸시한 작업 정리 (step 3)

> 실행 환경: **Windows PowerShell 5.1 + Git Bash**
> 저장소: <https://github.com/ilovejava26/ai-issue>
> 대상: `docs/til.md`, `docs/retrospective.md`, `docs/postmortem.md`
> 결과: 이슈 **#14~#16** 생성, `docs/`와 `step2.md`, `step3.md` 커밋·push

---

## 1. 작업 개요

| 문서 | 이슈 | 제목 | 라벨 |
| --- | --- | --- | --- |
| `docs/til.md` | #14 | docs: 14일 TIL 정리 | `type: docs` |
| `docs/retrospective.md` | #15 | docs: 2주 스프린트 회고 | `type: docs` |
| `docs/postmortem.md` | #16 | docs: Spearman 결측치 처리 부검 (가상 시나리오) | `type: docs` |

이슈 본문은 각 문서의 **전체 마크다운**을 그대로 사용했습니다.
이후 TIL 형식을 `What → Why → How`에서 `Why → What → How`로 수정하면서 #14의 본문도 다시 동기화했습니다.

---

## 2. 사전 확인

```powershell
gh auth status
gh issue list --repo ilovejava26/ai-issue --limit 20
Get-ChildItem docs
```

확인 결과:

- `ilovejava26` 계정으로 로그인됨
- 기존 이슈: #1~#13
- `docs/`에 `til.md`, `retrospective.md`, `postmortem.md` 존재

---

## 3. 이슈 생성 명령

GitHub CLI에는 `--body-file` 옵션이 있어 마크다운 파일을 본문으로 그대로 전달할 수 있습니다.

```bash
gh issue create \
  --title "docs: 14일 TIL 정리" \
  --body-file docs/til.md \
  --label "type: docs"

gh issue create \
  --title "docs: 2주 스프린트 회고" \
  --body-file docs/retrospective.md \
  --label "type: docs"

gh issue create \
  --title "docs: Spearman 결측치 처리 부검 (가상 시나리오)" \
  --body-file docs/postmortem.md \
  --label "type: docs"
```

> `--body-file`은 파일을 UTF-8로 읽어 그대로 전송하므로, 한글 본문을 명령줄 인자로 넘길 때보다 안전합니다.

---

## 4. Git Bash PATH 설정

이 환경에서는 Git Bash에서 `gh`가 PATH에 잡히지 않았습니다.

```bash
gh --version
# bash: gh: command not found
```

GitHub CLI 설치 경로를 PATH에 추가합니다.

```bash
export PATH="/c/Program Files/GitHub CLI:$PATH"
gh --version
```

> PowerShell에서 `bash -lc 'export PATH="/c/Program Files/GitHub CLI:$PATH"; ...'`처럼
> 한 줄로 넘기면 PowerShell이 내부 큰따옴표를 제거해 경로가 분리됩니다.
> 그래서 아래처럼 **`.sh` 파일로 작성한 뒤 실행**했습니다.

---

## 5. 중복 방지 스크립트

`docs/` 문서를 이슈로 만들 때 이미 같은 제목의 이슈가 있으면 건너뛰도록 했습니다.

`C:\Users\user\AppData\Local\Temp\opencode\create_doc_issues.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

export PATH="/c/Program Files/GitHub CLI:$PATH"
cd /c/workspace/ai-issue

existing="$(gh issue list --state all --limit 100 --json title --jq '.[].title')"

create_issue() {
  local title="$1"
  local body_file="$2"

  if grep -Fxq "$title" <<< "$existing"; then
    echo "[skip] $title"
    return
  fi

  gh issue create \
    --title "$title" \
    --body-file "$body_file" \
    --label "type: docs"
}

create_issue "docs: 14일 TIL 정리" "docs/til.md"
create_issue "docs: 2주 스프린트 회고" "docs/retrospective.md"
create_issue "docs: Spearman 결측치 처리 부검 (가상 시나리오)" "docs/postmortem.md"
```

실행:

```powershell
& 'C:\Program Files\Git\bin\bash.exe' '/c/Users/user/AppData/Local/Temp/opencode/create_doc_issues.sh'
```

실행 결과:

```text
https://github.com/ilovejava26/ai-issue/issues/14
https://github.com/ilovejava26/ai-issue/issues/15
https://github.com/ilovejava26/ai-issue/issues/16
```

---

## 6. 검증

이슈 목록:

```powershell
gh issue list --repo ilovejava26/ai-issue --limit 5
```

```text
16  OPEN  docs: Spearman 결측치 처리 부검 (가상 시나리오)  type: docs
15  OPEN  docs: 2주 스프린트 회고                         type: docs
14  OPEN  docs: 14일 TIL 정리                            type: docs
13  OPEN  docs: API 레퍼런스와 2주차 튜토리얼 작성          type: docs, priority: medium
12  OPEN  fix: 분산 계산의 부동소수점 오차 개선 (Welford)   type: bug, area: descriptive, priority: medium
```

본문 확인 (PowerShell 콘솔에서 한글이 깨져 보일 수 있으므로 파일로 저장해 확인):

```powershell
cmd /c "gh api repos/ilovejava26/ai-issue/issues/14 --jq .body > C:\Users\user\AppData\Local\Temp\opencode\body14.md"
```

`body14.md` 첫 부분:

```markdown
# 14일 TIL — Python 통계 라이브러리 2주 스프린트

> 기간: 2026-11-09 ~ 2026-11-22
> 전제: 마일스톤 **Sprint 1 · Week 1**(#2~#6), **Sprint 1 · Week 2**(#7~#13)의 모든 이슈가 계획대로 완료되었다.
```

> `gh issue view`의 출력이 PowerShell에서 깨져 보이는 것은 콘솔 표시 문제이며, 실제 이슈 본문은 UTF-8로 정상 저장됩니다.

---

## 7. 문서 수정 후 이슈 본문 동기화

TIL의 형식을 **Why → What → How**로 바로잡은 뒤, 기존 이슈의 본문도 수정된 문서 내용으로 갱신했습니다.

```bash
gh issue edit 14 --body-file docs/til.md
gh issue edit 15 --body-file docs/retrospective.md
gh issue edit 16 --body-file docs/postmortem.md
```

실행 결과:

```text
https://github.com/ilovejava26/ai-issue/issues/14
https://github.com/ilovejava26/ai-issue/issues/15
https://github.com/ilovejava26/ai-issue/issues/16
```

검증: #14 본문의 Day 1이 `Why → What → How` 순서인지 확인했습니다.

```powershell
cmd /c "gh api repos/ilovejava26/ai-issue/issues/14 --jq .body > C:\Users\user\AppData\Local\Temp\opencode\body14_v2.md"
```

```markdown
- **Why**: 통계 라이브러리는 결과의 재현성이 핵심이다. 개발 환경이 사람마다 다르면 수치 검증 자체가 신뢰를 잃는다.
- **What**: `src/` 레이아웃, `pyproject.toml`, `ruff`/`mypy`/`pytest` 설정, 의존성 lock 파일을 구성했다.
- **How**: 버전을 고정하고, README의 설치 절차만으로 새 환경이 만들어지는지 매번 확인한다.
```

---

## 8. 커밋 및 push

### 8-1. 최초 문서 등록

변경 사항 확인:

```powershell
git status --short
```

```text
?? docs/
?? step2.md
?? step3.md
```

`docs/`와 `step2.md`, `step3.md`만 스테이징했습니다. (`.gh-login.log`는 제외)

```powershell
git add docs step2.md step3.md
git status --short
git commit -m "docs: add sprint records and issue documentation"
git push origin main
```

커밋 내용:

- `docs/til.md` — 14일 TIL
- `docs/retrospective.md` — Sprint 1/2 회고
- `docs/postmortem.md` — 가상 장애 부검
- `step2.md` — 라벨·마일스톤·이슈 등록 명령 정리
- `step3.md` — 문서를 이슈로 등록한 명령 정리

### 8-2. TIL 형식 수정

TIL을 `Why → What → How` 순서로 고친 뒤 수정 사항을 다시 커밋했습니다.

```powershell
git add docs/til.md step3.md
git commit -m "docs: align TIL with Why-What-How and sync issues"
git push origin main
```

---

## 9. 트러블슈팅

### 9-1. Git Bash에서 `gh: command not found`

```bash
export PATH="/c/Program Files/GitHub CLI:$PATH"
```

### 9-2. PowerShell에서 Git Bash 명령의 따옴표가 사라짐

```powershell
# 실패: 내부 큰따옴표가 제거되어 경로가 분리됨
& 'C:\Program Files\Git\bin\bash.exe' -lc 'export PATH="/c/Program Files/GitHub CLI:$PATH"; gh --version'
```

해결: 명령을 `.sh` 파일로 작성하고 파일을 실행합니다.

```powershell
& 'C:\Program Files\Git\bin\bash.exe' '/c/Users/user/AppData/Local/Temp/opencode/create_doc_issues.sh'
```

### 9-3. PowerShell에서 한글 제목이 깨질 때

`gh issue create --title "한글"` 대신, UTF-8 JSON 파일과 `gh api --input`을 사용합니다. (step 2 방식)

```powershell
$payload = @{
    title  = 'docs: 14일 TIL 정리'
    body   = [IO.File]::ReadAllText('docs/til.md', [Text.Encoding]::UTF8)
    labels = @('type: docs')
}
$json = $payload | ConvertTo-Json -Depth 10 -Compress
[IO.File]::WriteAllText('issue.json', $json, (New-Object Text.UTF8Encoding($false)))
gh api -X POST repos/ilovejava26/ai-issue/issues --input issue.json
```

---

## 10. 최종 상태

| 항목 | 결과 |
| --- | --- |
| 이슈 #14 | <https://github.com/ilovejava26/ai-issue/issues/14> |
| 이슈 #15 | <https://github.com/ilovejava26/ai-issue/issues/15> |
| 이슈 #16 | <https://github.com/ilovejava26/ai-issue/issues/16> |
| 라벨 | `type: docs` |
| 커밋 대상 | `docs/`, `step2.md`, `step3.md` |
| 제외 | `.gh-login.log` |
