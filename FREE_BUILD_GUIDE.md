# Atloryの無料ビルド手順（iOS / Android）

確認日: 2026-11-05

この手順は、Atloryの既存EAS設定と公開済みiOSビルド用GitHub Actionsに合わせています。ここでいう「無料」は、**新しいビルドサービス契約や追加のビルド料金を使わないこと**です。無制限の無料ビルドを保証する意味ではありません。

## 料金と配布方法

| 目的 | 無料で使う経路 | 条件 |
| --- | --- | --- |
| Androidを自分やテスターの端末へ配る | EAS FreeでAPKを作成 | Free枠は月15ビルド。Play Console不要 |
| Androidを開発中に端末・エミュレーターで動かす | Android Studioでローカル実行 | EAS枠は使わない。Android StudioとSDKが必要 |
| iPhoneを登録済み端末へ直接配る | EAS FreeでAd Hoc IPAを作成 | Free枠は月15ビルド。Apple Developer Programの有料会員と端末登録が必要 |
| iOSの直接配布ビルドを追加で作る | 公開リポジトリのGitHub Actions | 標準macOS runnerは公開repoで無料。既存の秘密情報と登録済みiPhoneが必要 |
| iPhoneへTestFlightで配る | EAS Freeでビルドし、App Store Connectへ提出 | Apple Developer Programの有料会員が必要。会費は継続して発生 |

ExpoのFree枠は低優先度で、毎月15回のAndroidビルドと15回のiOSビルドです。月の枠を使い切ってもFreeプランに超過料金は発生しませんが、翌月のリセットまで新しいクラウドビルドを開始できません。実行前にExpo DashboardのBillingで現在のプランを確認してください。Free以外のプランは追加使用料が発生する場合があります。

AppleのDeveloper Program会費はビルドごとの料金ではありませんが、TestFlightやiPhoneへのAd Hoc配布に必要です。AtloryのApple Developerアカウントはすでに有効です。無料のPersonal TeamだけでXcodeから端末へ入れる場合はプロビジョニングが7日で期限切れになり、TestFlightには配布できません。

## 共通準備

Windows PowerShellで非公開のAtloryソースリポジトリを開きます。ビルドのためにソースを公開リポジトリへコピーする必要はありません。

~~~powershell
pnpm install --frozen-lockfile
pnpm dlx eas-cli login
~~~

Expoアカウントは既存のAtlory EAS projectを選びます。ローカルの.env値はクラウドビルドへ自動で渡りません。必要な公開設定はEASの対象環境に登録されていることを確認してください。Supabase service-role keyなどの管理者秘密鍵をアプリ、GitHub、EASの公開環境変数へ入れないでください。

## Android APKを無料で作る

この方法はGoogle Playを使わず、APKを端末へ直接インストールする場合に使います。

1. Atloryソースのルートで次を実行します。

   ~~~powershell
   pnpm run build:android:preview
   ~~~

2. EASのビルドページで対象コミットを確認し、StatusがFINISHEDになるまで待ちます。
3. 表示されたインストールリンクをAndroid端末で開き、APKをダウンロードしてインストールします。Androidが未確認アプリのインストールを確認した場合は、配布元を確認してから許可します。

android-previewは、既存設定ではdevelopment環境を使う内部配布APKです。APKはPlay Store用のAABとは別で、直接インストールできます。Google Playで公開するには別途Play Consoleの設定とAABが必要です。

EASの無料枠を使わずローカルで開発版を動かすには、Android StudioとAndroid SDKを設定し、端末またはエミュレーターを起動して次を実行します。

~~~powershell
pnpm run android
~~~

これは開発用アプリをビルドして実行する方法です。共有用の署名済みAPKを作る手順とは異なります。

## iPhoneへ直接インストールする

登録済みiPhoneで確認するためのAd Hoc IPAを作ります。

1. まだ端末登録をしていない場合は、iPhoneのUDIDを登録します。

   ~~~powershell
   pnpm dlx eas-cli device:create
   ~~~

2. Atloryソースのルートでビルドします。

   ~~~powershell
   pnpm run build:ios:device-test
   ~~~

3. EASがFINISHEDになったら、iPhoneでインストールリンクを開いてIPAをインストールします。

Appleの署名では、ビルド時点で登録されている端末だけがAd Hoc IPAをインストールできます。新しいiPhoneを追加した場合は端末登録を更新して再ビルドします。この方法にもApple Developer Program会員資格が必要です。

