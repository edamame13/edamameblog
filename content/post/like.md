---
title: 「Cloud Functions For Firebase」を用いて"いいね機能"を実装した
date: 2025-07-17T00:03:14+09:00
draft: false
categories:
  - 技術
tags:
  - タグ
slug: like
---

## 概要
本記事は静的サイト（例：Hugo + GitHub Pages）に「いいねボタン」を追加し、クリック数をFirebaseで記録・取得する機能を実装した。

- **フロントエンド**：Hugo（GitHub Pages）
- **バックエンド**：Firebase Cloud Functions（JavaScript）
- **データストア**：Cloud Firestore

上記が現在の本ブログサイトの構成となっている。

## 完成イメージ

- ページごとに「いいね！」が押せる
- 押すと数が増える
- ローカルストレージで連打防止（20回まで）

## Firebase プロジェクト作成

1. [Firebase Console](https://console.firebase.google.com/) にアクセスし、新規プロジェクト作成
2. Blaze プラン（従量課金）にアップグレード  
   ※ Functions 第2世代・Artifact Registryの使用に必須  
   ※ 通常利用では無料枠内に収まることが多いです

## Firebase CLI を使って Functions 初期化
```bash
npm install -g firebase-tools
firebase login
firebase init functions
```

	•	言語：JavaScript
	•	ESLint：Yes（任意）
	•	npm install：Yes（推奨）

## Functionsコードを実装（functions/index.js）

```functions/index.js
const functions = require("firebase-functions");
const admin = require("firebase-admin");

admin.initializeApp();
const db = admin.firestore();

exports.likeHandler = functions.https.onRequest(async (req, res) => {
  // CORS設定
  res.set("Access-Control-Allow-Origin", "*");
  res.set("Access-Control-Allow-Methods", "GET, POST");
  res.set("Access-Control-Allow-Headers", "Content-Type");
  if (req.method === "OPTIONS") return res.status(204).send("");

  const articleId = req.path.split("/").pop();
  const docRef = db.collection("likes").doc(articleId);

  try {
    if (req.method === "GET") {
      const doc = await docRef.get();
      const count = doc.exists ? doc.data().count : 0;
      return res.status(200).json({ likes: count });
    }

    if (req.method === "POST") {
      const doc = await docRef.get();
      const current = doc.exists ? doc.data().count : 0;
      await docRef.set({ count: current + 1 });
      return res.status(200).json({ likes: current + 1 });
    }

    return res.status(405).send("Method Not Allowed");
  } catch (error) {
    console.error("Firestore Error:", error);
    return res.status(500).send("Internal Server Error");
  }
});
```

- Lint エラー修正（必要に応じて）
npm run lint -- --fix

## Functionをデプロイ
firebase deploy --only functions

	•	成功すると次のような URL が表示される：
https://us-central1-<your-project-id>.cloudfunctions.net/likeHandler


## フロントエンドのHTML+JS

```layouts/partials/like.html
<div class="like-container">
  <button id="like-button" title="いいね！">
    <svg xmlns="http://www.w3.org/2000/svg" width="38" height="38" viewBox="0 0 24 24" fill="#ea5550">
      <path d="..."></path>
    </svg>
    <span id="like-count">0</span>
  </button>
</div>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const apiEndpoint = "https://us-central1-<your-project-id>.cloudfunctions.net/likeHandler";
  const articleId = location.pathname;
  const likeButton = document.getElementById("like-button");
  const likeCountSpan = document.getElementById("like-count");
  const LIKE_LIMIT = 10;
  const userLikeCountKey = `user_like_count_${articleId}`;

  fetch(`${apiEndpoint}/${encodeURIComponent(articleId)}`)
    .then(res => res.json())
    .then(data => likeCountSpan.textContent = data.likes || 0)
    .catch(err => console.error("いいね数取得失敗:", err));

  const currentUserLikes = parseInt(localStorage.getItem(userLikeCountKey) || "0");
  if (currentUserLikes >= LIKE_LIMIT) {
    likeButton.disabled = true;
    likeButton.classList.add("limit-reached");
  }

  likeButton.addEventListener("click", () => {
    let userLikes = parseInt(localStorage.getItem(userLikeCountKey) || "0");
    if (userLikes >= LIKE_LIMIT) return;

    userLikes++;
    localStorage.setItem(userLikeCountKey, userLikes);
    likeCountSpan.textContent = parseInt(likeCountSpan.textContent) + 1;

    fetch(`${apiEndpoint}/${encodeURIComponent(articleId)}`, {
      method: "POST",
    })
    .then(res => res.json())
    .then(data => likeCountSpan.textContent = data.likes)
    .catch(err => {
      likeCountSpan.textContent = parseInt(likeCountSpan.textContent) - 1;
      console.error("いいね送信失敗:", err);
    });

    if (userLikes >= LIKE_LIMIT) {
      likeButton.disabled = true;
      likeButton.classList.add("limit-reached");
    }
  });
});
</script>
```

## 参考
[Firebase Cloud Functions公式ドキュメント](https://firebase.google.com/docs/functions?hl=ja)
[Cloud Firestoreドキュメント](https://firebase.google.com/docs/firestore?hl=ja)
[Blazeプランの無料枠](https://firebase.google.com/pricing)
[とても参考にしているサイト](https://blog.bokukoha.dev/)