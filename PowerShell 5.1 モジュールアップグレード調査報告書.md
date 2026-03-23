# PowerShell 5.1 モジュールアップグレード調査報告書

**調査日**: 2026-03-20

**ディープリサーチ検証日**: 2026-03-23（PowerShell Gallery 実データに基づく全面検証済み）

**対象CSV**: `GlobalMaintenanceTool_20260330release\Dockerfiles\expected_modules.csv`

**制約**: PowerShell 5.1 (Desktop) 互換 / PnP.PowerShell との依存関係維持

---

## 最重要制約：PnP.PowerShell と Az.Accounts の依存関係

過去事例として、Az.Accounts 5.3.2 が同梱する `System.Text.Encodings.Web` のバージョンが

PnP.PowerShell 1.11.0 の期待するバージョンと不整合を起こし、

`MissingMethodException`（`FindFirstCharacterToEncodeUtf8`）が発生した。

PowerShell 5.1 (Desktop) では、先にロードされたアセンブリが AppDomain 全体で共有されるため、

Az.Accounts が読み込んだアセンブリバージョンが PnP.PowerShell と競合する。

> **検証済み**: この問題は GitHub 上で複数報告されている実在の既知問題（pnp/powershell#727, #1643, #1280, #831、MicrosoftDocs/office-docs-powershell#7780、Azure/azure-powershell#22895 等）。

**結論:**

- **Az.Accounts 5.x** → PnP.PowerShell 1.x と共存不可

- **Az 14.0.0+** → Az.Accounts >= 5.0.0 を要求するため使用不可

- **Az 13.5.0** → Az.Accounts >= 4.2.0 を要求 → 4.x 系で満たせるため安全

---

## アップグレード提案一覧

### Az 関連モジュール（Az 13.5.0 同梱版の正確なバージョン付き）

