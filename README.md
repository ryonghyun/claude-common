# claude-common

여러 프로젝트에서 함께 쓰는 Claude Code 공통 규칙이다.
각 프로젝트의 `.claude/rules/common/` 폴더에 git subtree로 넣어 쓴다.

## 파일 구성

| 파일 | 내용 |
| --- | --- |
| `communication.md` | 요청 정리, 용어 표기, 한국어 문장, 출처 표기 |
| `work-scope.md` | 작업 범위, 작업 전 질문, 규칙 파일 관리 |
| `best-practices.md` | 모범 사례를 찾아 적용하는 방법, 출처 등급 |
| `git.md` | 커밋 메시지 형식 |
| `README.md` | 안내 문서. 규칙이 아니므로 각 프로젝트의 `settings.json`에서 Claude가 읽지 않도록 제외한다 |

## 프로젝트에 넣기

공통 규칙은 커밋이 하나 이상 있는 프로젝트 레포에 넣을 수 있다.
커밋이 하나도 없으면 `git subtree add`가 `working tree has modifications. Cannot add.` 오류를 낸다.
GitHub에서 레포를 만들 때 README 파일을 함께 만드는 옵션을 켜면 첫 커밋이 생긴다.

1. 프로젝트 레포를 clone하고 그 폴더로 이동한다. 이미 clone한 프로젝트라면 그 폴더로 이동만 한다.

   ```bash
   git clone https://github.com/ryonghyun/my-project.git
   cd my-project
   ```

2. 공통 규칙을 `.claude/rules/common/` 폴더로 가져온다.

   ```bash
   git subtree add --prefix=.claude/rules/common https://github.com/ryonghyun/claude-common.git main -m "chore(rules): add common rules subtree"
   ```

3. `.claude/settings.json`에 `claudeMdExcludes`를 넣어, 함께 들어온 `README.md`를 Claude가 읽지 않게 한다.
   Claude Code는 `.claude/rules/` 아래의 `.md` 파일을 모두 규칙으로 읽기 때문이다.

   ```json
   {
     "claudeMdExcludes": [
       "**/.claude/rules/common/README.md"
     ]
   }
   ```

   - 경로를 `**/`로 시작하는 이유: Claude Code는 `claudeMdExcludes`에 적은 경로를 상대 경로가 아니라 `/home/...`로 시작하는 전체 경로와 비교한다.
     그래서 컴퓨터마다 다른 앞부분을 `**/`(어떤 경로든 상관없다는 뜻)로 대신 쓴다.
   - `.claude/settings.json`이 이미 있으면 기존 내용은 그대로 두고 `claudeMdExcludes` 키만 추가한다.
     JSON은 키 사이를 쉼표로 구분하므로, 바로 앞 키 끝에 쉼표를 붙인다.

   `settings.json` 파일을 커밋한다.

   ```bash
   git add .claude/settings.json
   git commit -m "chore(config): exclude claude-common README from rules"
   ```

4. 변경 사항을 GitHub에 push한다.

   ```bash
   git push origin main
   ```

5. 프로젝트 폴더에서 Claude Code를 새로 실행하고 `/context`를 입력한다.
   Memory files 목록에 규칙 파일 4개가 있고 `README.md`는 없으면 제대로 들어간 것이다.

## 공통 규칙 고치기

공통 규칙은 사람이 직접 쓰지 않고, 프로젝트 안에서 Claude Code에게 요청해 Claude가 고친다.
Claude는 고칠 때 `best-practices.md`에 따라 모범 사례를 찾아 반영하고, `communication.md`에 따라 표현을 다듬는다.
문구 제안과 승인, 커밋 나누기, claude-common으로 보내기는 `work-scope.md`의 "규칙 파일 관리"에 적힌 절차를 따른다.

## 최신 공통 규칙 받아 오기

다른 프로젝트에서 고친 규칙은 지금 작업하는 프로젝트에 자동으로 들어오지 않는다.
받아 오지 않으면 프로젝트마다 Claude가 서로 다른 규칙을 따르게 된다.

claude-common이 바뀌었을 때 프로젝트마다 실행한다.
바뀌었는지 모르면 작업을 시작하기 전에 실행한다. 바뀐 것이 없으면 `Already up to date.`만 출력된다.

```bash
git subtree pull --prefix=.claude/rules/common https://github.com/ryonghyun/claude-common.git main -m "chore(rules): pull common rules"
git push origin main
```

## git subtree

### 쓰는 이유

- 규칙 파일이 프로젝트 레포에 일반 파일로 커밋된다. 그래서 어느 컴퓨터든, 클라우드에서 실행하는 Claude Code 세션이든 프로젝트를 clone하기만 하면 규칙이 들어온다.
- 프로젝트를 clone한 사람은 따로 설치하거나 설정할 것이 없다.
- 프로젝트에서 고친 규칙을 claude-common으로 보내고, 다른 프로젝트에서 받아 올 수 있다.

### 원리

claude-common 레포가 원본이고, 각 프로젝트의 `.claude/rules/common/` 폴더는 복사본이다.
git subtree 명령은 원본과 복사본 사이에서 변경을 옮긴다.
git subtree는 내부적으로 커밋, merge, push 같은 일반 git 기능을 쓴다. 프로젝트의 다른 파일은 지금까지처럼 다루면 된다.

- `git subtree add`: 원본의 파일을 프로젝트의 `.claude/rules/common/` 폴더에 복사하고 커밋한다.
  커밋 메시지에는 원본의 어느 커밋에서 가져왔는지 적어 둔다.
  그 뒤로 `.claude/rules/common/` 폴더의 파일은 프로젝트의 다른 파일과 다름없는 일반 파일이다.
- `git subtree pull`: 마지막으로 가져온 뒤 원본에 생긴 변경을 가져와 `.claude/rules/common/` 폴더에 합치고 커밋한다.
- `git subtree push`: 프로젝트 커밋 중 `.claude/rules/common/` 폴더를 바꾼 커밋만 골라 원본으로 보낸다.
  같은 파일이 프로젝트에서는 `.claude/rules/common/git.md`이고 원본에서는 `git.md`이므로,
  git subtree는 원본의 경로에 맞춘 커밋을 새로 만들어 보낸다.
  새로 만든 커밋은 바꾼 내용이 같아도 git에게는 다른 커밋이다. git은 파일 경로와 앞 커밋까지 포함해 커밋을 구별하기 때문이다.
- `--rejoin`: `git subtree push`에 붙이는 옵션이다.
  원본으로 보낸 커밋이 프로젝트의 어느 변경과 같은지 프로젝트에 커밋으로 기록한다. 이 커밋은 파일을 바꾸지 않는다.
  기록이 있으면 다음 `git subtree push`가 마지막 기록 이후의 커밋만 살펴보므로 빨라지고,
  `git subtree pull` 때 git이 원본으로 보낸 변경을 이미 합친 변경으로 알아보므로 충돌이 날 가능성이 줄어든다.
  대신 `git log`에 같은 변경이 프로젝트 커밋과 원본으로 보낸 커밋으로 두 번 보인다.
- `-m`: `add`, `pull`, `push --rejoin`은 일반 `git push`와 달리 프로젝트에 커밋을 새로 만든다.
  `-m`은 그 커밋의 제목을 정한다. 붙이지 않으면 `Split '...' into commit ...` 같은 기본 제목이 붙어 `git.md`의 커밋 메시지 규칙에서 벗어난다.
