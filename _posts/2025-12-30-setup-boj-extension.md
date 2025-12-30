---
layout: post
title: "[Github] 백준과 깃허브 연동"
date: 2025-12-30 21:00:00 +0900
categories: [개발, 자동화]
tags: [github-actions, python, baekjoon, solved.ac]
---

알고리즘 공부를 하면서 가장 번거로운 것은 **"문제 풀이 코드 저장"**과 **"풀이 방법 정리"**였습니다.
마침 깃허브 사용을 익숙하게 할 겸, **Chrome 확장 프로그램**과 **GitHub Actions**를 조합하여 저만의 자동화 시스템을 구축해 보았습니다.

---

## 📅 Phase 1. 시작: 백준허브(BaekjoonHub) 도입
처음에는 단순히 "푼 코드를 깃허브에 올리고 싶다"라는 생각으로 시작했습니다.

### 🛠️ 설치 및 연동
Chrome 확장 프로그램인 **[BaekjoonHub]**를 설치했습니다.
- **장점:** 백준에서 '제출' 버튼만 누르면 자동으로 내 GitHub 리포지리에 코드가 푸시(Push)된다는 점이었습니다.
- **아쉬운 점:** 다만, 코드 파일만 덩그러니 올라가고, 내가 어떤 생각으로 풀었는지 적을 공간이 없었습니다. (`README.md`가 생성되긴 하지만 단순한 문제 설명뿐)

---

## 📅 Phase 2. 욕심: "이슈(Issue)로 풀이 노트를 만들자"
코드가 올라갈 때, 자동으로 **GitHub Issue**를 생성해서 문제 정보와 풀이 템플릿을 만들어주면 좋겠다고 생각했습니다.
그래서 GitHub Actions를 이용해 파이썬 스크립트를 돌리기로 했습니다.

### 🎯 목표 기능
1. 백준허브가 코드를 Push하면 Actions가 감지.
2. `Solved.ac` API로 문제의 난이도, 태그, 제목을 가져옴.
3. 예쁜 마크다운 형식으로 **GitHub Issue** 생성.
4. 해당 문제 폴더의 `README.md`에 이슈 링크를 자동으로 추가.

---

## 📅 Phase 3. Gemini를 이용한 스크립트 작성
아무래도 markdown과 프로그램 언어를 이용하여 스크립트를 작성해 본 경험이 적어서, AI의 도움을 받아 스크립트를 작성했습니다.
AI를 사용하면서 항상 느끼는 점은, 사용자에 따라 AI 성능도 달라지는 것 같습니다. 왜냐하면 저는 한번에 에러없는 코드를 얻어본 적이 없기 때문이죠.
하지만 몇 번 코드에서 이상한 점을 찾고, 오류를 AI한테 설명해 주는 과정을 거치면 간단한 코드들은 바로 실행할 수 있게 되니, 매력적인건 변하지 않는 것 같습니다.

아무튼 이제 백준에서 문제를 풀면, 해당 문제는 깃허브에 저장되고, 자동으로 풀이 노트가 Issue에 생성됩니다.
이슈가 자동으로 생성되지 않으면, Actions 탭에서 왼쪽의 **Auto Create Issue & Link to Folder** 도구를 클릭한 다음, 오른쪽의 **Run workflow**에서 풀이 노트 생성을 원하는 문제 번호를 입력하시면 해당 문제가 생성됩니다. (단, 백준 리포지터리 폴더에 해당 문제 폴더가 생성되어 있어야 합니다.)

### ✨ 최종 결과물
자동으로 생성된 이슈는 아래와 같이 예쁘게 정리됩니다. 