| モジュール | 現行バージョン | 提案バージョン | メジャー差 | 備考 |
|-----------|-------------|-------------|----------|------|
| Az | 8.0.0 | **13.5.0** | +5 | Az.Accounts 4.x で動作する最高版（13.x 系最終版） |
| Az.Accounts | 4.0.2 | **4.2.0** | 0 | 4.x 系最終版。5.x は PnP.PowerShell と競合するため不可 |
| Az.Advisor | 1.1.2 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.Aks | 4.1.0 | **6.1.1** | +2 | 破壊的変更確認要 |
| Az.AnalysisServices | 1.1.4 | **1.2.0** | 0 | |
| Az.ApiManagement | 3.0.0 | **4.1.0** | +1 | 破壊的変更確認要 |
| Az.AppConfiguration | 1.1.0 | **1.4.1** | 0 | |
| Az.ApplicationInsights | 2.0.0 | **2.3.0** | 0 | |
| Az.Attestation | 1.0.0 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.Automation | 1.7.3 | **1.11.1** | 0 | |
| Az.Batch | 3.2.0 | **3.7.0** | 0 | |
| Az.Billing | 2.0.0 | **2.2.0** | 0 | |
| Az.Cdn | 2.1.0 | **3.3.1** | +1 | 破壊的変更確認要 |
| Az.CloudService | 1.1.0 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.CognitiveServices | 1.11.0 | **1.16.0** | 0 | |
| Az.Compute | 4.27.0 | **9.3.0** | +5 | **高リスク** 破壊的変更確認必須 |
| Az.ContainerInstance | 3.1.0 | **4.1.1** | +1 | 破壊的変更確認要 |
| Az.ContainerRegistry | 3.0.0 | **4.3.0** | +1 | 破壊的変更確認要 |
| Az.CosmosDB | 1.8.0 | **1.18.0** | 0 | |
| Az.DataBoxEdge | 1.1.0 | **1.2.1** | 0 | |
| Az.Databricks | 1.2.0 | **1.10.0** | 0 | |
| Az.DataFactory | 1.16.7 | **1.19.2** | 0 | |
| Az.DataLakeAnalytics | 1.0.2 | **1.1.0** | 0 | |
| Az.DataLakeStore | 1.3.0 | **1.4.0** | 0 | |
| Az.DataShare | 1.0.1 | **1.1.1** | 0 | |
| Az.DeploymentManager | 1.1.0 | **削除（Az 13.5.0 に同梱されない）** | - | Az 13.5.0 から除外。使用箇所がある場合は個別インストール要検討 |
| Az.DesktopVirtualization | 3.1.0 | **5.4.1** | +2 | 破壊的変更確認要 |
| Az.DevTestLabs | 1.0.2 | **1.1.0** | 0 | |
| Az.Dns | 1.1.2 | **1.3.1** | 0 | |
| Az.EventGrid | 1.3.0 | **2.2.0** | +1 | 破壊的変更確認要 |
| Az.EventHub | 2.0.0 | **5.2.0** | +3 | **高リスク** 破壊的変更確認必須 |
| Az.FrontDoor | 1.9.0 | **1.13.0** | 0 | |
| Az.Functions | 4.0.3 | **4.2.0** | 0 | |
| Az.HDInsight | 5.0.1 | **6.3.1** | +1 | 破壊的変更確認要 |
| Az.HealthcareApis | 2.0.0 | **2.1.0** | 0 | |
| Az.IotHub | 2.7.4 | **2.8.0** | 0 | |
| Az.KeyVault | 4.5.0 | **6.3.1** | +2 | **高リスク** 破壊的変更確認必須 |
| Az.Kusto | 2.1.0 | **2.4.0** | 0 | |
| Az.LogicApp | 1.5.0 | **1.6.0** | 0 | |
| Az.MachineLearning | 1.1.3 | **1.2.0** | 0 | |
| Az.Maintenance | 1.2.0 | **1.5.1** | 0 | |
| Az.ManagedServiceIdentity | 1.0.0 | **1.3.1** | 0 | |
| Az.ManagedServices | 3.0.0 | **3.1.0** | 0 | |
| Az.MarketplaceOrdering | 1.0.2 | **2.2.0** | +1 | 破壊的変更確認要 |
| Az.Media | 1.1.1 | **1.2.0** | 0 | |
| Az.Migrate | 1.1.2 | **2.7.0** | +1 | 破壊的変更確認要 |
| Az.Monitor | 3.0.1 | **6.0.2** | +3 | **高リスク** 破壊的変更確認必須 |
| Az.MySql | 1.0.0 | **1.3.0** | 0 | |
| Az.Network | 4.17.0 | **7.16.1** | +3 | **高リスク** 破壊的変更確認必須 |
| Az.NotificationHubs | 1.1.1 | **1.2.0** | 0 | |
| Az.OperationalInsights | 3.1.0 | **3.3.0** | 0 | |
| Az.PolicyInsights | 1.5.0 | **1.7.1** | 0 | |
| Az.PostgreSql | 1.1.0 | **1.2.0** | 0 | |
| Az.PowerBIEmbedded | 1.1.2 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.PrivateDns | 1.0.3 | **1.2.0** | 0 | |
| Az.RecoveryServices | 5.4.0 | **7.7.0** | +2 | 破壊的変更確認要 |
| Az.RedisCache | 1.6.0 | **1.11.0** | 0 | |
| Az.RedisEnterpriseCache | 1.0.0 | **1.4.1** | 0 | |
| Az.Relay | 1.0.3 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.ResourceMover | 1.1.0 | **1.3.0** | 0 | |
| Az.Resources | 6.0.0 | **7.11.0** | +1 | 破壊的変更確認要 |
| Az.Security | 1.3.0 | **1.8.0** | 0 | |
| Az.SecurityInsights | 1.1.0 | **3.2.0** | +2 | 破壊的変更確認要 |
| Az.ServiceBus | 1.9.0 | **4.1.1** | +3 | **高リスク** 破壊的変更確認必須 |
| Az.ServiceFabric | 3.0.2 | **3.5.0** | 0 | |
| Az.SignalR | 1.4.1 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.Sql | 3.9.0 | **6.0.3** | +3 | **高リスク** 破壊的変更確認必須 |
| Az.SqlVirtualMachine | 1.1.0 | **2.4.0** | +1 | 破壊的変更確認要 |
| Az.StackHCI | 1.1.1 | **2.5.0** | +1 | 破壊的変更確認要 |
| Az.Storage | 4.6.0 | **8.4.0** | +4 | **高リスク** 破壊的変更確認必須 |
| Az.StorageSync | 1.7.0 | **2.5.0** | +1 | 破壊的変更確認要 |
| Az.StreamAnalytics | 2.0.0 | **2.1.0** | 0 | |
| Az.Support | 1.0.0 | **2.1.0** | +1 | 破壊的変更確認要 |
| Az.Synapse | 1.4.0 | **3.2.1** | +2 | 破壊的変更確認要 |
| Az.TrafficManager | 1.1.0 | **1.3.0** | 0 | |
| Az.Websites | 2.11.2 | **3.4.1** | +1 | 破壊的変更確認要 |

#### Az 13.5.0 で新規追加されるモジュール（Az 8.0.0 には存在しない）

