# highlowrush-app プロジェクト一式（Capacitor + iOSプロジェクト）

`highlowrush-app.zip` の中身は、既存の`highlowrush-app`リポジトリ（すでにprivacy-policy.html / terms-of-service.htmlを配置済み）にそのまま統合する想定です。

## 中身

```
highlowrush-app/
├─ .gitignore
├─ capacitor.config.ts        # appId: com.yutaXXX.highlowrush
├─ package.json
├─ package-lock.json
├─ privacy-policy.html        # 既にGitHub Pagesで公開中のものと同一内容
├─ terms-of-service.html      # 既にGitHub Pagesで公開中のものと同一内容
├─ www/
│   └─ index.html             # プロトタイプ本体（そのままCapacitorのwebDirへ）
└─ ios/
    └─ App/                   # Xcodeプロジェクト（Capacitor 8系、SPM構成・Podfileなし）
```

## 仕様書に合わせて調整済みの項目

- `IPHONEOS_DEPLOYMENT_TARGET` = 17.0（min_iOS 17）
- `TARGETED_DEVICE_FAMILY` = "1"（iPhoneのみ、iPad非対応）
- `MARKETING_VERSION` = 1.0.0
- `UISupportedInterfaceOrientations` = Portraitのみ（縦画面固定・iPad用の設定は削除）
- `ITSAppUsesNonExemptEncryption` = false
- `GADApplicationIdentifier` = `ca-app-pub-8174756915786797~3111467770`（AdMobアプリID）
- `NSUserTrackingUsageDescription` は追加していません（仕様上ATTプロンプトなし）

## まだ入っていないもの（次のステップで対応）

- AdMob SDK（`@capacitor-community/admob` 等のプラグイン導入）
- Haptics / Share 等のCapacitor公式プラグイン（現状はWeb標準API＝`navigator.vibrate`/`navigator.share`のフォールバックのまま）
- アプリアイコン・起動画面画像（`Assets.xcassets`は空のプレースホルダーのまま）
- 左端スワイプ戻るの無効化（Capacitorのデフォルトテンプレートはナビゲーションスタックを持たない単一ビュー構成のため、通常は元々スワイプ戻るが機能しません。念のためビルド後に実機で確認をおすすめします）

## 既存リポジトリへの反映手順

お使いの環境（Windows、Mac無し）から、GitHub Desktop等で`highlowrush-app`をクローンしている前提で書きます。

1. `highlowrush-app.zip` を展開する
2. 展開してできたファイル・フォルダ一式を、ローカルの`highlowrush-app`リポジトリのフォルダにコピー（`privacy-policy.html`・`terms-of-service.html`は既存のものと同一内容なので上書きしてOKです）
3. GitHub Desktop（またはお使いのGitクライアント）で変更を確認し、コミット＆プッシュ
   - コミットメッセージ例: `Add Capacitor/iOS project scaffold`

## プッシュ後の確認

1. GitHub上で`highlowrush-app`リポジトリを開き、`ios/App/App.xcodeproj`等がアップロードされていることを確認
2. 次のステップ（Codemagicのワークフロー設定）へ進みます
