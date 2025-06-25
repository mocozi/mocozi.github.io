---
title: 딥시크 YAML
excerpt: 
tags: 
categories: 
toc: true
toc_sticky: true
date: 2025-06-25
last_modified_at: 
comments: true
author_profile: 
sidebar:
---
## 

```

name: Sync Blog Posts

on:
  push:
    paths:
      - '6. blog/**'

jobs:
  sync-posts:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source repo
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Configure Git
      run: |
        git config --global user.name "GitHub Actions"
        git config --global user.email "actions@github.com"

    - name: Copy files to target repo
      run: |
        # Create target directory if it doesn't exist
        mkdir -p ../target_repo/_posts
        
        # Copy only .md files from '6. blog' to '_posts'
        find "6. blog" -name '*.md' -exec cp {} ../target_repo/_posts \;
        
        # Optional: Rename files to Jekyll format (YYYY-MM-DD-title.md)
        cd ../target_repo/_posts
        for file in *.md; do
          if [[ ! "$file" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}- ]]; then
            # Extract date from frontmatter if available, or use current date
            DATE=$(grep -m 1 'date:' "$file" | cut -d ' ' -f 2 | sed 's/"//g' || date +%Y-%m-%d)
            # Clean up the date format
            DATE=$(echo "$DATE" | sed 's/[^0-9-]//g' | head -c 10)
            # If date extraction failed, use current date
            if [[ -z "$DATE" ]]; then
              DATE=$(date +%Y-%m-%d)
            fi
            mv "$file" "${DATE}-${file}"
          fi
        done

    - name: Checkout target repo
      uses: actions/checkout@v4
      with:
        repository: mocozi/mocozi.github.io
        path: ../target_repo
        token: ${{ secrets.ACTIONS_PAT }}

    - name: Commit and push changes
      run: |
        cd ../target_repo
        
        # Check if there are changes to commit
        if git diff --quiet --exit-code; then
          echo "No changes to commit"
        else
          git add _posts/
          git commit -m "Automated update from Obsidian blog folder [$(date +%Y-%m-%d)]"
          git push
        fi

```

>[!caution]
>반박시 당신의 말이 옳다.
