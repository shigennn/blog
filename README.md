# 概要

個人ブログ  
https://vshige.github.io/blog/

## 使わせていただいた template

- [Next.js + GitHub Pages のブログテンプレートを作った](https://zenn.dev/subt/articles/957bd5d01485e1)
- [Next.js で作ったブログを GitHub Pages にデプロイする](https://jamband.github.io/blog/2021/08/deploy-nextjs-app-to-github-pages/)

## 変更点と変更箇所

- ブログタイトル
  src/components/features/app/Header/Header.tsx | line16

- サイト名
  src/components/features/app/Seo/Seo.tsx | line13 to 19

- PAT 設定
  .github/workflows/deploy.yml | line41

- basePath
  next.config.js | line4

- Author Profile

  - Author name
    src/components/features/app/Profile/Profile.tsx | line13
  - Author disc
    src/components/features/app/Profile/Profile.tsx | line16

- SNS
  src/config/sns.tsx

- DarkMode を Defualt にする
  src/hooks/useDarkMode.ts | line9 | useState(true)
