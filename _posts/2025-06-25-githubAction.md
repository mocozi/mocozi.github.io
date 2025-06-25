---
title: 옵시디언으로 작성하고 깃허브 블로그에 자동 포스팅
excerpt: 깃허브 블로그에 글을 올리는 작업이 귀찮다. 옵시디언에서 바로 포스팅하는 방법을 알아보자.
tags: 
categories:
  - blog
toc: true
toc_sticky: true
date: 2025-06-25
last_modified_at: 2025-06-25
comments: true
author_profile: 
sidebar:
---
## Github Action을 이용해 자동화를 만든다.

1. 옵시디언(Obsidian)을 깃허브와 연동한다.
2. 특정 폴더가 갱신되면 깃허브 블로그의 posts폴더로 복사되는 깃허브 액션을 만들어 자동화 한다.

```

name: Sync Blog Posts to GitHub Pages

on:
  push:
    branches:
      - main  # 또는 기본 브랜치 이름 (예: master)
    paths:
      - '6. Blog/**'  # "6. Blog" 폴더 변경 시에만 트리거 (대소문자 확인요망)

jobs:
  sync-posts:
    runs-on: ubuntu-latest

    steps:
      # 1. 소스 리포지토리 (Obsidian) 체크아웃
      - name: Checkout Obsidian repo
        uses: actions/checkout@v4
        with:
          path: 'source-repo'

      # 2. Git 사용자 정보 설정
      - name: Configure Git
        run: |
          git config --global user.name "mocozi"
          git config --global user.email "mocozi@hotmail.com"

      # 3. 대상 리포지토리 (GitHub Pages) 체크아웃
      - name: Checkout GitHub Pages repo
        uses: actions/checkout@v4
        with:
          repository: mocozi/mocozi.github.io
          token: ${{ secrets.ACTIONS_PAT }}
          path: 'target-repo'

      # 4. 파일 복사 및 Jekyll 형식으로 변환
      - name: Copy and format posts
        run: |
          # "6. Blog" 폴더에서 .md 파일만 _posts로 복사
          cp -R source-repo/"6. Blog"/*.md target-repo/_posts/ || true

          # Jekyll 호환 파일명으로 변경 (YYYY-MM-DD-title.md)
          cd target-repo/_posts
          for file in *.md; do
            if [[ ! "$file" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}- ]]; then
              # Frontmatter에서 date 추출 (예: date: 2023-11-20)
              DATE=$(grep -m 1 'date:' "$file" | cut -d ' ' -f 2 | tr -d '"' || date +%Y-%m-%d)
              # 파일명에 날짜 추가
              mv "$file" "${DATE}-${file}"
            fi
          done

      # 5. 변경사항 커밋 및 푸시
      - name: Commit and Push
        run: |
          cd target-repo
          git add _posts/
          git diff --quiet && git diff --staged --quiet || git commit -m "Auto-sync posts from Obsidian [$(date +%Y-%m-%d)]"
          git push


```

특징
- 옵시디언의 "6. Blog" 폴더에서 .md 파일만 _posts로 복사한다.
- Jekyll 호환 파일명으로 변경 (YYYY-MM-DD-title.md)해서 복사한다.

현재 이 코드를 사용중이다.

아래는 원본(6. Blog)에 없는 파일은 대상(_posts)에서도 삭제하여 완벽하게 동기화하는 코드다.

```

# 워크플로우의 이름
name: Sync Blog posts to mocozi.github.io

# 언제 이 워크플로우를 실행할 것인가를 정의
on:
  push:
    branches:
      # 'main' 브랜치에 푸시가 일어날 때만 실행
      - main
    paths:
      # 푸시된 파일 중에 '6. Blog/' 폴더 내에 변경사항이 있을 때만 실행
      # 이렇게 하면 관련 없는 노트 수정 시에는 Actions가 실행되지 않아 효율적입니다.
      - '6. Blog/**'

# 어떤 작업을 수행할 것인가를 정의
jobs:
  sync:
    # 작업은 최신 우분투(리눅스) 환경에서 실행
    runs-on: ubuntu-latest
    steps:
      # 1. 소스 저장소(mocozi/Obsidian)의 코드를 내려받습니다.
      - name: Checkout Obsidian Vault Repo
        uses: actions/checkout@v4
        with:
          # 소스 코드를 'source'라는 폴더에 내려받습니다.
          path: source

      # 2. 대상 저장소(mocozi/mocozi.github.io)의 코드를 내려받습니다.
      - name: Checkout Blog Repo
        uses: actions/checkout@v4
        with:
          # 대상 저장소의 주소를 명시합니다.
          repository: mocozi/mocozi.github.io
          # 2단계에서 등록한 Secret(PAT)을 사용하여 인증합니다.
          token: ${{ secrets.ACTIONS_PAT }}
          # 대상 코드를 'destination'이라는 폴더에 내려받습니다.
          path: destination

      # 3. 소스 폴더의 파일을 대상 폴더로 복사합니다.
      - name: Sync files to _posts
        run: |
          # rsync라는 강력한 복사 명령어를 사용합니다.
          # -a: 원본 파일의 속성을 유지하며 복사
          # -v: 복사되는 파일 목록을 보여줌
          # --delete: 원본(6. Blog)에 없는 파일은 대상(_posts)에서도 삭제하여 완벽하게 동기화
          rsync -av --delete source/'6. Blog'/ destination/_posts/

      # 4. 변경된 내용을 대상 저장소에 커밋하고 푸시합니다.
      - name: Commit and Push to Blog Repo
        run: |
          # 작업 디렉토리를 대상 저장소를 내려받은 'destination' 폴더로 변경
          cd destination
          
          # git에 커밋할 사용자 정보를 설정 (봇이 작업했음을 명시)
          git config user.name "GitHub Actions Bot"
          git config user.email "actions@github.com"
          
          # git status --porcelain으로 변경된 파일이 있는지 확인합니다.
          # 변경된 파일이 없으면, 불필요한 커밋을 만들지 않고 작업을 종료합니다.
          if [[ -z $(git status --porcelain) ]]; then
            echo "No changes to commit"
          else
            # _posts 폴더의 모든 변경사항을 스테이징
            git add _posts
            # 커밋 메시지 작성
            git commit -m "docs: Update Blog posts from Obsidian Vault"
            # 변경사항을 원격 저장소로 푸시
            git push
          fi
          

```

