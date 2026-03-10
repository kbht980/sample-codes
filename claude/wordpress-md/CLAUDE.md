# プロジェクト概要

このリポジトリは WordPress サイトのテーマ・プラグイン開発用です。

WordPress本体（core）はGit管理していません。
このリポジトリでは主に `wp-content` 以下のみを管理します。

編集対象

wp-content/
themes/
plugins/

WordPress core ファイルの編集は禁止です。

---

# ディレクトリ構成

project-root/

wp-content/
  themes/
  plugins/

.github/
  workflows/

CLAUDE.md

---

# 開発環境

Language

PHP 8.x  
HTML5  
SCSS / CSS  
JavaScript (Vanilla JS)

CMS

WordPress

Version Control

Git / GitHub

---

# Git運用

ブランチ構成

main  
→ 本番環境

stg  
→ ステージング環境

feature/*  
→ 開発ブランチ

基本フロー

featureブランチ作成  
↓  
Pull Request → stg  
↓  
ステージング確認  
↓  
stg → main マージ  
↓  
本番デプロイ

---

# デプロイ

GitHub Actions により自動デプロイします。

stg branch push  
→ ステージング環境

main branch push  
→ 本番環境

本番サーバーの直接編集は禁止です。

---

# WordPress開発ルール

以下を必ず守ってください。

・WordPress core は変更しない  
・wp-content 以下のみ編集する  
・WordPress Coding Standards に従う  

参考  
https://developer.wordpress.org/coding-standards/

---

# WordPressテンプレート階層

WordPressのテンプレート階層に従って実装します。

front-page.php  
home.php  
single.php  
archive.php  
page.php  
index.php

テンプレートが肥大化する場合は  
`template-parts` に分割します。

例

<?php get_template_part('template-parts/content','post'); ?>

---

# 再利用パーツの実装方針

再利用する表示パーツはテンプレートパーツ化を優先します。

テーマテンプレートから呼び出す場合

get_template_part()

を使用します。

例

<?php get_template_part('template-parts/top','blog-list'); ?>

ショートコードは必須ではありません。

以下の場合のみショートコードを検討します。

・固定ページ本文から呼び出したい  
・投稿本文から呼び出したい  
・ブロックエディタから配置したい  

---

# カスタム投稿設計

必要に応じてカスタム投稿タイプを使用します。

register_post_type()

例

post  
→ お知らせ

blog  
→ 医療コラム  
→ 施術解説  
→ 院長コラム  
→ 健康情報

分類が必要な場合は  
カスタムタクソノミーを使用します。

---

# WordPress Queryルール

query_posts() は使用しません。

以下を使用します。

WP_Query  
get_posts()

例

$args = [
'post_type' => 'post',
'posts_per_page' => 5
];

$query = new WP_Query($args);

ループ終了後は必ず

wp_reset_postdata();

を実行します。

---

# セキュリティ

出力時は必ずエスケープします。

esc_html()  
esc_attr()  
esc_url()

例

echo esc_html($title);

ユーザー入力はサニタイズします。

sanitize_text_field()  
sanitize_email()

---

# WordPress関数優先

可能な限り WordPress のテンプレート関数を使用します。

例

the_title()  
the_permalink()  
the_time()  

SQLの直接実行は最終手段とします。

---

# コーディング規約

PHP

・インデントはスペース4  
・配列は [] 記法

例

$args = [
'post_type' => 'post'
];

HTML

・インデントを整える  
・inline styleは使用しない  

CSS

BEM風命名を推奨

例

.card  
.card__title  
.card__content  

---

# パフォーマンス

不要なクエリを避けます。

必要に応じてキャッシュを利用します。

Transients API

例

set_transient('top_posts',$data,HOUR_IN_SECONDS);

---

# フロントページ実装

トップページでは  
カスタム投稿の新着一覧を表示する場合があります。

例

blog 新着5件表示

WP_Query を使用して実装します。

---

# コード修正時のルール

Claude は以下を守ること

・既存構造を壊さない  
・テンプレート階層を維持する  
・互換性を保つ  
・可読性を重視する  

---

# Claudeへの指示

Claude がコード生成する場合

・WordPress API を優先する  
・テンプレートタグを使用する  
・セキュリティを考慮する  
・可読性の高いコードを書く  
・必要に応じてコメントを書く  

---

# 禁止事項

以下は禁止

WordPress core 編集  
本番サーバー直接編集  
query_posts() 使用  
未エスケープ出力