以下のモジュールは Az 8.0.0 には含まれておらず、Az 13.5.0 で新たに追加される。既存スクリプトへの影響はないが、expected_modules.csv の更新が必要:

| モジュール | バージョン |
|-----------|----------|
| Az.App | 2.0.1 |
| Az.ArcResourceBridge | 1.1.0 |
| Az.Automanage | 1.1.0 |
| Az.ConfidentialLedger | 1.1.0 |
| Az.ConnectedMachine | 1.1.1 |
| Az.DataProtection | 2.7.0 |
| Az.DevCenter | 2.0.1 |
| Az.DnsResolver | 1.1.1 |
| Az.ElasticSan | 1.4.0 |
| Az.HealthDataAIServices | 1.0.0 |
| Az.LoadTesting | 1.1.0 |
| Az.MachineLearningServices | 1.2.0 |
| Az.NetworkCloud | 1.1.0 |
| Az.Nginx | 1.2.0 |
| Az.Oracle | 1.1.0 |
| Az.ResourceGraph | 1.2.1 |
| Az.StackHCIVM | 1.1.0 |
| Az.StorageMover | 1.5.0 |
| Az.Workloads | 1.0.0 |

### Microsoft 365 関連モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |
|-----------|-------------|-------------|------|
| PnP.PowerShell | 1.11.0 | **1.12.0** | PS 5.1 対応の最終版（2.x は PS 7.2+ 必須、3.x は PS 7.4.0+ 必須） |
| Microsoft.Graph（全サブモジュール） | 1.27.0 | **1.28.0** | 1.x 最終版（2023-06-08 公開）。2.x (2.36.1) も PS 5.1 対応だが破壊的変更あり |
| ExchangeOnlineManagement | 3.1.0 | **3.9.2** | 同一メジャー版内の更新（2026-01-05 公開） |
| MicrosoftTeams | 6.1.0 | **7.6.0** | PS 5.1 対応確認済み（2026-01-23 公開） |
| Microsoft.Online.SharePoint.PowerShell | 16.0.27011.12008 | **変更不要** | 既に最新（2026-03-03 公開） |
| Microsoft.PowerApps.Administration.PowerShell | 2.0.168 | **2.0.216** | 最新版（2025-09-13 公開） |

### Power BI 関連モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |
|-----------|-------------|-------------|------|
| MicrosoftPowerBIMgmt | 1.2.1111 | **1.3.83** | 最新版（2026-03-18 公開） |
| MicrosoftPowerBIMgmt.Admin | 1.2.1111 | **1.3.83** | |
| MicrosoftPowerBIMgmt.Capacities | 1.2.1111 | **1.3.83** | |
| MicrosoftPowerBIMgmt.Data | 1.2.1111 | **1.3.83** | |
| MicrosoftPowerBIMgmt.Profile | 1.2.1111 | **1.3.83** | |
| MicrosoftPowerBIMgmt.Reports | 1.2.1111 | **1.3.83** | |
| MicrosoftPowerBIMgmt.Workspaces | 1.2.1111 | **1.3.83** | |

### その他モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |
|-----------|-------------|-------------|------|
| MSAL.PS | 4.37.0.0 | **変更不要** | 既に最新（2021-11-19 以降更新なし） |
| PackageManagement | 1.4.8.1 | **変更不要** | 既に最新（2022-07-01 公開） |
| PowerShellGet | 2.2.5 | **変更不要** | 2.x 系では最新（3.x は別モジュール PSResourceGet） |
| SqlServer | 21.1.18256 | **22.4.5.1** | 最新版（2025-06-17 公開）、PS 5.1 対応確認済み |

---

## リスク評価

### 最高リスク（要テスト・要 Breaking Changes 調査）

1. **Az 8.0.0 → 13.5.0（メジャー +5）**

   - メジャーバージョンを5つ跨ぐため、大量の破壊的変更が含まれる
   - 既知の破壊的変更例: `Resolve-Error` エイリアス削除（Az.Accounts 4.0.0 で実施、Az.Accounts 3.0.5 で事前告知済み）
   - **必ず Az.Accounts 4.2.0 + PnP.PowerShell 1.12.0 の組み合わせを事前検証すること**
   - Az.Accounts 4.2.0 で問題が出る場合、Az 13.0.0 (Az.Accounts >= 4.0.0) + Az.Accounts 4.0.2 固定にフォールバック可能
   - **サブモジュール30件以上でメジャーバージョンが変わるため、影響範囲は極めて広い**（後述の「メジャーバージョンジャンプ一覧」参照）

