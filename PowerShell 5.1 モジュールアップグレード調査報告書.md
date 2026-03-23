# PowerShell 5.1 モジュールアップグレード調査報告書

**調査日**: 2026-03-20

**対象CSV**: `GlobalMaintenanceTool_20260330release\Dockerfiles\expected_modules.csv`

**制約**: PowerShell 5.1 (Desktop) 互換 / PnP.PowerShell との依存関係維持

---

## 最重要制約：PnP.PowerShell と Az.Accounts の依存関係

過去事例として、Az.Accounts 5.3.2 が同梱する `System.Text.Encodings.Web` のバージョンが

PnP.PowerShell 1.11.0 の期待するバージョンと不整合を起こし、

`MissingMethodException`（`FindFirstCharacterToEncodeUtf8`）が発生した。

PowerShell 5.1 (Desktop) では、先にロードされたアセンブリが AppDomain 全体で共有されるため、

Az.Accounts が読み込んだアセンブリバージョンが PnP.PowerShell と競合する。

**結論:**

- **Az.Accounts 5.x** → PnP.PowerShell 1.x と共存不可

- **Az 14.0.0+** → Az.Accounts >= 5.0.0 を要求するため使用不可

- **Az 13.5.0** → Az.Accounts >= 4.2.0 を要求 → 4.x 系で満たせるため安全

---

## アップグレード提案一覧

### Az 関連モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |

|-----------|-------------|-------------|------|

| Az | 8.0.0 | **13.5.0** | Az.Accounts 4.x で動作する最高版（13.x 系最終版、13.6.0 は存在しない） |

| Az.Accounts | 4.0.2 | **4.2.0** | 4.x 系最終版。5.x は PnP.PowerShell と競合するため不可 |

| Az.Advisor | 1.1.2 | Az 13.5.0 同梱版 | Az インストール時に自動更新 |

| Az.Aks | 4.1.0 | Az 13.5.0 同梱版 | |

| Az.AnalysisServices | 1.1.4 | Az 13.5.0 同梱版 | |

| Az.ApiManagement | 3.0.0 | Az 13.5.0 同梱版 | |

| Az.AppConfiguration | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.ApplicationInsights | 2.0.0 | Az 13.5.0 同梱版 | |

| Az.Attestation | 1.0.0 | Az 13.5.0 同梱版 | |

| Az.Automation | 1.7.3 | Az 13.5.0 同梱版 | |

| Az.Batch | 3.2.0 | Az 13.5.0 同梱版 | |

| Az.Billing | 2.0.0 | Az 13.5.0 同梱版 | |

| Az.Cdn | 2.1.0 | Az 13.5.0 同梱版 | |

| Az.CloudService | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.CognitiveServices | 1.11.0 | Az 13.5.0 同梱版 | |

| Az.Compute | 4.27.0 | **9.3.0** | Az 13.5.0 同梱版 |

| Az.ContainerInstance | 3.1.0 | Az 13.5.0 同梱版 | |

| Az.ContainerRegistry | 3.0.0 | Az 13.5.0 同梱版 | |

| Az.CosmosDB | 1.8.0 | Az 13.5.0 同梱版 | |

| Az.DataBoxEdge | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.Databricks | 1.2.0 | Az 13.5.0 同梱版 | |

| Az.DataFactory | 1.16.7 | Az 13.5.0 同梱版 | |

| Az.DataLakeAnalytics | 1.0.2 | Az 13.5.0 同梱版 | |

| Az.DataLakeStore | 1.3.0 | Az 13.5.0 同梱版 | |

| Az.DataShare | 1.0.1 | Az 13.5.0 同梱版 | |

| Az.DeploymentManager | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.DesktopVirtualization | 3.1.0 | Az 13.5.0 同梱版 | |

| Az.DevTestLabs | 1.0.2 | Az 13.5.0 同梱版 | |

| Az.Dns | 1.1.2 | Az 13.5.0 同梱版 | |

| Az.EventGrid | 1.3.0 | Az 13.5.0 同梱版 | |

