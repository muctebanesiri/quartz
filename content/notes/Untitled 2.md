---
title:
description:
keywords:
draft: true
tags:
date:
---

```
name: Download from Commit & Save to Repo
on:
  push:
    branches:
      - "**"
jobs:
  save-file:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Install aria2
        run: sudo apt-get install -y aria2

      - name: Extract URLs and download files
        run: |
          MSG=$(git log -1 --pretty=%B)
          echo "Commit message: $MSG"

          if echo "$MSG" | grep -qP 'download-zip:'; then
            MODE="zip"
            URL_LINE=$(echo "$MSG" | grep -oP 'download-zip:\s*\K.*')
          elif echo "$MSG" | grep -qP 'download:'; then
            MODE="normal"
            URL_LINE=$(echo "$MSG" | grep -oP 'download:\s*\K.*')
          else
            echo "❌ No download command found in commit message"
            exit 1
          fi

          echo "Mode: $MODE"
          mkdir -p downloads tmp_downloads

          for URL in $URL_LINE; do
            FILENAME=$(basename "$URL")
            echo "⬇️ Downloading $URL -> $FILENAME"

            aria2c \
              --split=2 \
              --max-connection-per-server=2 \
              --min-split-size=90M \
              --dir="tmp_downloads" \
              "$URL"
          done

          if [ "$MODE" = "zip" ]; then
            ARCHIVE_NAME="tmp_downloads/archive_$(date +%Y%m%d_%H%M%S).zip"
            zip -j "$ARCHIVE_NAME" tmp_downloads/*
          fi

          # Split any zip over 90MB using zip native split (unzip compatible)
          for FILE in tmp_downloads/*; do
            [ -f "$FILE" ] || continue
            SIZE=$(stat -c%s "$FILE")
            LIMIT=$((90 * 1024 * 1024))
            BASENAME=$(basename "$FILE")

            if [ "$SIZE" -gt "$LIMIT" ]; then
              echo "✂️ $BASENAME is $(( SIZE / 1024 / 1024 ))MB — splitting with zip..."
              # zip -s creates: file.zip, file.z01, file.z02, ...
              #zip -s 90m "downloads/${BASENAME}.zip" --out "downloads/${BASENAME}.zip" "$FILE"
              zip -s 90m "downloads/${BASENAME}.zip" -j "$FILE"
              echo "✅ Parts: $(ls downloads/${BASENAME}* | wc -l)"
            else
              cp "$FILE" "downloads/$BASENAME"
            fi
          done

          rm -rf tmp_downloads

      - name: Commit & Push
        run: |
          BRANCH="${GITHUB_REF_NAME}"
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"
          git add downloads/
          git commit -m "Add downloaded files from commit [skip ci]" || echo "Nothing to commit"
          echo "🚀 Pushing to branch: $BRANCH"
          git push origin HEAD:$BRANCH
```