2. **Microsoft.Graph 1.x の EOL リスク（セキュリティ上の最高リスク）**

   - 1.28.0 は **2023年6月8日が最終リリース**
   - Microsoft Graph SDK のサポートポリシーでは、最新メジャーバージョン（2.x）リリース後、前メジャーバージョン（1.x）は **12ヶ月間セキュリティ修正のみ** 提供される
   - 2.0.0 は **2023年7月5日** にリリースされたため、1.x のセキュリティサポートは **2024年7月頃に終了**
   - 調査時点（2026-03-20）で **約1年8ヶ月間、セキュリティパッチを含む一切のサポートが提供されていない**
   - 2.x は PS 5.1 に対応しているため PS バージョンの制約にはならないが、コマンドレット名・パラメータ・戻り値に破壊的変更があるためスクリプト改修が必要
   - 今回は 1.28.0（1.x 最終版）を採用するが、**セキュリティリスクを受容した上での判断であり、2.x 移行計画を早急に策定すべき**

### 高リスク（メジャーバージョン +3 以上のサブモジュール）

以下のサブモジュールは特に破壊的変更の影響が大きい可能性がある:

| モジュール | 現行 → 提案 | メジャー差 | 要確認事項 |
|-----------|-----------|----------|----------|
| Az.Compute | 4.27.0 → 9.3.0 | +5 | VM管理コマンドレットの引数・戻り値変更 |
| Az.Storage | 4.6.0 → 8.4.0 | +4 | Blob/Table/Queue 操作の API 変更 |
| Az.Network | 4.17.0 → 7.16.1 | +3 | NSG/VNet/LB 関連コマンドレット変更 |
| Az.Sql | 3.9.0 → 6.0.3 | +3 | DB管理コマンドレットの変更 |
| Az.EventHub | 2.0.0 → 5.2.0 | +3 | イベントハブ管理の全面刷新の可能性 |
| Az.Monitor | 3.0.1 → 6.0.2 | +3 | 監視・アラート関連コマンドレット変更 |
| Az.ServiceBus | 1.9.0 → 4.1.1 | +3 | サービスバス管理の全面刷新の可能性 |

### 中リスク（メジャーバージョン +1〜+2）

3. **MicrosoftTeams 6.x → 7.x** — メジャーバージョン更新のため動作確認推奨

4. **Az サブモジュール メジャー +2:**
   - Az.KeyVault 4.5.0 → 6.3.1（シークレット管理・証明書管理で引数/戻り値変更の可能性）
   - Az.Aks 4.1.0 → 6.1.1
   - Az.DesktopVirtualization 3.1.0 → 5.4.1
   - Az.RecoveryServices 5.4.0 → 7.7.0
   - Az.SecurityInsights 1.1.0 → 3.2.0
   - Az.Synapse 1.4.0 → 3.2.1

5. **Az サブモジュール メジャー +1:**
   Az.Advisor, Az.ApiManagement, Az.Attestation, Az.Cdn, Az.CloudService, Az.ContainerInstance, Az.ContainerRegistry, Az.EventGrid, Az.HDInsight, Az.MarketplaceOrdering, Az.Migrate, Az.PowerBIEmbedded, Az.Relay, Az.Resources, Az.SignalR, Az.SqlVirtualMachine, Az.StackHCI, Az.StorageSync, Az.Support, Az.Websites

### 低リスク

6. ExchangeOnlineManagement 3.1.0 → 3.9.2（マイナー更新）

7. MicrosoftPowerBIMgmt 1.2.1111 → 1.3.83（マイナー更新）

8. SqlServer 21.x → 22.x（メジャーだが影響範囲が限定的）

9. PnP.PowerShell 1.11.0 → 1.12.0（マイナー更新）

10. Microsoft.PowerApps.Administration.PowerShell 2.0.168 → 2.0.216（パッチ更新）

---

## Az モジュール依存関係マップ

| Az バージョン | 要求する Az.Accounts | PnP.PowerShell 共存 |
|-------------|-------------------|-------------------|
| 8.0.0（現行） | >= 2.8.0 | ○ |
| 12.0.0 | >= 3.0.0 | ○ |
| 12.5.0 | >= 3.0.5 | ○ |
| 13.0.0 | >= 4.0.0 | ○（Az.Accounts 4.0.2 で可） |
| **13.5.0（推奨）** | **>= 4.2.0** | **○（Az.Accounts 4.2.0 で可）** |
| 14.0.0 | >= 5.0.0 | **×（5.x は PnP と競合）** |
| 15.4.0（最新） | >= 5.3.3 | **×** |

