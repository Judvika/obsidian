## 1. 목표
- **회사 Vault** → 회사 GitHub 계정(`co-20020455/obsidian`)과만 동기화  
- **개인 Vault** → 개인 GitHub 계정(`Judvika/obsidian`)과만 동기화  
- PC, 모바일, 태블릿에서 모두 일관된 방식으로 관리  
- 회사 자료와 개인 자료가 서로 오염되지 않도록 분리  

---

## 2. PC 환경

### 회사 PC
- 오직 **회사 Vault**만 사용.  
- `.git sparse-checkout`으로 `Private` 폴더 제외.  
- SSH 키(`id_ed25519_company`)를 GitHub에 등록 후 SSH 연결.

### 개인 PC
- **회사 Vault + 개인 Vault** 동시에 사용.  
- `Private` 폴더를 회사 repo에서 제거 후 개인 repo(`Judvika/obsidian`)로 분리.  
- SSH 키 2개(`id_ed25519_company`, `id_ed25519_personal`)를 등록,  
  `~/.ssh/config`에서 `github-company`, `github-personal`로 분리 관리.  

---

## 3. 모바일/태블릿 환경

### 문제
- Obsidian Git 플러그인은 **한 Vault = 한 Repo**만 지원.  
- 처음엔 `obsidian/obsidian/Private` 폴더를 회사 repo에 둬서  
  → Pull 시 회사 repo에 Private 내용이 섞이는 문제 발생.

### 해결
- Vault 자체를 분리:
📂 Obsidian
┣ 📂 CompanyVault → 회사 repo (co-20020455/obsidian)
┗ 📂 PrivateVault → 개인 repo (Judvika/obsidian)

markdown
코드 복사
- Termux로 각각 `git init`, `git remote add`, `git push` 설정.  
- Obsidian 앱에서 Vault 2개를 각각 불러오기.  

---

## 4. 모바일 인증 방식

### SSH ❌
- Obsidian Git 플러그인에서 SSH 지원 불완전 → 에러 발생.

### HTTPS + Personal Access Token (PAT) ✅
1. GitHub → Settings → Developer Settings → **Generate new token (classic)**  
 - Note: `Obsidian-Mobile`  
 - Expiration: 필요에 맞게 (예: 90일)  
 - Scopes: `repo` 체크  
2. Termux에서:
 ```bash
 git config --global credential.helper store
git push 시:

Username → hyunho

Password → 발급받은 토큰(ghp_...)

한 번 입력 후 자동 저장됨.

5. 실패 사례와 교훈
❌ 실패 1: Private 폴더를 회사 repo에 둔 상태
결과: 회사 repo에 Private까지 푸시됨.

원인: sparse-checkout은 PC에서만 동작, 모바일은 전부 추적.

교훈: 모바일은 Vault 자체 분리 필수.

❌ 실패 2: SSH 연결 시도
결과: ssh: Could not resolve hostname github-company 오류.

원인: 플러그인이 SSH 미지원.

교훈: 모바일은 HTTPS + 토큰.

❌ 실패 3: HTTPS push 시 Password 입력
결과: 비밀번호로 인증 불가.

원인: GitHub 정책 변경 (2021년 이후 비밀번호 불가).

교훈: Personal Access Token만 사용 가능.

6. 최종 구조
bash
코드 복사
📂 Obsidian
 ┣ 📂 CompanyVault   → 회사 repo (co-20020455/obsidian)
 ┗ 📂 PrivateVault   → 개인 repo (Judvika/obsidian)
PC → SSH (회사/개인 키 분리)

모바일/태블릿 → HTTPS + PAT

Obsidian Git 플러그인 → 각 Vault마다 Push/Pull/Commit 실행

✅ 이렇게 하면:

회사 메모 ↔ 회사 계정만 동기화

개인 메모 ↔ 개인 계정만 동기화

서로 간섭 없음, 충돌 없음