| Az.EventHub | 2.0.0 | Az 13.5.0 同梱版 | |

| Az.FrontDoor | 1.9.0 | Az 13.5.0 同梱版 | |

| Az.Functions | 4.0.3 | Az 13.5.0 同梱版 | |

| Az.HDInsight | 5.0.1 | Az 13.5.0 同梱版 | |

| Az.HealthcareApis | 2.0.0 | Az 13.5.0 同梱版 | |

| Az.IotHub | 2.7.4 | Az 13.5.0 同梱版 | |

| Az.KeyVault | 4.5.0 | **6.3.1** | Az 13.5.0 同梱版（+2 メジャー、破壊的変更確認要） |

| Az.Kusto | 2.1.0 | Az 13.5.0 同梱版 | |

| Az.LogicApp | 1.5.0 | Az 13.5.0 同梱版 | |

| Az.MachineLearning | 1.1.3 | Az 13.5.0 同梱版 | |

| Az.Maintenance | 1.2.0 | Az 13.5.0 同梱版 | |

| Az.ManagedServiceIdentity | 1.0.0 | Az 13.5.0 同梱版 | |

| Az.ManagedServices | 3.0.0 | Az 13.5.0 同梱版 | |

| Az.MarketplaceOrdering | 1.0.2 | Az 13.5.0 同梱版 | |

| Az.Media | 1.1.1 | Az 13.5.0 同梱版 | |

| Az.Migrate | 1.1.2 | Az 13.5.0 同梱版 | |

| Az.Monitor | 3.0.1 | Az 13.5.0 同梱版 | |

| Az.MySql | 1.0.0 | Az 13.5.0 同梱版 | |

| Az.Network | 4.17.0 | **7.16.1** | Az 13.5.0 同梱版 |

| Az.NotificationHubs | 1.1.1 | Az 13.5.0 同梱版 | |

| Az.OperationalInsights | 3.1.0 | Az 13.5.0 同梱版 | |

| Az.PolicyInsights | 1.5.0 | Az 13.5.0 同梱版 | |

| Az.PostgreSql | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.PowerBIEmbedded | 1.1.2 | Az 13.5.0 同梱版 | |

| Az.PrivateDns | 1.0.3 | Az 13.5.0 同梱版 | |

| Az.RecoveryServices | 5.4.0 | Az 13.5.0 同梱版 | |

| Az.RedisCache | 1.6.0 | Az 13.5.0 同梱版 | |

| Az.RedisEnterpriseCache | 1.0.0 | Az 13.5.0 同梱版 | |

| Az.Relay | 1.0.3 | Az 13.5.0 同梱版 | |

| Az.ResourceMover | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.Resources | 6.0.0 | **7.11.0** | Az 13.5.0 同梱版 |

| Az.Security | 1.3.0 | Az 13.5.0 同梱版 | |

| Az.SecurityInsights | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.ServiceBus | 1.9.0 | Az 13.5.0 同梱版 | |

| Az.ServiceFabric | 3.0.2 | Az 13.5.0 同梱版 | |

| Az.SignalR | 1.4.1 | Az 13.5.0 同梱版 | |

| Az.Sql | 3.9.0 | **6.0.3** | Az 13.5.0 同梱版 |

| Az.SqlVirtualMachine | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.StackHCI | 1.1.1 | Az 13.5.0 同梱版 | |

| Az.Storage | 4.6.0 | **8.4.0** | Az 13.5.0 同梱版 |

| Az.StorageSync | 1.7.0 | Az 13.5.0 同梱版 | |

| Az.StreamAnalytics | 2.0.0 | Az 13.5.0 同梱版 | |

| Az.Support | 1.0.0 | Az 13.5.0 同梱版 | |

| Az.Synapse | 1.4.0 | Az 13.5.0 同梱版 | |

| Az.TrafficManager | 1.1.0 | Az 13.5.0 同梱版 | |

| Az.Websites | 2.11.2 | **3.4.1** | Az 13.5.0 同梱版（+1 メジャー、破壊的変更確認要） |

### Microsoft 365 関連モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |

