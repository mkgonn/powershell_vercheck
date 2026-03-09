# PowerShell 7 + Linux移植 互換性調査レポート

**調査日**: 2026-03-06
**現行環境**: Windows / PowerShell 5.1.20348.2849
**移行先候補**: PowerShell 7.x (pwsh) / Linux
**調査手法**: Microsoft公式ドキュメント、GitHub Issues、PowerShell Gallery、コミュニティブログ、複数リサーチエージェント並列調査

---

## エグゼクティブサマリー

PowerShell 7への移行では10モジュール中8モジュールが対応済み（うちMSAL.PSはアーカイブ済で早期代替推奨）。Linux移植では6モジュールが完全対応（うちMSAL.PSは動作宣言済だが実運用未検証）、2モジュールが部分対応、2モジュールが完全非対応。

**最大のブロッカー**: `Microsoft.Online.SharePoint.PowerShell` と `Microsoft.PowerApps.Administration.PowerShell` の2つ。いずれも.NET Framework依存のため、PS7ネイティブ動作不可・Linux完全非対応。

**懸念されていたTeams**: PS7.2+で問題なく動作。Linuxでも基本操作は可能。

---

## 総合判定マトリクス


| モジュール                                         | バージョン       | PS7対応      | Linux対応       | 総合判定        |
| --------------------------------------------- | ----------- | ---------- | ------------- | ----------- |
| Microsoft.Graph                               | 1.27.0      | 完全対応       | 完全対応          | GO          |
| Az                                            | 8.0.0       | 完全対応       | 完全対応          | GO          |
| SqlServer                                     | 21.1.18256  | 対応         | 部分対応(57%)     | 要検証         |
| PnP.PowerShell                                | 1.11        | 完全対応        | 完全対応          | GO（長期的にはv3推奨） |
| Microsoft.Online.SharePoint.PowerShell        | 16.0.27011.12008 | 非ネイティブ     | 非対応           | BLOCK・要代替   |
| MicrosoftTeams                                | 6.1.0       | 対応(7.2+、v7.3.1推奨) | 部分対応     | 要検証         |
| ExchangeOnlineManagement                      | 3.1.0       | 対応(7.0.3+) | 部分対応(REST時OK) | 要検証         |
| Microsoft.PowerApps.Administration.PowerShell | 2.0.168     | PS5.1専用    | 非対応           | BLOCK・要代替   |
| MicrosoftPowerBIMgmt                          | 1.2.1111    | 完全対応       | 完全対応          | GO          |
| MSAL.PS                                       | 4.37.0.0    | 動作するがアーカイブ済 | 動作宣言済(実運用未検証) | 非推奨・要代替     |


---

## 各モジュール詳細分析

### 1. Microsoft.Graph 1.27.0 — PS7・Linux完全対応

- **PS7**: PowerShell 7以降が**推奨バージョン**。PS5.1でも動作するがPS7推奨
- **Linux**: Windows/macOS/Linux全プラットフォーム完全対応
- **.NET**: .NET Standard 2.0ベース（クロスプラットフォーム）
- **REST APIベース**でGraph APIを呼び出すためWindows固有API依存がゼロ
- **注意**: v2.26以降で.NET 8依存に変更。Azure Automation（PS7.1/7.2環境）では動作しない問題が報告されている。PS7.4以上を使えば問題なし
- **判定**: そのまま移行可能。最も安心

**参考リンク**:

- [Microsoft Graph PowerShell SDK overview - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview?view=graph-powershell-1.0)
- [Install the Microsoft Graph PowerShell SDK - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/microsoftgraph/installation?view=graph-powershell-1.0)

---

### 2. Az 8.0.0 — PS7・Linux完全対応

- **PS7**: PowerShell 7.0.6 LTS以上が**推奨**。全プラットフォーム対応
- **Linux**: 完全対応。Azure Cloud Shell（Linuxベース）でも標準搭載されており実績豊富
- **.NET**: .NET Standard 2.0ベース。AzureRM（Windows専用）の後継としてLinux対応を明確に設計
- **注意**: Az 8.0 → 最新版への更新推奨（APIバージョン対応）
- **判定**: そのまま移行可能