## TestFlightへ無料枠で提出する

EASのビルド無料枠を使い、AppleのTestFlightへ配布する手順です。Apple Developer Programの会費は別途必要ですが、TestFlightビルドごとのApple料金はありません。

~~~powershell
pnpm run build:ios:testflight
pnpm run submit:ios:testflight
~~~

1. 1行目がiOSのストア配布用ビルドを作ります。Atloryの現在のtestflight-review profileでは審査用デモが有効になります。
2. ビルドがFINISHEDになったら、2行目でApp Store Connectへ提出します。
3. App Store ConnectでAppleの処理完了を待ち、内部テストグループへ割り当てます。初回の外部テストにはAppleのBeta App Reviewが必要です。

審査用デモを含まないproduction profileのビルドを作る場合は、1行目の代わりに次を使います。

~~~powershell
pnpm dlx eas-cli build --platform ios --profile production
~~~

提出前に、App Store Connectの対象アプリ・バージョン・ビルド番号を必ず確認します。ビルド成功はTestFlightへの配布完了を意味しません。

## iOSの追加ビルドをGitHub Actionsで作る

EAS無料枠の外でiOS Ad Hocビルドが必要な場合は、既存の公開リポジトリの手動workflowを使えます。

1. [Atlory iOS builder のActions](https://github.com/Idearca/atlory-ios-builder/actions)を開きます。
2. 「Atlory iOS device build」を選び、「Run workflow」を実行します。
3. 成功後、ExpoのAtlory projectでビルド結果を開き、iPhoneへインストールします。workflowはdevice-test用IPAを作ります。

このworkflowは公開リポジトリ上の標準macOS runnerで非公開のAtloryソースを取得して実行し、IPAをExpoへアップロードします。GitHubは公開リポジトリの標準runnerを無料で提供しています。workflowの実行上限は60分です。ExpoへのアップロードとApple署名にはそれぞれのアカウント・設定が必要です。

workflowには、既存のGitHub Actions secretsとしてATLORY_READ_TOKENとEXPO_TOKENが必要です。トークン値をソース、workflowログ、Markdownへ書かないでください。ATLORY_READ_TOKENはAtlory非公開repoだけの読み取り権限にし、信頼できるブランチのコードだけを実行してください。公開runner上で非公開ソースを実行するため、リスクが完全になくなる方法ではありません。

このworkflowはAd Hoc IPA用です。登録していないiPhoneにはインストールできません。TestFlightへ出す場合は、上記のstore配布profileとApp Store Connectへの提出を使います。

## 無料枠を無駄にしない使い方

- EASクラウドビルドは、ネイティブ依存関係・権限・ネイティブ設定を変えたときに使います。
- JavaScriptやTypeScriptだけの修正は、既存Development ClientとMetroを使って確認できる場合があります。
- ビルド結果では、FINISHED、platform、profile、version、コミットを確認します。
- ビルドが成功しただけで、地図・認証・投稿・通知・課金などが端末で動くとは扱いません。必要なEAS環境変数と実機動作を個別に確認します。
- AndroidのEAS local buildはWindowsで公式サポートされていません。Android StudioでのExpoローカル実行とEAS cloud buildを混同しないでください。iOSのXcodeローカルビルドにはMacが必要です。

## 公式資料

- [Expo EAS Pricing](https://expo.dev/pricing)
- [Expo: EAS plans and monthly free quota](https://docs.expo.dev/billing/plans/)
- [Expo: Free plan quota exhaustion](https://docs.expo.dev/billing/faq/)
- [Expo: Create and install Android APKs](https://docs.expo.dev/build-reference/apk/)
- [Expo: Internal distribution and Ad Hoc requirements](https://docs.expo.dev/build/internal-distribution/)
- [Expo: TestFlight distribution](https://docs.expo.dev/submit/testflight/)
- [Expo: Local builds and Windows support](https://docs.expo.dev/build-reference/local-builds/)
- [GitHub: Choose a runner and public repository billing](https://docs.github.com/en/actions/how-tos/write-workflows/choose-where-workflows-run/choose-the-runner-for-a-job)
- [Apple: Personal Team signing limits](https://developer.apple.com/help/account/basics/about-your-developer-account/)
- [Apple: Developer Program enrollment and membership](https://developer.apple.com/help/account/membership/program-enrollment/)
