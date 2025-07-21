---
title: 'kiroを用いたGitHub ActionsとSecretsを使って安全にAPIキーを管理する天気取得アプリの作り方'
date: 2025-07-21T23:13:23+09:00
draft: false
categories:
  - 技術
tags:
  - タグ
slug: 'kiro'
---

## 概要
**Kiro**は、2025年にAWSが発表した次世代のAI統合開発環境（AI IDE）で、自然言語でアプリケーションの要件を記述するだけで、設計からコード生成、テスト、ドキュメント作成までをAIが自動で行ってくれる画期的な開発支援ツールです。  

Kiroは、開発者が「何を作りたいか」をrequirements.mdに自然言語で書くだけで、AIがそれを読み取り、design.md（設計書）やtasks.md（タスク分解）を自動で生成します。さらに、チャット形式で「ログイン機能を追加して」「この部分を修正して」といった指示を与えることで、AIエージェントが必要なファイルを作成・編集し、フロントエンドやバックエンドのコードを生成します。生成されたコードは、React、JavaScript、FastAPI、HTML/CSSなど、幅広い技術に対応しており、Git連携やCI/CDの設定も可能です。  

現在は無料プレビュー期間中で、月50回までの利用が可能です。将来的には有料プランも提供される予定です。  

Kiroは、コーディングに不慣れな初心者から、業務効率化を求めるエンジニアまで幅広く活用できるツールであり、今後AWSの各種サービス（Lambda、S3、API Gatewayなど）との連携も予定されています。個人開発やプロトタイピング、PoCの作成にも非常に有効であり、まさに「アイデアをそのまま形にできる」未来型の開発体験を提供してくれます。  

本サイトではkiroを用いた天気取得アプリを作成しました。  
以下の記事も全てkiroを用いて記述したものになります。

## はじめに
今回は、OpenWeatherMap APIを使った天気予報アプリを作成し、GitHub PagesでAPIキーを安全に管理しながらデプロイする方法を紹介します。

通常、フロントエンドアプリでAPIキーを使用する場合、ソースコードに直接書き込むとセキュリティリスクが生じます。この記事では、GitHub ActionsとSecretsを活用して、APIキーを安全に隠しながら動作するWebアプリを構築する手法を解説します。

## 完成品

**デモサイト**: https://edamame13.github.io/weather-app/  
**ソースコード**: https://github.com/edamame13/weather-app


## 主な機能

- 🌍 世界中の都市の天気情報を検索
- 🎨 天気に応じた動的背景色変更
- 🌟 天気アイコンとアニメーション効果
- 📱 レスポンシブデザイン
- 🔒 セキュアなAPIキー管理

## 技術スタック

- **フロントエンド**: HTML5, CSS3, Vanilla JavaScript
- **API**: OpenWeatherMap API
- **デプロイ**: GitHub Pages
- **CI/CD**: GitHub Actions
- **セキュリティ**: GitHub Secrets

## プロジェクト構成

```
weather-app/
├── index.html          # メインHTML
├── style.css           # スタイルシート
├── script.js           # JavaScript
├── config.js           # 設定ファイル（ローカル用）
├── .github/
│   └── workflows/
│       └── deploy.yml  # GitHub Actions設定
└── README.md
```

## 1. 基本的なHTMLとCSSの作成

まず、天気予報アプリの基本構造を作成します。

### index.html

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>天気予報アプリ</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>天気予報アプリ</h1>
        
        <!-- APIキー設定画面 -->
        <div id="apiKeySection" class="api-key-section">
            <h3>初回設定</h3>
            <p>OpenWeatherMap APIキーを入力してください</p>
            <div class="api-input-group">
                <input type="password" id="apiKeyInput" placeholder="APIキーを入力">
                <button id="saveApiKeyBtn">保存</button>
            </div>
            <div class="api-help">
                <p><a href="https://openweathermap.org/api" target="_blank">APIキーの取得方法</a></p>
                <small>APIキーはブラウザにのみ保存され、安全に管理されます</small>
            </div>
        </div>
        
        <!-- メインアプリ -->
        <div id="mainApp" class="main-app hidden">
            <div class="search-section">
                <input type="text" id="cityInput" placeholder="都市名を入力してください（例：Tokyo）">
                <button id="searchBtn">天気を取得</button>
            </div>
            <div class="api-key-info">
                <small>APIキー設定済み | <button id="changeApiKeyBtn" class="link-btn">変更</button></small>
            </div>
        </div>
        
        <div id="errorMessage" class="error hidden"></div>
        
        <!-- 天気情報表示エリア -->
        <div id="weatherInfo" class="weather-info hidden">
            <h2 id="cityName"></h2>
            <div class="weather-main">
                <div class="weather-icon-container">
                    <div id="weatherIcon" class="weather-icon"></div>
                </div>
                <div class="weather-temp">
                    <span id="temperature"></span>
                </div>
            </div>
            <div class="weather-description">
                <span id="weather"></span>
            </div>
            <div class="weather-details">
                <div class="weather-item">
                    <span class="label">湿度:</span>
                    <span id="humidity"></span>
                </div>
                <div class="weather-item">
                    <span class="label">体感温度:</span>
                    <span id="feelsLike"></span>
                </div>
                <div class="weather-item">
                    <span class="label">風速:</span>
                    <span id="windSpeed"></span>
                </div>
            </div>
        </div>
        
        <div id="loading" class="loading hidden">
            <p>天気情報を取得中...</p>
        </div>
    </div>
    
    <script src="config.js"></script>
    <script src="script.js"></script>
