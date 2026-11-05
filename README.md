# Atlory mobile build resources

この公開リポジトリには手動ビルド用ワークフローと、iOS/Androidの無料ビルド手順だけを置いています。AtloryのソースやIPAはGitHubの公開ファイル・Actions artifactに置きません。ワークフローは非公開の `Idearca/atlory-mobile` から指定ブランチを一時取得し、GitHubのmacOS runner上でローカルビルドしたIPAをExpoプロジェクトへアップロードします。共有リンクは公開ログに出さない設定です。runner上で非公開ソースを実行するため、リスクをゼロにはできません。

無料枠の条件、Android APK、iOS Ad Hoc、TestFlight、GitHub Actionsの使い分けは [FREE_BUILD_GUIDE.md](./FREE_BUILD_GUIDE.md) を参照してください。

## 初回設定

`Idearca/atlory-ios-builder` の Settings → Secrets and variables → Actions に次のRepository secretsを登録します。

- `ATLORY_READ_TOKEN`: `Idearca/atlory-mobile` だけにアクセスできるfine-grained token。権限はContents: readのみ。
- `EXPO_TOKEN`: Atlory EASプロジェクトにアクセスできるExpo access token。

トークン値はリポジトリのファイルやチャットに書かないでください。ビルド後はExpo dashboardのAtloryプロジェクトからIPAを取得します。Expo側でInternal buildへの未認証アクセスを無効にしてから実行してください。

## ビルド

Actions → Atlory iOS device build → Run workflow を実行します。macOS runner上でのビルドは公開リポジトリの標準runner利用枠内で無料です。ビルド時間の上限は60分です。

iPhoneへ配布する `device-test` IPAはAd Hoc署名です。Apple Developer Programの有料会員資格と、登録済みiPhoneが必要です。TestFlightも同じ年会費が必要です。TestFlightは年会費以外のビルドごとのApple料金はありません。