|-----------|-------------|-------------|------|

| PnP.PowerShell | 1.11.0 | **1.12.0** | PS 5.1 対応の最終版（2.x は PS 7.2+、3.x は PS 7.4+ 必須） |

| Microsoft.Graph（全サブモジュール） | 1.27.0 | **1.28.0** | 1.x 最終版。2.x (2.36.1) も PS5.1 対応だが破壊的変更あり |

| ExchangeOnlineManagement | 3.1.0 | **3.9.2** | 同一メジャー版内の更新 |

| MicrosoftTeams | 6.1.0 | **7.6.0** | PS 5.1 対応確認済み |

| Microsoft.Online.SharePoint.PowerShell | 16.0.27011.12008 | **変更不要** | 既に最新 |

| Microsoft.PowerApps.Administration.PowerShell | 2.0.168 | **2.0.216** | |

### Power BI 関連モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |

|-----------|-------------|-------------|------|

| MicrosoftPowerBIMgmt | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Admin | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Capacities | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Data | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Profile | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Reports | 1.2.1111 | **1.3.83** | |

| MicrosoftPowerBIMgmt.Workspaces | 1.2.1111 | **1.3.83** | |

### その他モジュール

| モジュール | 現行バージョン | 提案バージョン | 備考 |

|-----------|-------------|-------------|------|

| MSAL.PS | 4.37.0.0 | **変更不要** | 既に最新（2021年以降更新なし） |

| PackageManagement | 1.4.8.1 | **変更不要** | 既に最新 |

| PowerShellGet | 2.2.5 | **変更不要** | 2.x 系では最新（3.x は別モジュール） |

| SqlServer | 21.1.18256 | **22.4.5.1** | |

---

## リスク評価

### 高リスク（要テスト）

1. **Az 8.0.0 → 13.5.0**

   - メジャーバージョンを5つ跨ぐため、破壊的変更の可能性あり

   - 既知の破壊的変更例: `Resolve-Error` エイリアス削除（Az.Accounts 4.0.0 で実施、Az 13.0.0 が Az.Accounts >= 4.0.0 を要求するため影響）

   - **必ず Az.Accounts 4.2.0 + PnP.PowerShell 1.12.0 の組み合わせを事前検証すること**

   - Az.Accounts 4.2.0 で問題が出る場合、Az 13.0.0 (Az.Accounts >= 4.0.0) + Az.Accounts 4.0.2 固定にフォールバック可能

2. **Microsoft.Graph 1.x の EOL リスク（セキュリティ上の高リスク）**

   - 1.28.0 は **2023年6月8日が最終リリース** であり、調査時点で約2年9ヶ月間セキュリティパッチが提供されていない

   - Microsoft は 2.x への移行を推奨しており、1.x へのセキュリティ修正は提供されない可能性が高い

   - 2.x は PS 5.1 に対応しているため PS バージョンの制約にはならないが、コマンドレット名・パラメータ・戻り値に破壊的変更があるためスクリプト改修が必要

   - 今回は 1.28.0（1.x 最終版）を採用するが、**セキュリティリスクを受容した上での判断であり、2.x 移行計画を早急に策定すべき**

### 中リスク

3. **MicrosoftTeams 6.x → 7.x**

   - メジャーバージョン更新のため動作確認推奨

### 低リスク

4. ExchangeOnlineManagement 3.1.0 → 3.9.2（マイナー更新）

5. MicrosoftPowerBIMgmt 1.2.1111 → 1.3.83（マイナー更新）

6. SqlServer 21.x → 22.x（メジャーだが影響範囲が限定的）

7. PnP.PowerShell 1.11.0 → 1.12.0（マイナー更新）

8. Microsoft.PowerApps.Administration.PowerShell 2.0.168 → 2.0.216（パッチ更新）

---

## Az モジュール依存関係マップ

| Az バージョン | 要求する Az.Accounts | PnP.PowerShell 共存 |

|-------------|-------------------|-------------------|

| 8.0.0（現行） | >= 2.8.0 | ○ |

