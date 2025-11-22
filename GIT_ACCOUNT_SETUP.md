# 프로젝트별 Git 계정 관리 가이드

여러 Git 계정(예: 개인 계정, 회사 계정)을 사용하는 환경에서 프로젝트별로 다른 계정을 사용하는 방법을 안내합니다.

## 목차

- [문제 상황](#문제-상황)
- [해결 방법 개요](#해결-방법-개요)
- [SSH 키 기반 설정 (권장)](#ssh-키-기반-설정-권장)
- [다른 프로젝트에 적용하기](#다른-프로젝트에-적용하기)
- [테스트 및 확인](#테스트-및-확인)
- [문제 해결](#문제-해결)
- [참고: Personal Access Token 사용법](#참고-personal-access-token-사용법)

---

## 문제 상황

여러 Git 계정을 사용할 때 다음과 같은 문제가 발생할 수 있습니다:

```
remote: Permission to userA/repo.git denied to userB.
fatal: unable to access 'https://github.com/userA/repo.git/': The requested URL returned error: 403
```

또는

```
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/user/repo.git/'
```

이는 현재 인증된 계정이 해당 저장소에 접근 권한이 없거나, 잘못된 계정으로 인증되어 있을 때 발생합니다.

---

## 해결 방법 개요

프로젝트별로 다른 Git 계정을 사용하는 방법은 크게 두 가지가 있습니다:

1. **SSH 키 기반 방법 (권장)**: 각 계정마다 별도의 SSH 키를 생성하고, SSH config를 통해 프로젝트별로 다른 키를 사용
2. **Personal Access Token 방법**: HTTPS를 사용하되, 각 프로젝트마다 다른 토큰 사용

이 가이드는 **SSH 키 기반 방법**을 상세히 설명합니다.

---

## SSH 키 기반 설정 (권장)

### 1단계: SSH 키 생성

각 Git 계정마다 별도의 SSH 키를 생성합니다.

```bash
# 계정별로 다른 키 이름 사용
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_계정명
```

**예시:**
```bash
# webjun7979 계정용 키 생성
ssh-keygen -t ed25519 -C "webjun7979@github" -f ~/.ssh/id_ed25519_webjun7979
```

**설명:**
- `-t ed25519`: Ed25519 알고리즘 사용 (RSA보다 안전하고 빠름)
- `-C "comment"`: 키에 대한 주석 (이메일 주소 등)
- `-f ~/.ssh/id_ed25519_계정명`: 키 파일 경로 및 이름
- 비밀번호 입력을 건너뛰려면 `-N ""` 옵션 추가 (보안상 권장하지 않음)

**생성되는 파일:**
- `~/.ssh/id_ed25519_webjun7979`: 개인 키 (절대 공유하지 마세요!)
- `~/.ssh/id_ed25519_webjun7979.pub`: 공개 키 (GitHub에 등록)

### 2단계: 공개 키 확인

생성된 공개 키를 확인합니다.

```bash
cat ~/.ssh/id_ed25519_webjun7979.pub
```

출력 예시:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG2Pp3vbPJjuFvqJONy+qGLkWZOg3Ept47l6V1bLFfab webjun7979@github
```

### 3단계: GitHub에 SSH 키 추가

1. GitHub에 로그인
2. 우측 상단 프로필 아이콘 클릭 → **Settings**
3. 좌측 메뉴에서 **SSH and GPG keys** 클릭
4. **New SSH key** 버튼 클릭
5. 다음 정보 입력:
   - **Title**: 키를 식별할 수 있는 이름 (예: "MacBook - webjun7979")
   - **Key**: 2단계에서 확인한 공개 키 전체를 복사하여 붙여넣기
6. **Add SSH key** 클릭

### 4단계: SSH Config 파일 설정

`~/.ssh/config` 파일을 편집하여 각 계정별로 다른 SSH 키를 사용하도록 설정합니다.

```bash
# SSH config 파일 열기 (없으면 자동 생성됨)
nano ~/.ssh/config
# 또는
vim ~/.ssh/config
```

다음 형식으로 추가:

```
Host github.com-계정명
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_계정명
    IdentitiesOnly yes
```

**예시:**
```
Host github.com-webjun7979
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_webjun7979
    IdentitiesOnly yes
```

**설명:**
- `Host github.com-webjun7979`: SSH 연결 시 사용할 호스트 별칭
- `HostName github.com`: 실제 GitHub 호스트
- `User git`: Git 사용자명 (항상 `git`)
- `IdentityFile`: 사용할 SSH 키 경로
- `IdentitiesOnly yes`: 지정된 키만 사용 (다른 키 시도 안 함)

**여러 계정 설정 예시:**
```
# 개인 계정
Host github.com-webjun7979
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_webjun7979
    IdentitiesOnly yes

# 회사 계정
Host github.com-company
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_company
    IdentitiesOnly yes
```

### 5단계: GitHub 호스트 키 추가 (최초 1회)

GitHub의 호스트 키를 `known_hosts`에 추가합니다.

```bash
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

### 6단계: 프로젝트의 원격 URL 변경

프로젝트의 원격 저장소 URL을 SSH 형식으로 변경합니다.

```bash
# 현재 원격 URL 확인
git remote -v

# 원격 URL을 SSH 형식으로 변경
git remote set-url origin git@github.com-계정명:계정명/저장소명.git
```

**예시:**
```bash
# 변경 전
origin  https://github.com/webjun7979/supabase-starter-tutorial.git

# 변경 명령
git remote set-url origin git@github.com-webjun7979:webjun7979/supabase-starter-tutorial.git

# 변경 후 확인
git remote -v
# origin  git@github.com-webjun7979:webjun7979/supabase-starter-tutorial.git (fetch)
# origin  git@github.com-webjun7979:webjun7979/supabase-starter-tutorial.git (push)
```

**URL 형식 설명:**
- `git@github.com-webjun7979`: SSH config에서 정의한 Host 별칭
- `:계정명/저장소명.git`: GitHub 저장소 경로

---

## 다른 프로젝트에 적용하기

다른 프로젝트에서 다른 계정을 사용하려면:

### 방법 1: 이미 설정된 계정 사용

이미 SSH 키와 config가 설정되어 있다면, 해당 프로젝트의 원격 URL만 변경하면 됩니다.

```bash
cd /path/to/other-project
git remote set-url origin git@github.com-다른계정명:다른계정명/저장소명.git
```

### 방법 2: 새로운 계정 추가

1. **새 SSH 키 생성**
   ```bash
   ssh-keygen -t ed25519 -C "new-account@github" -f ~/.ssh/id_ed25519_newaccount
   ```

2. **GitHub에 공개 키 추가** (3단계 참고)

3. **SSH config에 추가**
   ```bash
   # ~/.ssh/config 파일에 추가
   Host github.com-newaccount
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_newaccount
       IdentitiesOnly yes
   ```

4. **프로젝트 원격 URL 변경**
   ```bash
   git remote set-url origin git@github.com-newaccount:newaccount/repo.git
   ```

---

## 테스트 및 확인

### SSH 연결 테스트

각 계정별로 SSH 연결을 테스트합니다.

```bash
ssh -T git@github.com-계정명
```

**성공 시 출력:**
```
Hi 계정명! You've successfully authenticated, but GitHub does not provide shell access.
```

**실패 시 출력:**
- `Permission denied`: SSH 키가 GitHub에 등록되지 않았거나 잘못된 키 사용
- `Host key verification failed`: known_hosts에 GitHub 호스트 키가 없음

### Git 작업 테스트

```bash
# 현재 원격 URL 확인
git remote -v

# 최신 정보 가져오기
git fetch

# 간단한 변경사항 push 테스트
git push origin main
```

---

## 문제 해결

### 문제 1: "Permission denied (publickey)"

**원인**: SSH 키가 GitHub에 등록되지 않았거나 잘못된 키 사용

**해결 방법:**
1. 공개 키가 GitHub에 올바르게 등록되었는지 확인
2. SSH config의 `IdentityFile` 경로가 올바른지 확인
3. SSH 키 권한 확인:
   ```bash
   chmod 600 ~/.ssh/id_ed25519_계정명
   chmod 644 ~/.ssh/id_ed25519_계정명.pub
   ```

### 문제 2: "Host key verification failed"

**원인**: GitHub 호스트 키가 known_hosts에 없음

**해결 방법:**
```bash
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

### 문제 3: 여전히 잘못된 계정으로 인증됨

**원인**: SSH config가 제대로 적용되지 않음

**해결 방법:**
1. SSH config 파일 문법 확인:
   ```bash
   ssh -F ~/.ssh/config -T git@github.com-계정명
   ```
2. SSH config 파일 권한 확인:
   ```bash
   chmod 600 ~/.ssh/config
   ```
3. 원격 URL이 올바른 Host 별칭을 사용하는지 확인

### 문제 4: 여러 계정을 사용할 때 혼동됨

**해결 방법:**
- 각 프로젝트의 원격 URL을 명확히 확인:
  ```bash
  git remote -v
  ```
- SSH config에 주석 추가:
  ```
  # 개인 프로젝트용 - webjun7979 계정
  Host github.com-webjun7979
      HostName github.com
      User git
      IdentityFile ~/.ssh/id_ed25519_webjun7979
      IdentitiesOnly yes
  ```

### 문제 5: 기존 HTTPS credential 제거

기존에 저장된 HTTPS 인증 정보를 제거하려면:

**macOS (Keychain):**
```bash
# 특정 호스트의 credential 제거
git credential-osxkeychain erase <<EOF
host=github.com
protocol=https
EOF
```

**Linux:**
```bash
git credential-cache exit
# 또는
git config --global --unset credential.helper
```

---

## 참고: Personal Access Token 사용법

SSH 대신 Personal Access Token을 사용하는 방법입니다.

### 1단계: Personal Access Token 생성

1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. **Generate new token (classic)** 클릭
3. 권한 선택 (최소한 `repo` 권한 필요)
4. 토큰 생성 후 **반드시 복사** (다시 볼 수 없음)

### 2단계: 원격 URL에 사용자명 포함

```bash
git remote set-url origin https://계정명@github.com/계정명/저장소명.git
```

### 3단계: Push/Pull 시 토큰 입력

비밀번호 입력 요청 시 **Personal Access Token**을 입력합니다.

**단점:**
- 매번 토큰을 입력해야 함 (credential helper 사용 시 자동화 가능)
- 토큰이 만료되면 재생성 필요
- SSH보다 보안성이 낮을 수 있음

**장점:**
- 설정이 간단함
- SSH 키 관리 불필요

---

## 요약

프로젝트별로 다른 Git 계정을 사용하는 가장 좋은 방법:

1. ✅ 각 계정마다 별도의 SSH 키 생성
2. ✅ SSH config에 계정별 Host 설정
3. ✅ GitHub에 공개 키 등록
4. ✅ 프로젝트별로 원격 URL을 해당 계정의 Host 별칭으로 변경

이 방법을 사용하면 여러 Git 계정을 혼동 없이 안전하게 관리할 수 있습니다.

---

## 추가 리소스

- [GitHub SSH 키 가이드](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [SSH Config 파일 문서](https://www.ssh.com/academy/ssh/config)
- [Personal Access Token 가이드](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