**補足**: Az 13.5.0 は 13.x 系の最終版であり、13.6.0 以降は公開されていない（2026-03-23 PowerShell Gallery 確認済み）。次のバージョンは Az 14.0.0 であり、Az.Accounts 5.x を要求するため PnP.PowerShell との共存が不可。したがって **Az 13.5.0 が PnP.PowerShell と共存できる理論上の上限** である。

**Az.Accounts バージョン一覧（参考）**:
- 4.x 系: 4.0.0, 4.0.1, 4.0.2, 4.1.0, **4.2.0**（最終版、2025-05-06 公開）
- 5.x 系: 5.0.0 〜 **5.3.3**（最新、2026-03-03 公開）

---

## Dockerfile での固定方法（再発防止）

過去事例を踏まえ、すべてのモジュールで `-RequiredVersion` を明示的に指定すること。

**重要**: Az.Accounts を先にインストールし、Az を後からインストールすること。Az を先にインストールすると依存関係により Az.Accounts の最新互換版（5.x 系を含む）が自動インストールされる可能性がある。

```dockerfile

# ===== インストール順序が重要 =====

# Step 1: Az.Accounts を先に固定
RUN powershell Install-Module Az.Accounts -RequiredVersion 4.2.0 -Force

# Step 2: Az をインストール（Az.Accounts 4.2.0 が既にあるため上書きされない）
RUN powershell Install-Module Az -RequiredVersion 13.5.0 -Force -AllowClobber

# Microsoft 365
RUN powershell Install-Module PnP.PowerShell -RequiredVersion 1.12.0 -Force

RUN powershell Install-Module ExchangeOnlineManagement -RequiredVersion 3.9.2 -Force

RUN powershell Install-Module MicrosoftTeams -RequiredVersion 7.6.0 -Force

RUN powershell Install-Module Microsoft.Graph -RequiredVersion 1.28.0 -Force

RUN powershell Install-Module Microsoft.PowerApps.Administration.PowerShell -RequiredVersion 2.0.216 -Force

# Power BI
RUN powershell Install-Module MicrosoftPowerBIMgmt -RequiredVersion 1.3.83 -Force

# その他
RUN powershell Install-Module SqlServer -RequiredVersion 22.4.5.1 -Force

```

**Microsoft.Graph サブモジュールについて**: `Install-Module Microsoft.Graph -RequiredVersion 1.28.0` を実行すると、依存する全サブモジュール（Microsoft.Graph.Authentication, Microsoft.Graph.Users 等）が同一バージョンで自動インストールされる。expected_modules.csv に記載のサブモジュールを個別にインストールする必要はない。ただし、ビルド後に `Get-InstalledModule Microsoft.Graph.* | Select Name, Version` で全サブモジュールが 1.28.0 であることを検証すること。

---

## ディープリサーチ検証結果（2026-03-23）

PowerShell Gallery の公開情報および Microsoft 公式ドキュメントをもとに本報告書の内容を再検証した結果を以下に記す。

### 検証に使用した情報源