| 12.0.0 | >= 3.0.0 | ○ |

| 12.5.0 | >= 3.0.5 | ○ |

| 13.0.0 | >= 4.0.0 | ○（Az.Accounts 4.0.2 で可） |

| **13.5.0（推奨）** | **>= 4.2.0** | **○（Az.Accounts 4.2.0 で可）** |

| 14.0.0 | >= 5.0.0 | **✕（5.x は PnP と競合）** |

| 15.4.0（最新） | >= 5.3.3 | **✕** |

**補足**: Az 13.5.0 は 13.x 系の最終版であり、13.6.0 以降は公開されていない（2026-03-23 時点）。次のバージョンは Az 14.0.0 であり、Az.Accounts 5.x を要求するため PnP.PowerShell との共存が不可。したがって **Az 13.5.0 が PnP.PowerShell と共存できる理論上の上限** である。

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

## クロスチェック結果（2026-03-20 追記）

PowerShell Gallery の公開情報をもとに本報告書の内容を再検証した結果を以下に記す。

### 修正済みの誤り

Az 13.5.0 に同梱されるサブモジュールのバージョンに **6件の誤り** があり修正した:

| モジュール | 旧（誤り） | 修正後（正） |

|-----------|----------|-----------|

| Az.Compute | 8.6.0 | 9.3.0 |

| Az.KeyVault | 6.3.0 | 6.3.1 |

| Az.Network | 7.12.0 | 7.16.1 |

| Az.Resources | 7.7.0 | 7.11.0 |

| Az.Sql | 5.4.0 | 6.0.3 |

| Az.Storage | 7.6.0 | 8.4.0 |

**影響**: サブモジュールのメジャーバージョンが想定より大きいケースがあるため（特に Az.Compute 9.x, Az.Storage 8.x, Az.Sql 6.x）、破壊的変更の確認範囲が報告書初版より広くなる。

### 検証済み（正確であることを確認）

- Az 13.5.0 の存在と Az.Accounts >= 4.2.0 の要求 → **正確**

- Az 14.0.0 が Az.Accounts >= 5.0.0 を要求 → **正確**

- Az 15.4.0 が最新版（2026-03-03 公開）→ **正確**

- PnP.PowerShell 1.12.0 が PS 5.1 対応の最終 1.x 版 → **正確**

- Microsoft.Graph 1.28.0 が 1.x 最終版 → **正確**

- Microsoft.Graph 2.x が PS 5.1 対応（最新 2.36.1）→ **正確**

- ExchangeOnlineManagement 3.9.2 が最新 GA → **正確**

- MicrosoftTeams 7.6.0 が PS 5.1 対応 → **正確**

- Microsoft.Online.SharePoint.PowerShell 16.0.27011.12008 が最新 → **正確**

- Microsoft.PowerApps.Administration.PowerShell 2.0.216 が最新 → **正確**

- MicrosoftPowerBIMgmt 1.3.83 が最新 → **正確**

- MSAL.PS 4.37.0.0 が最新（2021年以降更新なし）→ **正確**

- SqlServer 22.4.5.1 が最新、PS 5.1 対応 → **正確**

- PackageManagement 1.4.8.1 が最新 → **正確**

- PowerShellGet 2.2.5 が 2.x 最新 → **正確**

- `FindFirstCharacterToEncodeUtf8` MissingMethodException → **実在する既知の問題**（pnp/powershell#727、PowerShell/vscode-powershell#3510 等で報告）

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

PnP.PowerShell は既に **3.x**（PS 7.4.0 以上必須）がリリースされている。報告書では「2.x 以降は PS7+ 必須」と記載しているが、**3.x が最新メジャーライン** であることを認識しておくべき。将来の PS 7 移行計画では 3.x を検討対象とすること。

### 3. Az サブモジュールのメジャーバージョンジャンプが大きい

修正後のバージョンでは、以下のモジュールで **2世代以上のメジャーバージョンジャンプ** が発生する:

| モジュール | 現行 → 提案 | メジャー差 |

|-----------|-----------|----------|

