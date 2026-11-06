# ai-issue 원격 저장소 생성 작업 정리 (step 1)

> 실행 환경: **Git Bash** (Windows)
> 현재 폴더(`C:\workspace\ai-issue`)는 git 저장소가 아니었고, `gh` CLI도 로그인되어 있지 않은 상태에서 시작했습니다.

---

## 1. 환경 확인

```bash
git status        # git 저장소 여부 확인 (not a git repository)
gh --version      # gh 2.103.3 확인
gh auth status    # 미로그인 확인
```

---

## 2. 브라우저 인증 ✅ 실행됨

```bash
gh auth login --hostname github.com --git-protocol https --web --scopes repo
```

- 안내되는 일회용 코드(예: `B682-42A2`)를 확인합니다.
- "Press Enter to open github.com in your browser..." 프롬프트가 나오면 **Enter**를 누릅니다.
- 브라우저에서 https://github.com/login/device 를 열고 코드를 입력해 승인합니다.

프롬프트를 자동으로 넘기고 싶다면 Git Bash에서는 `printf`를 사용합니다.

```bash
printf '\n\n' | gh auth login --hostname github.com --git-protocol https --web --scopes repo
```

인증 결과 확인:

```bash
gh auth status
```

---

## 3. 로컬 저장소 초기화 ✅ 실행됨

```bash
git init -b main
```

---

## 4. 초기 파일 생성 및 커밋 ✅ 실행됨

```bash
git add README.md
git commit -m "Initial commit"
```

---

## 5. 원격 저장소 생성 + push (인증 후 실행)

```bash
gh repo create ai-issue --public --source . --remote origin --push
```

- `--source .` : 현재 폴더를 로컬 소스로 지정
- `--remote origin` : 원격 이름을 `origin`으로 지정
- `--push` : 생성 직후 push까지 수행 (이미 커밋이 있어 그대로 올라감)

---

## 6. 검증 (push 후)

```bash
gh repo view --web        # 브라우저로 저장소 열기
git remote -v             # origin 주소 확인
git log --oneline -1      # 커밋 확인
```

---

## 참고

- PowerShell에서 실행할 경우 stderr 출력이 `NativeCommandError`로 표시될 수 있지만, Git Bash에서는 해당 오류가 나타나지 않습니다.
- Git Bash에서 `gh`가 인식되지 않으면 GitHub CLI 설치 경로(예: `C:\Program Files\GitHub CLI`)가 `PATH`에 있는지 확인하세요.
