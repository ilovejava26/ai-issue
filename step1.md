# ai-issue 원격 저장소 생성 및 이슈 등록 작업 정리 (step 1)

> 실행 환경: **Windows PowerShell 5.1** (`C:\workspace\ai-issue`)
> `gh` CLI는 이미 설치되어 있었고, `ilovejava26` 계정으로 로그인된 상태에서 시작했습니다.
> 로컬 저장소(`main`)와 초기 커밋 2개(`Initial commit`, `docs: add step1 command summary`)는 이미 만들어져 있었고, 원격 저장소는 아직 없는 상태였습니다.

---

## 1. 환경 및 로그인 상태 확인

```powershell
gh auth status
```

실행 결과(요약):

```text
github.com
  ✓ Logged in to github.com account ilovejava26 (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
```

추가로 현재 폴더 상태를 확인했습니다.

```powershell
Get-ChildItem -Force
git status --short --branch
git remote -v
git log --oneline -5
```

확인된 내용:

- 현재 브랜치: `main`
- 원격(`origin`): 아직 등록되지 않음
- 커밋: `8cd8aec docs: add step1 command summary`, `d407dfc Initial commit`
- 미추적 파일: `.gh-login.log` (이번 작업과 무관하여 그대로 둠)

---

## 2. 원격 저장소 존재 여부 확인

이미 같은 이름의 저장소가 있는지 먼저 확인했습니다. (중복 생성 방지)

```powershell
gh repo view ilovejava26/ai-issue --json name,visibility,url
```

실행 결과:

```text
GraphQL: Could not resolve to a Repository with the name 'ilovejava26/ai-issue'. (repository)
```

→ 저장소가 없다는 뜻이므로 새로 생성하기로 했습니다. (PowerShell에서는 stderr가 `NativeCommandError`로 표시되지만, 이는 정상적인 "없음" 응답입니다.)

---

## 3. 원격 저장소 생성 + 초기 push

```powershell
gh repo create ai-issue --public --source . --remote origin --push
```

옵션 설명:

- `ai-issue` : GitHub에 만들 저장소 이름
- `--public` : 공개(public) 저장소로 생성
- `--source .` : 현재 폴더를 로컬 소스로 지정
- `--remote origin` : 원격 이름을 `origin`으로 지정
- `--push` : 생성 직후 현재 커밋을 push

실행 결과(요약):

```text
https://github.com/ilovejava26/ai-issue
branch 'main' set up to track 'origin/main'.
To https://github.com/ilovejava26/ai-issue.git
 * [new branch]      HEAD -> main
```

> PowerShell이 git의 진행 메시지(stderr)를 빨간 `NativeCommandError`로 출력하고 종료 코드를 1로 표시했지만, 저장소 생성과 push는 정상 완료되었습니다. (아래 6번에서 검증)

생성된 저장소: <https://github.com/ilovejava26/ai-issue>

---

## 4. 이슈 생성 ("배포됨")

### 4-1. 한글 제목 인코딩 문제 회피

Windows PowerShell 5.1에서 `gh issue create --title "배포됨"`처럼 한글을 명령줄 인자로 넘기면 인코딩이 깨질 수 있습니다.
그래서 **UTF-8 JSON 파일을 만들어 `gh api --input`으로 전달**하는 방식을 사용했습니다.

이슈 내용을 담은 JSON 파일(`C:\Users\user\AppData\Local\Temp\opencode\issue.json`):

```json
{"title":"배포됨","body":""}
```

### 4-2. 이슈 생성 실행

```powershell
gh api -X POST repos/ilovejava26/ai-issue/issues `
  --input "C:\Users\user\AppData\Local\Temp\opencode\issue.json" `
  --jq '.number, .title, .html_url'
```

실행 결과:

```text
1
배포됨
https://github.com/ilovejava26/ai-issue/issues/1
```

- 이슈 번호: **#1**
- 제목: **배포됨**
- 본문: 비어 있음
- URL: <https://github.com/ilovejava26/ai-issue/issues/1>

> 참고: `--jq` 안에서 큰따옴표를 `\"`로 이스케이프하면 PowerShell이 백슬래시를 그대로 넘겨
> `failed to parse jq expression` 오류가 납니다. jq 식에는 가급적 따옴표를 쓰지 않거나,
> Git Bash에서 실행하는 것이 안전합니다.

---

## 5. (참고) Git Bash에서 실행할 경우

인코딩 문제가 없는 Git Bash에서는 아래처럼 간단히 실행할 수 있습니다.

```bash
gh repo create ai-issue --public --source . --remote origin --push
gh issue create --repo ilovejava26/ai-issue --title "배포됨" --body ""
```

---

## 6. 최종 검증

저장소 정보:

```powershell
gh repo view ilovejava26/ai-issue --json name,visibility,url,defaultBranchRef
```

실행 결과:

```json
{"defaultBranchRef":{"name":"main"},"name":"ai-issue","url":"https://github.com/ilovejava26/ai-issue","visibility":"PUBLIC"}
```

원격 주소:

```powershell
git remote -v
```

실행 결과:

```text
origin  https://github.com/ilovejava26/ai-issue.git (fetch)
origin  https://github.com/ilovejava26/ai-issue.git (push)
```

이슈 확인:

```powershell
gh issue list --repo ilovejava26/ai-issue
```

실행 결과:

```text
1  OPEN  배포됨    2026-11-06T06:31:49Z
```

---

## 최종 상태 요약

| 항목 | 결과 |
| --- | --- |
| 저장소 | <https://github.com/ilovejava26/ai-issue> (public) |
| 기본 브랜치 | `main` (`origin/main` 추적 중) |
| 원격 | `origin` → `https://github.com/ilovejava26/ai-issue.git` |
| 이슈 | [#1 배포됨](https://github.com/ilovejava26/ai-issue/issues/1) |

---

## 참고

- PowerShell에서 `gh`/`git`의 stderr 출력이 `NativeCommandError`로 보일 수 있습니다. 실제 성공 여부는 `gh repo view`, `git remote -v` 등으로 확인하는 것이 확실합니다.
- 한글 제목/본문을 명령줄 인자로 넘길 때는 UTF-8 JSON 파일 + `gh api --input` 방식을 권장합니다.
- `gh`가 인식되지 않으면 GitHub CLI 설치 경로(예: `C:\Program Files\GitHub CLI`)가 `PATH`에 있는지 확인하세요.