| Az.Compute | 4.27.0 → 9.3.0 | +5 |

| Az.Storage | 4.6.0 → 8.4.0 | +4 |

| Az.Network | 4.17.0 → 7.16.1 | +3 |

| Az.Sql | 3.9.0 → 6.0.3 | +3 |

| Az.KeyVault | 4.5.0 → 6.3.1 | +2 |

| Az.Websites | 2.11.2 → 3.4.1 | +1 |

各モジュールの Breaking Changes ドキュメントを確認し、使用しているコマンドレットへの影響を事前に洗い出すこと。

**特に Az.KeyVault は +2 メジャーの破壊的変更があり、シークレット管理・証明書管理のコマンドレットで引数や戻り値の変更が発生している可能性が高い。優先的に確認すること。**

### 4. MSAL.PS の保守終了リスク

MSAL.PS は **2021年11月を最後に更新が停止** しており、実質的にメンテナンスされていない。Microsoft 公式サポート対象外のコミュニティモジュールである。依存している場合、将来的に認証フローの非互換（Azure AD → Entra ID 移行等）が発生するリスクがある。代替として `Microsoft.Identity.Client`（MSAL.NET）の直接利用や、Az.Accounts 組み込みの認証機能への移行を検討すべき。

### 5. Microsoft.Graph 1.28.0 の EOL リスク（高リスク）

Microsoft.Graph 1.x は **2023年6月が最終リリース** であり、Microsoft は 2.x への移行を推奨している。セキュリティ修正も 2.x のみに提供される可能性が高い。

**1.x を継続利用するリスク**:
- 約2年9ヶ月間セキュリティパッチが未提供。今後 CVE が発見されても修正されない
- Microsoft Graph API 側の変更により、将来的に 1.x のコマンドレットが動作しなくなる可能性

**2.x 移行の技術的障壁**:
- 2.x は PS 5.1 に対応しているため、PS バージョンの制約にはならない
- ただしコマンドレット名（Beta 系が `*-MgBeta*` にリネーム）、パラメータ、戻り値に破壊的変更がある
- スクリプト改修が必要（改修量はスクリプトの複雑度に依存）

今回は 1.28.0 を採用するが、**早期に 2.x 移行計画を策定すること** を強く推奨する。

### 6. SqlServer モジュールのメジャーバージョン更新

SqlServer 21.x → 22.x はメジャー更新だが、報告書では「低リスク（影響範囲が限定的）」と評価している。ただし、22.x では `Invoke-Sqlcmd` のデフォルト動作変更（`-TrustServerCertificate` の扱い等）がある可能性があるため、SQL Server 接続を行うスクリプトの動作確認を推奨する。

### 7. ExchangeOnlineManagement の UseRpsSession 非推奨化

ExchangeOnlineManagement 3.9.2 では `UseRpsSession` パラメータが非推奨（deprecated）となっている。既存スクリプトで `Connect-ExchangeOnline -UseRPSSession` を使用している場合、REST ベースの接続への移行が必要。PS 5.1 環境では REST ベース接続がデフォルトとなるが、一部のコマンドレットの出力形式が異なる場合がある。

---

## 次のステップ

1. テスト環境で Az.Accounts 4.2.0 + PnP.PowerShell 1.12.0 の互換性を検証（**アセンブリバージョンの実機確認を含む**）

2. 問題があれば Az 13.0.0 + Az.Accounts 4.0.2 にフォールバック

3. Az.KeyVault / Az.Compute / Az.Storage / Az.Network / Az.Sql / Az.Websites の **Breaking Changes ドキュメントを確認**し、使用コマンドレットへの影響を調査

4. 検証通過後、expected_modules.csv と Dockerfile を更新（**Az.Accounts → Az の順序でインストール**）

5. Microsoft.Graph 2.x 移行計画を策定（**PS 5.1 対応済みのため技術的障壁は低いが、スクリプト改修が必要**）

6. MSAL.PS の代替手段の調査・計画

7. ExchangeOnlineManagement の UseRpsSession 非推奨化に伴うスクリプト影響調査