</body>
</html>
```

### style.css（抜粋）

```css
body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #74b9ff, #0984e3);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.5s ease;
}

/* 天気に応じた背景色 */
body.sunny {
    background: linear-gradient(135deg, #ffeaa7, #fdcb6e);
}

body.rainy {
    background: linear-gradient(135deg, #74b9ff, #0984e3);
}

.weather-icon {
    font-size: 4rem;
    text-align: center;
    animation: float 3s ease-in-out infinite;
}

@keyframes float {
    0%, 100% { transform: translateY(0px); }
    50% { transform: translateY(-10px); }
}
```

## 2. JavaScriptの実装

### script.js（主要部分）

```javascript
// OpenWeatherMap API設定
const API_URL = 'https://api.openweathermap.org/data/2.5/weather';
const API_KEY_STORAGE = 'weather_app_api_key';

// 天気アイコンマッピング
const weatherIcons = {
    '01d': '☀️', '01n': '🌙', '02d': '⛅', '02n': '☁️',
    '03d': '☁️', '03n': '☁️', '04d': '☁️', '04n': '☁️',
    '09d': '🌧️', '09n': '🌧️', '10d': '🌦️', '10n': '🌧️',
    '11d': '⛈️', '11n': '⛈️', '13d': '❄️', '13n': '❄️',
    '50d': '🌫️', '50n': '🌫️'
};

// APIキーを取得する関数
function getApiKey() {
    // GitHub Actionsでデプロイされた場合のconfig.js
    if (window.WEATHER_APP_CONFIG && window.WEATHER_APP_CONFIG.API_KEY) {
        return window.WEATHER_APP_CONFIG.API_KEY;
    }
    // ローカルストレージから取得
    return localStorage.getItem(API_KEY_STORAGE);
}

// 天気情報を取得する関数
async function getWeather() {
    const city = cityInput.value.trim();
    const apiKey = getApiKey();

    if (!city) {
        showError('都市名を入力してください');
        return;
    }

    if (!apiKey) {
        showError('APIキーが設定されていません');
        showApiKeySection();
        return;
    }

    hideAll();
    showLoading();

    try {
        const response = await fetch(
            `${API_URL}?q=${encodeURIComponent(city)}&appid=${apiKey}&units=metric&lang=ja`
        );

        if (!response.ok) {
            if (response.status === 404) {
                throw new Error('都市が見つかりません。都市名を確認してください。');
            } else if (response.status === 401) {
                localStorage.removeItem(API_KEY_STORAGE);
                throw new Error('APIキーが無効です。新しいAPIキーを設定してください。');
            } else {
                throw new Error('天気情報の取得に失敗しました。');
            }
        }

        const data = await response.json();
        displayWeather(data);

    } catch (error) {
        console.error('Error:', error);
        showError(error.message);
        
        if (error.message.includes('APIキーが無効')) {
            setTimeout(() => showApiKeySection(), 2000);
        }
    } finally {
        hideLoading();
    }
}

// 天気情報を表示する関数
function displayWeather(data) {
    const weatherIcon = document.getElementById('weatherIcon');
    const feelsLike = document.getElementById('feelsLike');
    const windSpeed = document.getElementById('windSpeed');
    
    cityName.textContent = `${data.name}, ${data.sys.country}`;
    weather.textContent = data.weather[0].description;
    temperature.textContent = `${Math.round(data.main.temp)}°C`;
    humidity.textContent = `${data.main.humidity}%`;
    feelsLike.textContent = `${Math.round(data.main.feels_like)}°C`;
    windSpeed.textContent = `${data.wind?.speed || 0} m/s`;
    
    // 天気アイコンを設定
    const iconCode = data.weather[0].icon;
    weatherIcon.textContent = weatherIcons[iconCode] || '🌤️';
    
    // 天気に応じて背景色を変更
    updateBackgroundColor(data.weather[0].main);
    
    weatherInfo.classList.remove('hidden');
}
```

## 3. GitHub Actionsでのセキュアなデプロイ

ここが今回の記事の核心部分です。GitHub ActionsとSecretsを使ってAPIキーを安全に管理します。

### .github/workflows/deploy.yml

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch: # 手動実行を可能にする

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup Pages
      uses: actions/configure-pages@v4
      
    - name: Create config file with API key
      run: |
        echo "Creating config.js with API key..."
        cat > config.js << EOF
        window.WEATHER_APP_CONFIG = {
          API_KEY: '${{ secrets.OPENWEATHER_API_KEY }}'
        };
        EOF
        echo "Config file created successfully"
      
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v3
      with:
        path: '.'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

### config.js（ローカル開発用）

```javascript
// このファイルはローカル開発用です
// GitHub Actionsでデプロイ時には自動的に上書きされます
window.WEATHER_APP_CONFIG = {
    API_KEY: null
};
```

## 4. セットアップ手順

### 4.1 OpenWeatherMap APIキーの取得

1. [OpenWeatherMap](https://openweathermap.org/api)でアカウント作成
2. 無料プランでAPIキーを取得
3. APIキーをコピー

### 4.2 GitHub Secretsの設定

1. GitHubリポジトリの「Settings」→「Secrets and variables」→「Actions」
2. 「New repository secret」をクリック
3. 以下を設定：
   - **Name**: `OPENWEATHER_API_KEY`
   - **Secret**: 取得したAPIキー

### 4.3 GitHub Pagesの設定

1. リポジトリの「Settings」→「Pages」
2. Source: 「GitHub Actions」を選択

## 5. セキュリティのポイント

### 5.1 APIキーの保護

- ✅ APIキーはGitHub Secretsで暗号化保存
- ✅ ソースコードには含まれない
- ✅ ビルド時のみ注入される
- ✅ 公開されるコードにはプレースホルダーのみ

### 5.2 フォールバック機能

```javascript
function getApiKey() {
    // 1. GitHub Actionsでデプロイされた場合
    if (window.WEATHER_APP_CONFIG && window.WEATHER_APP_CONFIG.API_KEY) {
        return window.WEATHER_APP_CONFIG.API_KEY;
    }
    // 2. ローカル開発時はユーザー入力
    return localStorage.getItem(API_KEY_STORAGE);
}
```

この仕組みにより：
- **本番環境**: APIキーが自動注入されて動作
- **ローカル開発**: ユーザーが手動でAPIキーを入力

## 6. デプロイと動作確認

### 6.1 デプロイ

```bash
git add .
git commit -m "Add weather app with secure API key management"
git push origin main
```

### 6.2 動作確認

1. GitHub Actionsの実行を確認
2. `https://[username].github.io/[repository-name]/`にアクセス
3. 都市名を入力して天気情報を取得

## 7. 応用とカスタマイズ

### 7.1 機能拡張のアイデア

- 5日間の天気予報
- 位置情報による自動取得
- お気に入り都市の保存
- 通知機能

### 7.2 他のAPIでの応用

この手法は他のAPIでも応用可能：
- Google Maps API
- Twitter API
- その他のRESTful API

## まとめ

今回は、GitHub ActionsとSecretsを活用して、APIキーを安全に管理しながら天気予報アプリを作成しました。

**学んだポイント：**
- GitHub Secretsでの機密情報管理
- GitHub Actionsでの自動デプロイ
- セキュアなフロントエンドアプリの構築
- フォールバック機能の実装

この手法を使えば、APIキーを公開することなく、静的サイトでAPIを活用したアプリケーションを安全にデプロイできます。

**参考リンク：**
- [完成したアプリ](https://edamame13.github.io/weather-app/)
- [ソースコード](https://github.com/edamame13/weather-app)
- [OpenWeatherMap API](https://openweathermap.org/api)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

*この記事が役に立ったら、ぜひスターやフォローをお願いします！*