**参考リンク**:

- [Introducing the Azure Az PowerShell module - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az)
- [How to install Azure PowerShell - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/azure/install-azure-powershell)

---

### 3. SqlServer 21.1.18256 — PS7対応/Linux部分対応

- **PS7**: PowerShell 5.0以上対応。PS7で動作するが、`Invoke-Sqlcmd`がPS7で認識されないケースが複数報告されている
- **Linux**: **109コマンド中62コマンドのみ利用可能**（約57%）
- **使えなくなるもの**: SQL Agent関連、SQLPS互換コマンド、Windows認証依存コマンド、SMO（SQL Server Management Objects）のWindows専用機能
- **使えるもの**: 純粋なT-SQL操作（`Invoke-Sqlcmd`経由）はLinuxでも動作
- **代替**: `dbatools`（700+ cmdlet、完全クロスプラットフォーム、コミュニティ製）が推奨
- **判定**: 基本的なクエリ実行は可能だが、管理系コマンドに制限あり

**参考リンク**:

- [Manage SQL Server on Linux with PowerShell - Microsoft Learn](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-manage-powershell-core)
- [Download SQL Server PowerShell Module - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/sql-server/download-sql-server-ps-module)
- [dbatools - SQL Server automation with PowerShell](https://dbatools.io/)
- [SQLServerPSModule Issue #72 - GitHub](https://github.com/microsoft/SQLServerPSModule/issues/72)

---

### 4. PnP.PowerShell 1.11 — PS7・Linux対応（v1.11で動作可、長期的にはv3推奨）

- **PS7**: **v1.11はPS7でそのまま動作する**。v3へのアップグレードは必須ではない
- **Linux**: クロスプラットフォーム設計。Windows/macOS/Linux完全対応。Raspberry Pi・Azure Functionsでの動作実績もあり
- **.NET**: v1.x系は.NET Standard/.NET Core、v3系は.NET 8ベース

#### バージョン別PS対応状況

| バージョン | PS5.1 | PS7.0-7.3 | PS7.4+ | Linux |
| ------- | ----- | -------- | ------ | ----- |
| v1.11（現行） | 一部動作 | 動作する | 動作する | 動作する |
| v1.12（5.1対応の最終版） | 動作する | 動作する | 動作する | 動作する |
| v2.x | 非対応 | 動作する | 動作する | 動作する |
| v3.x（最新） | 非対応 | 非対応 | PS7.4.6+必須 | 動作する |

#### 将来的にv3へ移行する場合の破壊的変更


| 変更内容                          | 影響                                   |
| ----------------------------- | ------------------------------------ |
| `-SPOManagementShell` パラメータ削除 | 独自Entra IDアプリの登録が必須に                 |
| `-WebLogin` 認証モード削除           | `-Interactive` or `-DeviceLogin` に変更 |
| .NET 8.0ベースに変更                | PS 5.1/7.3以下は動作不可                    |
| 一部コマンドレット削除・名称変更              | スクリプト修正が必要                           |


- **判定**: v1.11のままPS7・Linuxへ移行可能。ただしv1.x系は今後M365側のAPI変更で動かなくなるリスクがあるため、長期的にはv3への移行を推奨

**参考リンク**:

- [PnP PowerShell公式ドキュメント](https://pnp.github.io/powershell/)
- [PnP PowerShell v3 upcoming changes](https://pnp.github.io/blog/pnp-powershell/pnp-powershell-v3-0-0-preview/)
- [Migration Guide v2→v3 - GitHub](https://github.com/pnp/powershell/blob/dev/MIGRATE-2.0-to-3.0.md)

---

### 5. Microsoft.Online.SharePoint.PowerShell 16.0.27011.12008 — 最大の問題モジュール

- **PS7**: **ネイティブ非対応**。`Import-Module -UseWindowsPowerShell` で動作可能だが、これはWindows上でのみ使える互換レイヤー。PSEdition_Coreタグは付与されておらず、PSEdition_Desktop のまま
- **Linux**: **完全に非対応**。UseWindowsPowerShell互換レイヤーはWindowsのみ。Azure Cloud Shell（Linux）でも非対応と公式Q&Aで確認
- **技術的理由**: .NET Framework 4.x依存、WinRM依存（v16.0.27011.12008でも変更なし）

#### 16.0.8029.0 → 16.0.27011.12008 での主な変更

| 変更カテゴリ | 内容 |
| --- | --- |
| 認証プロトコル | IDCRL → **OAuth（モダン認証）に移行**。16.0.25814.12000以降で`Connect-SPOService`が自動的にOAuthを使用 |
| 新cmdlet | バージョン管理ポリシー関連（`Set-SPOListVersionPolicy`等）追加 |
| ランタイム互換性 | **変更なし**（Windows + PS 5.1が主対象のまま） |

> **重要**: IDCRL認証は**2026年5月1日に廃止予定**（MC1028318）。旧バージョン（16.0.8029.0等）を使い続けると接続不能になるため、このバージョンへの更新は必須

#### UseWindowsPowerShell互換レイヤーの制限


| 制限事項             | 詳細                     |
| ---------------- | ---------------------- |
| Windows専用        | Linux/macOSでは使用不可      |
| オブジェクトのメソッドが使えない | シリアライズされるため            |
| パフォーマンス低下        | バックグラウンドでPS5.1セッションが起動 |
| 一部コマンドの制限        | プロキシ経由のためフル機能が使えない場合あり |


#### 代替案


| 代替手段                    | カバー範囲                   | 備考                       |
| ----------------------- | ----------------------- | ------------------------ |
| **PnP.PowerShell v3**   | SharePoint管理の大部分をカバー    | 最も推奨。クロスプラットフォーム完全対応     |
| **Microsoft Graph API** | SharePoint Settings API | 機能が限定的（2022年登場以降あまり進展なし） |
| **SharePoint REST API** | 直接呼び出し                  | 低レベル操作が必要                |


- **判定**: Linux移植時は必ずPnP.PowerShellに置き換えが必要

**参考リンク**:

- [How to use Microsoft.Online.SharePoint.PowerShell with PowerShell 7 - Koskila.net](https://www.koskila.net/how-to-use-microsoft-online-sharepoint-powershell-with-powershell-7/)
- [SharePoint Online PowerShell OAuth upgrade - office365itpros.com](https://office365itpros.com/2025/03/14/sharepoint-online-powershell-oauth/)
- [Legacy SharePoint Authentication (IDCRL) Is Retiring - TechCommunity](https://techcommunity.microsoft.com/blog/microsoftmissioncriticalblog/legacy-sharepoint-authentication-idcrl-is-retiring-%E2%80%94-what-to-do-before-may-1-202/4499131)
- [PowerShell Module 16.0.27011.12008 released - Icewolf Blog](https://blog.icewolf.ch/archive/2026/03/04/powershell-module-microsoft-online-sharepoint-powershell-16-0-27011-12008)
- [PowerShell Issue #12108 - GitHub](https://github.com/PowerShell/PowerShell/issues/12108)
- [about_Windows_PowerShell_Compatibility - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_windows_powershell_compatibility)

---

### 6. MicrosoftTeams 6.1.0 — PS7・Linux対応

- **PS7**: PowerShell 7.2以上で対応。基本操作は動作するが、**v6系ではPS7でのモジュールロード時に接続エラーの報告あり**。v7.3.1へのアップグレードでPS7安定性が大幅改善
- **Linux**: 全プラットフォーム対応。ただし一部の高度な管理cmdletはWinRM依存でLinux非対応の場合あり
- **注意**: Microsoftは「最良の動作にはPS5.1（Windows）を推奨」と記載しており、PS7/Linuxでの動作は公式にはベストエフォート

#### バージョン更新に関する注意


| バージョン      | 変更内容                                             |
| ---------- | ------------------------------------------------ |
| v6系        | PS7でのモジュールロード時に接続エラーの報告あり                        |
| v7.0.0     | `CustomizeFederation`パラメータ廃止、`OptionFlags`出力属性削除 |
| v7.3.1（推奨） | 2025年8月リリース。PS7での安定性が大幅に改善                       |


- **判定**: 移行可能。v7.3.1へのアップグレード推奨

**参考リンク**:

- [Teams PowerShell Supported Versions - Microsoft Learn](https://learn.microsoft.com/en-us/microsoftteams/teams-powershell-supported-versions)
- [Teams PowerShell Release Notes - Microsoft Learn](https://learn.microsoft.com/en-us/microsoftteams/teams-powershell-release-notes)
- [MicrosoftTeams 7.3.1 released - Icewolf Blog](https://blog.icewolf.ch/archive/2025/08/22/microsoftteams-powershell-module-7-3-1-released/)
- [Install Microsoft Teams PowerShell - Microsoft Learn](https://learn.microsoft.com/en-us/microsoftteams/teams-powershell-install)

---

### 7. ExchangeOnlineManagement 3.1.0 — PS7・Linux対応（条件付き）

- **PS7**: v2.0.4以降でPS7.0.3+対応。現行のv3.1.0はPS7.2+で動作する。**v3.5.0以降はPS 7.4.0必須**（.NET 8.0依存）
- **Linux**: macOS/Linux対応（v3.0以降）

#### Linux上の制限


| 項目                                  | 状態               |
| ----------------------------------- | ---------------- |
| REST APIベースのコマンドレット                 | 全て動作             |
| WinRM依存のコマンドレット（旧Remote PowerShell） | 非対応              |
| 一部認証フロー（WAM等）                       | Linuxでは使用不可      |
| `-UseRPSSession:$false` フラグ         | RESTモード明示指定で回避可能 |


- **最新版**: v3.9.2（2026年1月時点）
- **判定**: 移行可能。REST APIベースのコマンドを使用すればLinuxでも問題なし。v3.5.0以上へのアップグレード推奨

**参考リンク**:

- [About the Exchange Online PowerShell V3 module - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/exchange/exchange-online-powershell-v2?view=exchange-ps)
- [Connect to Exchange Online PowerShell - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/exchange/connect-to-exchange-online-powershell)

---

### 8. Microsoft.PowerApps.Administration.PowerShell 2.0.168 — PS5.1専用

- **PS7**: **非対応**。.NET Framework依存のため、PowerShell 7（.NET Core）では動作不可
- **Linux**: **完全に非対応**
- **技術的理由**: モジュールが.NET Framework 4.xに依存しており、.NET Core/.NET 6+では動作しない
- **コミュニティ実装**: Linux用Dockerイメージの実験的サンプルが存在するが、認証が一部のみ対応で完全動作は保証されない

#### 代替案


| 代替手段                           | 特徴                                                          |
| ------------------------------ | ----------------------------------------------------------- |
| **Power Platform CLI (`pac`)** | クロスプラットフォーム対応。推奨                                            |
| **Power Platform REST API**    | `https://api.bap.microsoft.com` を直接呼び出し                     |
| **Microsoft Graph API**        | 一部Power Platform管理機能をカバー（`/providers/Microsoft.PowerApps/`） |


- **判定**: Linux移植不可。REST API or Power Platform CLIに要置き換え

**参考リンク**:

- [PowerShell support for Power Apps - Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/admin/powerapps-powershell)
- [PowerShell Installation for Power Platform - Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/admin/powershell-installation)
- [Docker sample for PowerApps Admin on Linux - GitHub](https://github.com/Grant-Archibald-MS/docker-Microsoft.PowerApps.Administration.PowerShell)

---

### 9. MicrosoftPowerBIMgmt 1.2.1111 — PS7・Linux完全対応

- **PS7**: PowerShell Core v6以上で対応
- **Linux**: PowerShell Coreが動作する全OSプラットフォームで対応
- **.NET**: .NET 4.7.1（Windows PS）+ .NET Core（PowerShell Core）両対応
- **REST APIベース**の実装のためWindows固有依存なし
- **判定**: そのまま移行可能

**参考リンク**:

- [Power BI Cmdlets reference - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/power-bi/overview?view=powerbi-ps)
- [MicrosoftPowerBIMgmt - GitHub](https://github.com/microsoft/powerbi-powershell)

---

### 10. MSAL.PS 4.37.0.0 — アーカイブ済・非推奨

- **PS7**: 動作するが、**2023年9月にリポジトリがアーカイブ済**。今後のアップデートなし
- **Linux**: PowerShell Galleryのタグに`Linux`, `MacOS`, `PSEdition_Core`が明示されており動作を宣言。ただし実運用での検証は限定的
- **状態**: Microsoftが公式にアーカイブ宣言。公式サポートなし

#### 代替案


| 代替手段                | 特徴                         | PS7/Linux             |
| ------------------- | -------------------------- | --------------------- |
| **PSMSALNet**       | MSAL.NETのPSラッパー。PS 7.4+必須  | Windows/macOS/Linux対応 |
| **Connect-MgGraph** | Microsoft.Graph認証機能で直接認証   | 完全対応                  |
| **Az.Accounts**     | `Get-AzAccessToken`でトークン取得 | 完全対応                  |


- **注意**: Microsoftは公式のMSAL PowerShellラッパーを維持していない。MSAL.PS/PSMSALNetともコミュニティ維持
- **判定**: 速やかにPSMSALNet or Microsoft.Graph認証に移行すべき

**参考リンク**:

- [MSAL.PS GitHub (Archived)](https://github.com/AzureAD/MSAL.PS)
- [PSMSALNet - GitHub](https://github.com/SCOMnewbie/PSMSALNet)
- [Testing PSMSALNet because MSAL.PS has been archived - Icewolf Blog](https://blog.icewolf.ch/archive/2023/10/16/testing-psmsalnet-because-msal-ps-has-been-archived/)
- [MSAL.PS - PowerShell Gallery](https://www.powershellgallery.com/packages/MSAL.PS/4.37.0.0)

---

## Linux移植時の影響マトリクス


| モジュール                     | 動作しないコマンド/機能  | 原因                       | 推奨代替手段                               |
| ------------------------- | ------------- | ------------------------ | ------------------------------------ |
| **SharePoint.PowerShell** | 全コマンド         | .NET Framework + WinRM依存 | PnP.PowerShell v3                    |
| **PowerApps.Admin**       | 全コマンド         | .NET Framework依存         | Power Platform CLI / REST API        |
| **SqlServer**             | ~47コマンド（109中） | SMO Windows API依存        | dbatools / 直接SQL接続                   |
| **MSAL.PS**               | 一部認証フロー       | アーカイブ済・未検証               | PSMSALNet / Az.Accounts              |
| **EXO Management**        | WinRM依存コマンド   | WinRM非対応                 | REST APIモード(`-UseRPSSession:$false`) |
| **MicrosoftTeams**        | 一部管理コマンド      | WinRM依存                  | Graph API Teams エンドポイント              |


---

## 技術的背景: なぜ動かないのか

### .NET Framework vs .NET Core/.NET 6+

PowerShell 5.1は.NET Framework 4.x上で動作し、PowerShell 7は.NET Core/.NET 6/7/8上で動作する。これが互換性の根本的な分岐点。


| ランタイム               | PS対応          | OS          | 備考                             |
| ------------------- | ------------- | ----------- | ------------------------------ |
| .NET Framework 4.x  | PS 5.1のみ      | Windows専用   | COM/DCOM/WinRM等のWindows API利用可 |
| .NET Standard 2.0   | PS 5.1 + PS 7 | クロスプラットフォーム | 両方で動作する共通仕様                    |
| .NET Core / .NET 6+ | PS 7のみ        | クロスプラットフォーム | 新しいAPIのみ                       |
| .NET 8              | PS 7.4+のみ     | クロスプラットフォーム | 最新。Microsoft.Graph v2.26+等     |


### UseWindowsPowerShell互換レイヤー

PS7にはWindows上でのみ利用可能な互換レイヤーがある。`Import-Module XXX -UseWindowsPowerShell` とすると、バックグラウンドでPS5.1プロセスが起動し、暗黙的リモーティングで接続する。

**重要な制限**:

- **Windows上でのみ動作**（Linux/macOSでは使用不可）
- オブジェクトのメソッドが使えない（シリアライズされるため）
- パフォーマンス低下
- 一部コマンドの制限あり

→ Linux移植の文脈では**この互換レイヤーは使えない**

---

## 推奨移行戦略

### Phase 1: 即座に移行可能（リスク低）


| モジュール                | 作業内容             |
| -------------------- | ---------------- |
| Microsoft.Graph      | そのまま移行。PS7.4+推奨  |
| Az                   | そのまま移行。最新版への更新推奨 |
| MicrosoftPowerBIMgmt | そのまま移行           |


### Phase 2: バージョンアップ+テスト必要（リスク中）


| モジュール                    | 作業内容                                            |
| ------------------------ | ----------------------------------------------- |
| MicrosoftTeams           | v7.3.1へアップグレード。破壊的変更（CustomizeFederation廃止等）を確認 |
| ExchangeOnlineManagement | v3.5.0以上にアップグレード。REST APIモードでの動作確認              |
| PnP.PowerShell           | v1.11のままPS7/Linuxで動作確認。長期的にはv3.0への移行を検討（移行ガイド参照） |


### Phase 3: 代替モジュール/APIへの書き換え（リスク高）


| モジュール                                         | 作業内容                                    |
| --------------------------------------------- | --------------------------------------- |
| MSAL.PS                                       | PSMSALNet or Connect-MgGraph に置き換え      |
| SqlServer（Linux時）                             | 使用コマンドの棚卸し。非対応分はdbatools or 直接SQL       |
| Microsoft.Online.SharePoint.PowerShell        | **PnP.PowerShell v3に完全移行**              |
| Microsoft.PowerApps.Administration.PowerShell | **Power Platform CLI or REST APIに書き換え** |


### PS7バージョン選定の推奨

各モジュールの要件を考慮すると、**PowerShell 7.4（LTS）以上**を基準バージョンとすることを推奨。


| PS7バージョン | 対応モジュール数                   |
| -------- | -------------------------- |
| PS 7.0   | 6/10                       |
| PS 7.2   | 8/10（Teams, EXO追加）         |
| PS 7.4   | 8/10（EXO v3.5+対応、PnP v3対応） |
| PS 7.4.6 | 8/10（PnP v3の最終要件を満たす）      |


※ SharePoint管理シェルとPowerApps管理モジュールはPSバージョンに関わらず非対応

---

## 未解決・追加調査が必要な項目

1. Microsoft.PowerApps.Administration.PowerShellの最新版でPS7対応が改善されているかの確認
2. SqlServerモジュールの最新版（22.x）でのLinux対応cmdlet数の比較
3. PnP.PowerShell v1.11 → v3.0へのスクリプト変更コスト（破壊的変更の影響範囲）
4. MicrosoftTeams v6.1.0 → v7.x へのコマンドレット名変更・廃止の影響調査
5. ExchangeOnlineManagement v3.1.0 → v3.9.0 へのアップグレード後のLinux動作実績確認
6. MicrosoftTeams のLinux上で具体的に動作しないcmdletリストの確定

---

## 結論

### PowerShell 7への移行

10モジュール中**7モジュールは対応済み**。SharePoint管理シェル（`Microsoft.Online.SharePoint.PowerShell`）とPowerApps管理モジュール（`Microsoft.PowerApps.Administration.PowerShell`）が最大の障壁。MSAL.PSはアーカイブ済みで早期代替が必要。

### Linux移植

10モジュール中**6モジュールは完全対応（うちMSAL.PSは動作宣言済だがアーカイブ済で実運用未検証）、2モジュールは部分対応、2モジュールは完全非対応**。SharePoint管理シェル・PowerApps管理モジュールの2つはLinuxでは動作せず、代替手段への書き換えが必須。

### 懸念に対する回答

- **Teams**: PS7.2+で動作する。最新v7.3.1にアップグレードすれば安定。Linuxでも基本操作は可能。**懸念は杞憂**
- **SharePoint（SPO管理シェル）**: **懸念は正しい**。PS7ネイティブ非対応、Linux完全不可。ただし**PnP.PowerShellがほぼ完全な代替**として機能する

---

## 参考リンク一覧

### Microsoft公式ドキュメント

- [PowerShell 7 module compatibility - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/whats-new/module-compatibility?view=powershell-7.5)
- [Differences between Windows PowerShell 5.1 and PowerShell 7.x - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/whats-new/differences-from-windows-powershell)
- [about_Windows_PowerShell_Compatibility - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_windows_powershell_compatibility)
- [Microsoft Graph PowerShell SDK - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview?view=graph-powershell-1.0)
- [Az PowerShell module - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az)
- [Manage SQL Server on Linux with PowerShell - Microsoft Learn](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-manage-powershell-core)
- [Teams PowerShell Supported Versions - Microsoft Learn](https://learn.microsoft.com/en-us/microsoftteams/teams-powershell-supported-versions)
- [About the Exchange Online PowerShell V3 module - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/exchange/exchange-online-powershell-v2?view=exchange-ps)
- [PowerShell support for Power Apps - Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/admin/powerapps-powershell)
- [Power BI Cmdlets reference - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/power-bi/overview?view=powerbi-ps)

### GitHub・コミュニティ

- [PnP PowerShell公式](https://pnp.github.io/powershell/)
- [PnP PowerShell v3 breaking changes](https://pnp.github.io/blog/pnp-powershell/pnp-powershell-v3-0-0-preview/)
- [PnP PowerShell Migration Guide v2→v3](https://github.com/pnp/powershell/blob/dev/MIGRATE-2.0-to-3.0.md)
- [MSAL.PS GitHub (Archived)](https://github.com/AzureAD/MSAL.PS)
- [PSMSALNet - MSAL.PS replacement](https://github.com/SCOMnewbie/PSMSALNet)
- [MicrosoftPowerBIMgmt - GitHub](https://github.com/microsoft/powerbi-powershell)
- [Docker sample for PowerApps Admin on Linux](https://github.com/Grant-Archibald-MS/docker-Microsoft.PowerApps.Administration.PowerShell)
- [SQLServerPSModule Issue #72](https://github.com/microsoft/SQLServerPSModule/issues/72)
- [dbatools](https://dbatools.io/)

### ブログ・解説記事

- [SharePoint Online PowerShell with PowerShell 7 - Koskila.net](https://www.koskila.net/how-to-use-microsoft-online-sharepoint-powershell-with-powershell-7/)
- [SharePoint Online PowerShell OAuth upgrade - office365itpros.com](https://office365itpros.com/2025/03/14/sharepoint-online-powershell-oauth/)
- [Teams PowerShell v7.0.0 update - maxime.hiez.ca](https://maxime.hiez.ca/en/blog/2025-04-20-teams-update-module-powershell-7-0-0)
- [MicrosoftTeams 7.3.1 released - Icewolf Blog](https://blog.icewolf.ch/archive/2025/08/22/microsoftteams-powershell-module-7-3-1-released/)
- [Testing PSMSALNet - Icewolf Blog](https://blog.icewolf.ch/archive/2023/10/16/testing-psmsalnet-because-msal-ps-has-been-archived/)
- [Using Windows PowerShell Modules in PowerShell 7 - Petri](https://petri.com/using-windows-powershell-modules-in-powershell-7/)

