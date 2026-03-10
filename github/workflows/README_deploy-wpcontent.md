# kubohirotaka-cms
CMS２回目くろねこテスト用（AWS）
git　pushテスト用テキスト

# 🚀 WordPress (AWS Lightsail) × GitHub Actions 自動デプロイ構成

## 概要

本リポジトリは `main` ブランチへの push をトリガーとして  
AWS Lightsail 上の WordPress (`wp-content`) を自動デプロイする構成です。

---

## 🏗 構成図

```
Local (VSCode)
      ↓ push
GitHub (main)
      ↓
GitHub Actions
      ↓ SSH
AWS Lightsail
/opt/bitnami/wordpress/wp-content
```

---

# 📦 Git管理対象

## ✅ 管理する

```
wp-content/
 ├ themes/
 ├ plugins/
 └ mu-plugins/
```

## ❌ 管理しない

```
wp-content/uploads/
wp-content/cache/
wp-config.php
```

---

# 🛠 AWS 側設定

## 1. Lightsail インスタンス作成

- Platform: Linux/Unix
- Blueprint: WordPress (Bitnami)
- Public IPv4 を使用（Static IP未使用/※途中で静的IP使用に変更）

---

## 2. wp-content を Git 管理へ変更

```bash
cd /opt/bitnami/wordpress
mv wp-content wp-content.bak
mkdir wp-content
chown bitnami:daemon wp-content
```

GitHub リポジトリを clone:

```bash
cd /opt/bitnami/wordpress/wp-content
git clone git@github.com:USER/REPO.git .
```

---

## 3. サーバー側 Deploy key 作成

```bash
sudo -u bitnami ssh-keygen -t ed25519 -f /home/bitnami/.ssh/wpcontent_deploy_key
```

公開鍵を確認:

```bash
cat /home/bitnami/.ssh/wpcontent_deploy_key.pub
```

GitHub に登録:

```
Repository → Settings → Deploy keys → Add deploy key
```

---

# 🔐 GitHub 設定

## Repository Secrets

GitHub → Settings → Secrets and variables → Actions → Secrets

以下を追加:

| Name | 内容 |
|------|------|
| PROD_HOST | Lightsail の Public IPv4 |
| LIGHTSAIL_SSH_KEY | Lightsail接続用秘密鍵（.pem全文） |

---

# ⚙ GitHub Actions

`.github/workflows/deploy-wpcontent.yml`

```yaml
name: Deploy wp-content (git pull)

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Guard - ensure secrets exist
        run: |
          set -euo pipefail
          if [ -z "${{ secrets.PROD_HOST }}" ]; then
            echo "ERROR: PROD_HOST is empty."
            exit 1
          fi
          if [ -z "${{ secrets.LIGHTSAIL_SSH_KEY }}" ]; then
            echo "ERROR: LIGHTSAIL_SSH_KEY is empty."
            exit 1
          fi

      - name: Setup SSH
        run: |
          set -euo pipefail
          mkdir -p ~/.ssh
          echo "${{ secrets.LIGHTSAIL_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H "${{ secrets.PROD_HOST }}" >> ~/.ssh/known_hosts

      - name: Remote git sync wp-content (hard reset)
        run: |
          set -euo pipefail
          ssh -i ~/.ssh/id_ed25519 "bitnami@${{ secrets.PROD_HOST }}" << 'EOF'
            set -euo pipefail
            cd /opt/bitnami/wordpress/wp-content
            GIT_SSH_COMMAND="ssh -i /home/bitnami/.ssh/wpcontent_deploy_key" git fetch origin
            GIT_SSH_COMMAND="ssh -i /home/bitnami/.ssh/wpcontent_deploy_key" git reset --hard origin/main
            git rev-parse --short HEAD
          EOF
```

---

# 🚀 デプロイ手順

```
git checkout main
git add .
git commit -m "deploy"
git push origin main
```

→ GitHub Actions が自動実行  
→ Lightsail に反映

---

# 🔍 デプロイ確認

## サーバー側

```bash
cd /opt/bitnami/wordpress/wp-content
git log -1 --oneline
```

GitHub の最新コミットと一致していれば成功。

---

# ⚠ 注意事項

- サーバー側で直接編集しない（reset --hard で消える）
- Public IP が変更された場合は `PROD_HOST` を更新する
- uploads は Git 管理しない

---

# 📌 開発フロー

```
develop で作業
    ↓
PR作成
    ↓
mainへマージ
    ↓
自動デプロイ
```

---

# 🏁 現在の構成レベル

- 小〜中規模サイト向け
- CI/CD対応
- シンプル運用
- ロールバック可能（git管理）