- [PowerShell Gallery](https://www.powershellgallery.com/) — 各モジュールのバージョン・依存関係・公開日
- [Azure PowerShell Release Notes](https://learn.microsoft.com/en-us/powershell/azure/release-notes-azureps) — Az モジュールの破壊的変更情報
- [PnP PowerShell GitHub](https://github.com/pnp/powershell) — PnP.PowerShell の Issue・互換性情報
- [Microsoft Graph PowerShell SDK GitHub](https://github.com/microsoftgraph/msgraph-sdk-powershell) — Graph SDK のサポートポリシー
- [Microsoft Graph PowerShell SDK v2 GA 発表](https://devblogs.microsoft.com/microsoft365dev/upgrade-to-microsoft-graph-powershell-sdk-v2-now-generally-available/) — 2.0.0 リリース日確認
- [Microsoft Learn - Exchange Online PowerShell](https://learn.microsoft.com/en-us/powershell/exchange/connect-to-exchange-online-powershell) — UseRpsSession 非推奨情報

### 初版からの修正事項

#### 修正1: Az 13.5.0 サブモジュールバージョンの完全版記載

初版では主要6モジュールのみ具体的バージョンを記載し、残りは「Az 13.5.0 同梱版」と記載していたが、**全サブモジュールの正確なバージョンを PowerShell Gallery から取得し記載した**。結果、メジャーバージョンジャンプがあるモジュールは報告書初版の6件ではなく **30件以上** であることが判明。

#### 修正2: Az.DeploymentManager の削除

初版では Az.DeploymentManager を「Az 13.5.0 同梱版」と記載していたが、**Az 13.5.0 の依存関係リストに Az.DeploymentManager は含まれていない**。Az 8.0.0 → 13.5.0 の間に削除されたモジュールである。使用箇所がある場合は個別対応が必要。

#### 修正3: Az 13.5.0 新規追加モジュール（19件）の記載

Az 8.0.0 には存在せず Az 13.5.0 で新たに追加された19モジュールを一覧化した。expected_modules.csv の更新時に追加が必要。

#### 修正4: Microsoft.Graph 1.x EOL タイムラインの精緻化

初版では「約2年9ヶ月間セキュリティパッチが提供されていない」と記載していたが、Microsoft の SDK サポートポリシーに基づく正確なタイムラインは以下の通り:

- 1.28.0 最終リリース: 2023-06-08
- 2.0.0 リリース（GA）: 2023-07-05
- 1.x セキュリティサポート終了: 2024-07-05 頃（2.0.0 リリースから12ヶ月後）
- 調査時点（2026-03-20）: セキュリティサポート終了から **約1年8ヶ月**

「2年9ヶ月」は最終リリースからの経過期間であり、セキュリティサポート終了からの期間は約1年8ヶ月。いずれにしても現在はセキュリティ修正を含む一切のサポートが提供されていない状態。

#### 修正5: PnP.PowerShell 3.x の PS 要件精緻化

初版では「3.x は PS 7.4+ 必須」と記載していたが、正確には **PS 7.4.0 以上** が必要（[PnP PowerShell 公式インストールガイド](https://pnp.github.io/powershell/articles/installation.html) に明記）。最新安定版は **3.1.0**（2025-04-18 公開）。

### 検証済み（正確であることを確認）

- Az 13.5.0 の存在と Az.Accounts >= 4.2.0 の要求 → **正確**（公開日: 2025-05-06）
- Az 14.0.0 が Az.Accounts >= 5.0.0 を要求 → **正確**
- Az 15.4.0 が最新版（2026-03-03 公開）→ **正確**
- Az.Accounts 4.2.0 が 4.x 系最終版 → **正確**（4.x: 4.0.0, 4.0.1, 4.0.2, 4.1.0, 4.2.0 の5バージョンのみ）
- Az 13.5.0 が 13.x 系最終版 → **正確**（13.6.0 は存在しない）
- PnP.PowerShell 1.12.0 が PS 5.1 対応の最終 1.x 版 → **正確**（2022-11-10 公開）
- PnP.PowerShell 2.x が PS 7.2+ 必須 → **正確**
- Microsoft.Graph 1.28.0 が 1.x 最終版 → **正確**（2023-06-08 公開）
- Microsoft.Graph 2.x が PS 5.1 対応（最新 2.36.1）→ **正確**（2026-03-18 公開）
- ExchangeOnlineManagement 3.9.2 が最新 GA → **正確**（2026-01-05 公開）
- MicrosoftTeams 7.6.0 が PS 5.1 対応 → **正確**（2026-01-23 公開）
- Microsoft.Online.SharePoint.PowerShell 16.0.27011.12008 が最新 → **正確**（2026-03-03 公開）
- Microsoft.PowerApps.Administration.PowerShell 2.0.216 が最新 → **正確**（2025-09-13 公開）
- MicrosoftPowerBIMgmt 1.3.83 が最新 → **正確**（2026-03-18 公開）
- MSAL.PS 4.37.0.0 が最新（2021-11-19 以降更新なし）→ **正確**
- SqlServer 22.4.5.1 が最新、PS 5.1 対応 → **正確**（2025-06-17 公開）
- PackageManagement 1.4.8.1 が最新 → **正確**（2022-07-01 公開）
- PowerShellGet 2.2.5 が 2.x 最新 → **正確**（2020-09-22 公開）
- `FindFirstCharacterToEncodeUtf8` MissingMethodException → **実在する既知の問題**（pnp/powershell#727, #1643, #1280, #831 等で報告）
- `Resolve-Error` エイリアス削除（Az.Accounts 4.0.0）→ **正確**（Az.Accounts 3.0.5 で事前告知、4.0.0 で実施）

---

## 追加の懸念事項・課題

### 1. Az.Accounts 4.x でも System.Text.Encodings.Web アセンブリを同梱

報告書では「Az.Accounts 5.x が PnP.PowerShell と競合」と記載しているが、**Az.Accounts 4.2.0 も同じ `System.Text.Encodings.Web.dll` を `lib\net461\` に同梱している**。5.x との差異は同梱 DLL のバージョン番号であり、4.x が安全であるという保証は PowerShell Gallery のメタデータだけでは確認できない。

**対策**: テスト環境で `Az.Accounts 4.2.0` + `PnP.PowerShell 1.12.0` を **実際にロードし**、以下を確認すること:

```powershell

Import-Module Az.Accounts -RequiredVersion 4.2.0

[System.Text.Encodings.Web.JavaScriptEncoder].Assembly.GetName().Version

Import-Module PnP.PowerShell -RequiredVersion 1.12.0

Connect-PnPOnline -Url https://tenant.sharepoint.com -Interactive

```

### 2. PnP.PowerShell 3.x の存在

PnP.PowerShell の最新安定版は **3.1.0**（2025-04-18 公開、PS 7.4.0 以上必須）。2.x の最終安定版は **2.12.0**（2024-09-09 公開、PS 7.2+ 必須）。将来の PS 7 移行計画では 3.x を検討対象とすること。

### 3. Az サブモジュールのメジャーバージョンジャンプが極めて多い

**メジャー +3 以上（最優先で Breaking Changes 確認）:**

| モジュール | 現行 → 提案 | メジャー差 |
|-----------|-----------|----------|
| Az.Compute | 4.27.0 → 9.3.0 | +5 |
| Az.Storage | 4.6.0 → 8.4.0 | +4 |
| Az.Network | 4.17.0 → 7.16.1 | +3 |
| Az.Sql | 3.9.0 → 6.0.3 | +3 |
| Az.EventHub | 2.0.0 → 5.2.0 | +3 |
| Az.Monitor | 3.0.1 → 6.0.2 | +3 |
| Az.ServiceBus | 1.9.0 → 4.1.1 | +3 |

**メジャー +2:**

| モジュール | 現行 → 提案 | メジャー差 |
|-----------|-----------|----------|
| Az.KeyVault | 4.5.0 → 6.3.1 | +2 |
| Az.Aks | 4.1.0 → 6.1.1 | +2 |
| Az.DesktopVirtualization | 3.1.0 → 5.4.1 | +2 |
| Az.RecoveryServices | 5.4.0 → 7.7.0 | +2 |
| Az.SecurityInsights | 1.1.0 → 3.2.0 | +2 |
| Az.Synapse | 1.4.0 → 3.2.1 | +2 |

**メジャー +1（20件）:**
Az.Advisor, Az.ApiManagement, Az.Attestation, Az.Cdn, Az.CloudService, Az.ContainerInstance, Az.ContainerRegistry, Az.EventGrid, Az.HDInsight, Az.MarketplaceOrdering, Az.Migrate, Az.PowerBIEmbedded, Az.Relay, Az.Resources, Az.SignalR, Az.SqlVirtualMachine, Az.StackHCI, Az.StorageSync, Az.Support, Az.Websites

**合計: メジャーバージョンジャンプが発生するサブモジュールは 33件。** 各モジュールの Breaking Changes ドキュメントを確認し、使用しているコマンドレットへの影響を事前に洗い出すこと。特に Az.Compute, Az.Storage, Az.Network, Az.KeyVault は使用頻度が高く、優先的に確認すべき。

### 4. Az.DeploymentManager の削除

Az 13.5.0 の依存関係リストに **Az.DeploymentManager は含まれていない**。Az 8.0.0 → 13.5.0 の間に削除されたモジュールである。既存スクリプトで Az.DeploymentManager のコマンドレットを使用している場合、以下の対応が必要:
- 使用箇所の洗い出し
- 個別インストール（`Install-Module Az.DeploymentManager` で最新版をインストール可能か確認）
- 代替コマンドレットへの移行検討

### 5. MSAL.PS の保守終了リスク

MSAL.PS は **2021年11月19日を最後に更新が停止** しており、実質的にメンテナンスされていない。Microsoft 公式サポート対象外のコミュニティモジュールである。依存している場合、将来的に認証フローの非互換（Azure AD → Entra ID 移行等）が発生するリスクがある。代替として `Microsoft.Identity.Client`（MSAL.NET）の直接利用や、Az.Accounts 組み込みの認証機能への移行を検討すべき。

### 6. Microsoft.Graph 1.28.0 の EOL リスク（最高リスク）

Microsoft.Graph 1.x は完全にサポート終了している:

| マイルストーン | 日付 |
|-------------|------|
| 1.28.0 リリース（1.x 最終版） | 2023-06-08 |
| 2.0.0 リリース（2.x GA） | 2023-07-05 |
| 1.x セキュリティサポート終了（推定） | 2024-07-05 |
| 調査時点 | 2026-03-20 |
| サポート終了からの経過 | **約1年8ヶ月** |

**1.x を継続利用するリスク**:
- セキュリティサポート終了から約1年8ヶ月が経過。今後 CVE が発見されても修正されない
- Microsoft Graph API 側の変更により、将来的に 1.x のコマンドレットが動作しなくなる可能性

**2.x 移行の技術的障壁**:
- 2.x は PS 5.1 に対応しているため、PS バージョンの制約にはならない
- ただしコマンドレット名（Beta 系が `*-MgBeta*` にリネーム）、パラメータ、戻り値に破壊的変更がある
- Microsoft は [v1→v2 移行ツールキット](https://devblogs.microsoft.com/microsoft365dev/microsoft-graph-powershell-v1-to-v2-migration-toolkit/) を提供しており、移行作業の支援が可能
- スクリプト改修が必要（改修量はスクリプトの複雑度に依存）

今回は 1.28.0 を採用するが、**早期に 2.x 移行計画を策定すること** を強く推奨する。

### 7. SqlServer モジュールのメジャーバージョン更新

SqlServer 21.x → 22.x はメジャー更新だが、報告書では「低リスク（影響範囲が限定的）」と評価している。ただし、22.x では `Invoke-Sqlcmd` のデフォルト動作変更（`-TrustServerCertificate` の扱い等）がある可能性があるため、SQL Server 接続を行うスクリプトの動作確認を推奨する。

### 8. ExchangeOnlineManagement の UseRpsSession 非推奨化

ExchangeOnlineManagement 3.x では `UseRpsSession` パラメータが非推奨（deprecated）となっている。既存スクリプトで `Connect-ExchangeOnline -UseRPSSession` を使用している場合、REST ベースの接続への移行が必要。PS 5.1 環境では REST ベース接続がデフォルトとなるが、一部のコマンドレットの出力形式が異なる場合がある。

---

## 次のステップ

1. テスト環境で Az.Accounts 4.2.0 + PnP.PowerShell 1.12.0 の互換性を検証（**アセンブリバージョンの実機確認を含む**）

2. 問題があれば Az 13.0.0 + Az.Accounts 4.0.2 にフォールバック

3. **Az サブモジュール33件の Breaking Changes ドキュメントを確認**（優先順位: Compute > Storage > Network > KeyVault > Sql > Monitor > EventHub > ServiceBus）

4. **Az.DeploymentManager 削除の影響調査**（使用箇所の洗い出し）

5. expected_modules.csv の更新:
   - 既存モジュールのバージョン更新
   - 新規追加モジュール19件の追加
   - Az.DeploymentManager の削除対応

6. Dockerfile を更新（**Az.Accounts → Az の順序でインストール**）

7. Microsoft.Graph 2.x 移行計画を策定（**PS 5.1 対応済みのため技術的障壁は低いが、スクリプト改修が必要。移行ツールキットの活用を推奨**）

8. MSAL.PS の代替手段の調査・計画

9. ExchangeOnlineManagement の UseRpsSession 非推奨化に伴うスクリプト影響調査

---

## 検証情報源

| 情報源 | URL | 用途 |
|--------|-----|------|
| PowerShell Gallery - Az 13.5.0 | https://www.powershellgallery.com/packages/az/13.5.0 | サブモジュール依存関係の完全リスト |
| PowerShell Gallery - Az.Accounts | https://www.powershellgallery.com/packages/Az.Accounts | 4.x/5.x 全バージョン一覧 |
| PowerShell Gallery - Az 14.0.0 | https://www.powershellgallery.com/packages/Az/14.0.0 | Az.Accounts >= 5.0.0 要求の確認 |
| Azure PowerShell Release Notes | https://learn.microsoft.com/en-us/powershell/azure/release-notes-azureps | Resolve-Error 削除等の破壊的変更 |
| PnP PowerShell GitHub Issues | https://github.com/pnp/powershell/issues/727 | FindFirstCharacterToEncodeUtf8 問題 |
| Microsoft Graph SDK v2 GA | https://devblogs.microsoft.com/microsoft365dev/upgrade-to-microsoft-graph-powershell-sdk-v2-now-generally-available/ | 2.0.0 リリース日（2023-07-05） |
| PnP PowerShell Installation | https://pnp.github.io/powershell/articles/installation.html | PS バージョン要件 |
| Exchange Online PowerShell | https://learn.microsoft.com/en-us/powershell/exchange/connect-to-exchange-online-powershell | UseRpsSession 非推奨 |