**자동 생성된 이슈 예시**
> **제목:** [BOJ] 23971번 ZOAC 4 - Bronze 3
>
> ![Tier Badge](https://img.shields.io/badge/Bronze%203-ad5600?style=flat-square&logo=solved.ac&logoColor=white)
>
> | 문제 정보 | 바로가기 |
> | :-: | :-: |
> | **난이도** | Bronze 3 |
> | **문제 번호** | 23971 |
> | **태그** | `수학`, `사칙연산` |
>
> **링크:** [문제 풀러 가기] / [내 정답 코드 보기(Github)]


**실제 화면**
{: .text-center }
| 백준 확장프로그램 | 자동 생성된 이슈 |
| :-: | :-: |
| ![백준 확장프로그램](/assets/img/2025post/2025-12-30-setup-boj-extension/2025-12-30-boj-extension.jpg) | ![자동 생성된 Issue](/assets/img/2025post/2025-12-30-setup-boj-extension/2025-12-30-boj-issue-note.jpg) |



### 📂 적용한 GitHub Actions 코드
혹시 필요한 분들을 위해 최종 완성된 `.github/workflows/auto-issue-link.yml` 코드와, 해당 코드에 필요한 파이썬 스크립트 `scripts/issue_linker.py`를 공유합니다. 각각 백준 확장프로그램이 자동으로 생성한 리포지리에서 폴더를 만들고, 알맞은 폴더에 코드를 넣으신 다음, 문제를 풀면 됩니다. 

참고로 코딩이나 형식 등이 마음에 들지 않는다 싶으면 수정하셔도 되고, 오류가 발생하면 댓글 부탁드리겠습니다.

[Github 링크](https://github.com/swh00/BOJ-solutions)

<details>
<summary><b>📜 .github/workflows/boj-auto-filler.yml (클릭)</b></summary>

```yaml
name: Auto Create Issue & Link to Folder

on:
  push:
    paths:
      - '**/*.py'
      - '**/*.java'
      - '**/*.cpp'
      - '**/*.c'
      - '**/*.cc'
      - '**/*.js'
      - '**/*.ts'
  workflow_dispatch:
    inputs:
      problem_id:
        description: '강제로 이슈를 생성할 문제 번호'
        required: false
        type: string

# 동시 실행 방지 (Race Condition 방지)
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

permissions:
  contents: write
  issues: write

jobs:
  link-issue-to-folder:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0  # git diff를 위해 전체 히스토리 가져오기

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install Dependencies
        run: pip install requests

      - name: Configure Git
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"

      - name: Run Issue Linker Script
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPO: ${{ github.repository }}
          BRANCH: ${{ github.ref_name }}
          TARGET_ID: ${{ inputs.problem_id }}
        run: python scripts/issue_linker.py
```
</details>

<details>
<summary><b>📜 scripts/issue_linker.py (클릭)</b></summary

```issue_linker.py
# scripts/issue_linker.py
import os
import requests
import subprocess
import re
import urllib.parse
import json
import textwrap

TIER_COLORS = {
    'Unrated': '333333', 'Bronze': 'ad5600', 'Silver': '435f7a',
    'Gold': 'ec9a00', 'Platinum': '27e2a4', 'Diamond': '00b4fc',
    'Ruby': 'ff0062', 'Master': 'b300e0'
}

def get_changed_files():
    """Git 변경 사항 또는 입력된 문제 번호에 해당하는 파일 검색"""
    target_id = os.environ.get('TARGET_ID', '').strip()
    
    # 1. 수동 실행 (문제 번호 입력 시)
    if target_id:
        print(f"🔎 [Manual] 문제 번호 {target_id}번 파일 검색 중...")
        found_files = []
        for root, _, files in os.walk("."):
            if ".git" in root: continue
            for file in files:
                if file.endswith(('.py', '.java', '.cpp', '.c', '.cc', '.js', '.ts')):
                    full_path = os.path.join(root, file)
                    # 경로 전체에서 아이디 검색
                    if str(target_id) in full_path:
                         found_files.append(full_path)
        return found_files

    # 2. 자동 실행 (Git 변경 파일 감지)
    try:
        # 한글 깨짐 방지 설정 후 실행
        subprocess.run(["git", "config", "--global", "core.quotepath", "false"])
        cmd = "git diff --name-only HEAD~1 HEAD"
        output = subprocess.check_output(cmd, shell=True).decode('utf-8')
        return [f.strip().strip('"') for f in output.split('\n') if f.strip()]
    except subprocess.CalledProcessError:
        print("⚠️ 이전 커밋을 찾을 수 없어 변경된 파일을 감지하지 못했습니다.")
        return []

def get_problem_info(problem_id):
    url = f"https://solved.ac/api/v3/problem/show?problemId={problem_id}"
    try:
        res = requests.get(url, headers={"Content-Type": "application/json"}, timeout=10)
        if res.status_code == 200:
            return res.json()
    except Exception as e:
        print(f"❌ Solved.ac API Error: {e}")
    return None

def get_existing_issue_url(problem_id):
    """이미 존재하는 이슈가 있는지 검색"""
    cmd = [
        "gh", "issue", "list",
        "--search", f"{problem_id} in:title",
        "--repo", os.environ['REPO'],
        "--json", "url",
        "--limit", "1"
    ]
    try:
        output = subprocess.check_output(cmd).decode('utf-8')
        result = json.loads(output)
        return result[0]['url'] if result else None
    except:
        return None

def update_readme(readme_path, issue_url):
    """README에 이슈 링크 추가"""
    if not os.path.exists(readme_path):
        print(f"⚠️ README 없음: {readme_path}")
        return False
    
    with open(readme_path, "r", encoding="utf-8") as f:
        content = f.read()
    
    if issue_url in content:
        return False # 이미 링크가 존재함
    
    with open(readme_path, "a", encoding="utf-8") as f:
        f.write(f"\n<br>\n\n### 💡 [노트] 풀이 보러가기\n")
        f.write(f"- [Github Issue 링크]({issue_url})\n")
    
    print(f"✅ README 업데이트: {readme_path}")
    return True

def create_issue(pid, file_path, data):
    repo = os.environ['REPO']
    branch = os.environ['BRANCH']
    
    title_ko = data['titleKo']
    level = data['level']
    
    # 뱃지 생성
    if level == 0: badge_name, badge_color = "Unrated", TIER_COLORS['Unrated']
    else:
        tiers = ['Bronze', 'Silver', 'Gold', 'Platinum', 'Diamond', 'Ruby']
        tier_idx = (level - 1) // 5
        tier_num = 5 - ((level - 1) % 5)
        # 인덱스 에러 방지
        if tier_idx < len(tiers):
            tier_name = tiers[tier_idx]
            badge_name = f"{tier_name} {tier_num}"
            badge_color = TIER_COLORS[tier_name]
        else:
             badge_name, badge_color = "Master", TIER_COLORS['Master']
    
    tier_badge_url = f"https://img.shields.io/badge/{badge_name.replace(' ', '%20')}-{badge_color}?style=flat-square&logo=solved.ac&logoColor=white"
    tags = ", ".join([f"`{t['displayNames'][0]['name']}`" for t in data['tags']])
    
    encoded_path = urllib.parse.quote(file_path)
    code_url = f"https://github.com/{repo}/blob/{branch}/{encoded_path}"
    problem_link = f"https://www.acmicpc.net/problem/{pid}"
    
    issue_title = f"[BOJ] {pid}번 {title_ko} - {badge_name}"
    issue_body = textwrap.dedent(f"""\
        # {issue_title}

        ![Tier]({tier_badge_url})

        | 문제 정보 | 바로가기 |
        | :-: | :-: |
        | **난이도** | {badge_name} |
        | **문제 번호** | {pid} |
        | **태그** | {tags} |

        <br>

        ### 🔗 링크
        - [문제 풀러 가기]({problem_link})
        - [내 정답 코드 보기 (Github)]({code_url})

        <br>

        ## 1. 문제 파악
        - 

        ## 2. 접근 방법
        1. 
        2. 

        ## 3. 코드 구현 시 주의점
        - 

        ## 4. 배우고 느낀 점
        - 
    """)
    
    # 이슈 생성 명령
    cmd = [
        "gh", "issue", "create",
        "--title", issue_title,
        "--body", issue_body,
        "--repo", repo
    ]
    
    try:
        proc = subprocess.run(cmd, capture_output=True, text=True)
        if proc.returncode == 0:
            return proc.stdout.strip()
        else:
            print(f"❌ 이슈 생성 실패: {proc.stderr}")
    except Exception as e:
        print(f"❌ 시스템 에러: {e}")
    return None

def main():
    files = get_changed_files()
    processed_ids = set()
    changes_made = False

    print(f"🔍 감지된 파일: {files}")

    for file_path in files:
        numbers = re.findall(r'(\d+)', file_path)
        if not numbers: continue
        
        # 백준 문제 번호는 보통 1000번 이상임
        pid = 0
        for num in numbers:
            if int(num) >= 1000:
                pid = int(num)
                break
        
        if pid == 0 or pid in processed_ids: continue

        print(f"-------------------------------------------")
        print(f"🚀 처리 중: {pid}번 (파일: {file_path})")
        
        # README 경로 찾기
        dir_path = os.path.dirname(file_path)
        readme_path = os.path.join(dir_path, "README.md")
        
        issue_url = get_existing_issue_url(pid)

        if not issue_url:
            data = get_problem_info(pid)
            if data:
                print(f"✨ 새 이슈 생성 시도: {pid}번")
                issue_url = create_issue(pid, file_path, data)
                if issue_url:
                    print(f"🎉 이슈 생성 완료: {issue_url}")
            else:
                print(f"❌ 문제 정보를 가져올 수 없음: {pid}")
        else:
            print(f"ℹ️ 이미 존재하는 이슈: {issue_url}")

        if issue_url and update_readme(readme_path, issue_url):
            subprocess.run(["git", "add", readme_path])
            changes_made = True
        
        processed_ids.add(pid)

    if changes_made:
        print("💾 변경사항 커밋 및 푸시 중...")
        subprocess.run(["git", "commit", "-m", "Auto: Link Github Issue to README"])
        subprocess.run(["git", "push"])
    else:
        print("💤 변경사항 없음 (README 업데이트 내역 없음)")

if __name__ == "__main__":
    main()
```
