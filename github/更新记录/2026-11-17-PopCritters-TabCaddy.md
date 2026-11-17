# 2026-11-17 更新记录：PopCritters / TabCaddy 支持页与隐私政策页

> 本文记录 2026-11-17 全天对 `cliechen/app-support` 支持站点的修改，以及 PopCritters 隐私清单（`PrivacyInfo.xcprivacy`）的代码改动。
> 后续任何智能体接手，从本文最后一节「继续工作入口」开始即可。

---

## 1. 目标站点

| 项目 | 值 |
|------|-----|
| 仓库 | `https://github.com/cliechen/app-support` |
| 分支 | `main` |
| Pages 源 | `docs/` 目录 |
| 支持页 | `docs/index.html` → https://cliechen.github.io/app-support/ |
| 隐私页 | `docs/privacy.html` → https://cliechen.github.io/app-support/privacy |
| 本地克隆 | `/Users/mac/Documents/apps/app-support` |
| 今日改动文件 | 仅 `docs/index.html`、`docs/privacy.html` |

当日线上状态：已发布，`Effective: November 17, 2026`。

---

## 2. 今日提交（20 个，全部在 main）

| 时间 | 提交 | 说明 |
|------|------|------|
| 02:17 | `ce25279` | 支持页加入 PopCritters |
| 02:17 | `2d24ecd` | 隐私页加入 PopCritters |
| 02:21 | `31a9ed1` | 支持页更新 PopCritters 广告模式 |
| 02:21 | `2f16e97` | 隐私页更新 PopCritters 广告模式 |
| 13:37 | `16591de` | 隐私页换成 PopCritters 草稿版政策 |
| 13:37 | `01c14c6` | 调整 App 排序并加入 PopCritters FAQ |
| 13:43 | `66839de` | 重建隐私页 HTML 结构 |
| 13:57 | `4048c4f` | PopCritters FAQ 双语（EN/ZH） |
| 13:57 | `42d41de` | 修正 IDFA 跟踪表述 |
| 14:01 | `a292dcd` | 修正 FAQ 奖励与备份措辞 |
| 14:10 | `2e5fad6` | 明确 TabCaddy 无付费去广告 |
| 14:16 | `c42009b` | 明确 TabCaddy 广告类型（横幅 + 插屏） |
| 14:22 | `c55ddbf` | TabCaddy 隐私页对齐修复（P0 + P1） |
| 14:22 | `d77faee` | 新增 TabCaddy FAQ（EN/ZH） |
| 15:55 | `3e813de` | 修正 TabCaddy 隐私页矛盾点 |
| 15:55 | `240e994` | 修正 TabCaddy FAQ 矛盾点（清空历史 + 离线范围） |
| 15:59 | `3ce3b61` | 恢复丢失的 Privacy Policy `h1` |
| 16:19 | `73cbce5` | 修正 PopCritters 同意路径与数据披露；FAQ 分组成卡 |
| 16:42 | `b5d5aa8` | **页面标题改为 PopCritters 优先；数据披露同步** |
| 16:48 | `52fb310` | **第三方共享段落对齐 Cash App 载荷** |
| 20:35 | `0d443ea` | **中立化主标题，兼容 PopCritters 与 TabCaddy 审核** |

加粗三条为本轮（页面复查与双 App 提审合规）产生的提交。

---

## 3. 本轮页面修改明细（`b5d5aa8` + `52fb310`）

### 3.1 PopCritters 提到最上面

- `docs/index.html`：`<title>` 与 `<h1>` → `PopCritters App Support`；首张 App 卡片即 PopCritters（原本已在首位，此次显式化到标题层）。
- `docs/privacy.html`：`<title>` 与 `<h1>` → `PopCritters Privacy Policy`；「适用范围」App 列表把 PopCritters 从末尾移到第一行；「Local Data Storage」列表同样把 PopCritters 移到第一行。

### 3.2 PopCritters 本地数据清单补全

原清单缺一大块，已按代码补齐：

- 新增：`daily challenge`、`daily mission and Mystery Chest state`、`in-progress level session`、`local ad-reward ledger`（记录哪次激励奖励已入账及时间）。
- 保留：游戏进度、关卡解锁与星级、金币余额、道具库存、成就、登录连续、每日特惠、主题、音效/音乐/触感/无障碍偏好、最多 500 条本地游戏与广告事件日志。
- 依据：`Sources/StorageKeys.swift`、`Sources/ProductProgress.swift`、`Sources/AdRewardStore.swift`（`critterpop_ad_reward_transactions_v2`）、`Sources/LevelManager.swift`（`activeSession`，UserDefaults）、`Sources/GameAnalytics.swift:41`（`suffix(500)`）。

### 3.3 广告与联网表述修正

- 广告段落开头从「Our apps are free to use and supported by advertising」改为「**PopCritters** 和 **TabCaddy** are free to use and supported by advertising」，避免替无广告 App 声明。
- UMP 触发条件：由「has recorded your choice for your region」改为「reports that ads may be requested for your region（有同意表单的地区在作答之后）；需要同意但尚未同意时不请求广告」。
  - 依据：`Sources/AdConsentManager.swift` 使用 UMP 的 `canRequestAds`，而非本地布尔值。
- PopCritters 广告形态：保持「仅空道具激励视频、无横幅无插屏、本版本」。依据：`Sources/RewardedAds.swift:82`（`isEnabledFirstLaunch` 只允许 `.emptyBooster`）、`:133`（`bannersEnabled = false`）、`:232`（每日 3 次、冷却 180s、最低关卡 1）。

### 3.4 TabCaddy 表述修正

- Venmo 校验时机：`save a handle` → 「**after you pause typing**」/「输入 Venmo 用户名时的校验」。依据：`TabCaddy/Features/Settings/SettingsSheetView.swift:87`（`onChange` 触发）、`:98`（600ms 防抖 HEAD 请求）。
- 支付链接载荷：明确「备注只在跳转 Venmo 时携带；Cash App 只收用户名与人均金额」。依据：`VenmoDeepLinkBuilder.nativeURL`（含 `note`）与 `cashAppURL`（`https://cash.app/$handle/amount`，无备注）。
- 同页第 119 行「Third-Party Services」段落原仍写「amount per person **and your optional note** to Venmo or Cash App」，与上一处矛盾，已在 `52fb310` 一并修正。

### 3.5 结构修复（`73cbce5`，本轮前一次）

- 新增「Help & FAQ」分组标题，两张 FAQ 卡收进 `.app-list`（获得 16px 间距），修掉 `</div><div class="contact">` 闭合错位。
- 新增锚点 `#popcritters-faq`、`#tabcaddy-faq`。
- 各补一条中英双语 FAQ：PopCritters「如何更改广告隐私选择」、TabCaddy「账单数据存哪、是否上传」。

### 3.6 主标题中立化与双 App 审核对齐

- `docs/index.html`：`<title>` 改为 `App Support | PopCritters &amp; TabCaddy`；`<h1>` 改为 `App Support`。
- `docs/privacy.html`：`<title>` 改为 `Privacy Policy | PopCritters &amp; TabCaddy`；`<h1>` 改为 `Privacy Policy`。
- 依据与原因：苹果审核员对 Support URL (Guideline 1.5) 和 Privacy Policy URL (Guideline 5.1.1) 审查极为机械。若主标题仅标注单一 App（PopCritters），TabCaddy 审核时极易被审核员误认为链接与提交应用不符而遭到拒审（Metadata Rejected）。中立化 `<h1>` 并将 `<title>` 明确设置为 `PopCritters & TabCaddy`，既能消除 TabCaddy 的误判风险，又能确保两款 App 在审核时在标签页与首屏完全可见。

---

## 4. 与代码逐条核对结论

### PopCritters

| 项目 | 页面写法 | 代码依据 | 结论 |
|------|----------|----------|------|
| 部署目标 | iOS 16+ | `PRODUCT_BUNDLE_IDENTIFIER = com.cliechen.PopCritters` 的 build settings：`IPHONEOS_DEPLOYMENT_TARGET = 16.0` | 一致 |
| 广告形态 | 仅空道具激励视频 | `RewardedAds.swift:82`、`:133` | 一致 |
| 每日上限 | 未写具体次数 | `launchPolicy` 3 次 / 180s / Lv1 | 页面未声明，无冲突 |
| ATT / IDFA | 不请求 ATT、不用 IDFA | 全局无 `requestTrackingAuthorization`，`Info.plist` 无 `NSUserTrackingUsageDescription` | 一致 |
| 同意入口 | Settings → Privacy & Ads → Ad Privacy Choices | `SettingsView.swift:245`、`:280`（`privacyOptionsRequired` 才显示）、`:298` | 一致 |
| MetricKit | 只留本机 | `Sources/PerformanceMonitor.swift`、`SettingsView.swift:984` | 一致 |
| 本地数据 7 类 | 已按上表补全 | `StorageKeys.swift` 等 | 一致 |

### TabCaddy

| 项目 | 页面写法 | 代码依据 | 结论 |
|------|----------|----------|------|
| 部署目标 | iOS 17+ | `IPHONEOS_DEPLOYMENT_TARGET = 17.0` | 一致 |
| ATT | 请求 ATT，拒绝后仅非个性化 | `Info.plist:42` + `AttConsentCoordinator.swift:290` | 一致 |
| 广告形态 | 横幅 + 偶发插屏 | `AdMobBannerView`、`InterstitialAdCoordinator` | 一致 |
| 无付费去广告 | 明确写出 | 无 StoreKit 内购（仅评分提示用到 `StoreKit`） | 一致 |
| 历史 30 条 | 最多 30 条 | `PastChecksStore.swift:16`（`maxChecksCount = 30`） | 一致 |
| 清空历史 | 垃圾桶 + `Clear All Past Checks` | `PastChecksSheetView.swift:60` | 一致 |
| 离线范围 | 仅广告 / Venmo 校验 / 支付链接联网 | `checkVenmoReachable`、`VenmoDeepLinkBuilder` | 一致 |
| 功能名 | Sober Shield、Guest of honor、Junk Fee、小票图、Cash App | `SoberShieldSheetView`、`Diner.isVIP`、`JunkFeeRadarSheetView`、`ImageRenderer`、`cashAppURL` | 一致 |

### 隐私清单（App 侧）

| App | 数据类型 | Linked | Tracking | `NSPrivacyTracking` | 跟踪域名 |
|-----|----------|--------|----------|---------------------|----------|
| PopCritters | 7 类（广告、设备 ID、粗略位置、产品交互、崩溃、性能、其他诊断） | 全部 false | 全部 false | false | 空 |
| TabCaddy | 7 类（同上） | 全部 false | 仅 DeviceID 为 true | true | 5 个（admob / doubleclick / googleads / googlesyndication / googleadservices） |

两者访问原因均只有 `NSPrivacyAccessedAPICategoryUserDefaults` / `CA92.1`。

---

## 5. PopCritters 代码改动（本地，**未提交**）

在 `/Users/mac/Documents/apps/PopCritters`：

| 文件 | 改动 |
|------|------|
| `PrivacyInfo.xcprivacy` | 数据类型 4 类 → 7 类，新增 `CrashData`、`PerformanceData`、`OtherDiagnosticData`；均 `Linked=false`、`Tracking=false`；`NSPrivacyTracking=false`、域名为空、API 原因不变；补注释说明诊断类来自广告 SDK，App 自身 MetricKit 摘要只留设备 |
| `Sources/SettingsView.swift:938` | Release 态应用内「Advertising」说明加入 crash / 性能 / 其他诊断数据；UMP 触发条件改为「reports that ads may be requested …（有同意表单的地区在作答之后）；需要同意但尚未同意时不请求广告」 |
| `en.lproj/Localizable.strings:57` | 同上字符串同步（key 与值都必须逐字一致，否则 `NSLocalizedString` 找不到翻译） |
| `COMPLIANCE_DECISION.md` §6 | 清单描述与 App Privacy 待办改为 7 类，注明须与清单一一对应（该文件为 untracked） |
| `ADMOB_INTEGRATION_SPEC.md:155` | 「四类」→「七类」（该文件为 untracked） |

校验：`plutil -lint` 对 `PrivacyInfo.xcprivacy` 与 `en.lproj/Localizable.strings` 均 OK；`plistlib` 读出 7 类、全部 Not Linked / Not Used for Tracking。

> 该仓库当前有 168 个未提交路径（工程改名、AppIcon、LaunchLogo、资源等），与上述改动无关，故未提交。

---

## 6. 待办（需要在 App Store Connect / Xcode 手动完成）

1. **PopCritters 重新 Archive 并上传**：改的是打包进 App 的清单，只改文件不发新包，线上问卷仍是旧的。
2. **App Privacy 问卷**：勾选 Device ID、Advertising Data、Coarse Location、Product Interaction、Crash Data、Performance Data、Other Diagnostic Data 共 7 类，全部 **Not Linked to You**、**Not Used for Tracking**。
3. **年龄分级**：问卷里 **Contains Ads = Yes**。前提成立——`Release.xcconfig` 为 `ADMOB_PRODUCTION_ADS = YES`（`Debug.xcconfig` 为 `NO`）；若哪天生产广告关闭再提交，需改回 No。
4. ~~**文案小差（可选）**~~ **已完成**：应用内 Release 字符串原写「UMP consent tool **records your choice** for your region」，与网页的「reports that ads may be requested」不一致（在无需同意的地区 UMP 不记录任何选择）。已改 `SettingsView.swift:938` 与 `en.lproj/Localizable.strings:57` 同一条字符串，key 与值同步更新，两侧逐字一致。

---

## 7. 继续工作入口

- **改页面**：本地仓库 `/Users/mac/Documents/apps/app-support`，直接编辑 `docs/*.html` → 提交 → 推 `main` → 等 Pages 重建。
  - 本机沙箱下 `git push` 需提权执行；推成功后可能报 `update_ref failed for ref 'refs/remotes/origin/main'`，那只是远程跟踪引用没写进去，推送本身已成功，下次 `git fetch` 自愈。
  - 另一条路径：用 GitHub Contents API（PAT）直接改，步骤见 `~/Documents/github/skills/GitHub_Pages_更新技能.md`。
- **验证线上**（必须带缓存穿透参数）：
  ```bash
  cb=$(date +%s)
  curl -s "https://cliechen.github.io/app-support/?cb=$cb" | grep -c "预期关键词"
  curl -s "https://cliechen.github.io/app-support/privacy?cb=$cb" | grep -c "预期关键词"
  ```
  同时用 `<h1>` / `<title>` / `<li>` 顺序核对，并检查旧表述计数为 0。
- **结构自检**（HTML 标签闭合）：
  ```bash
  python3 -c "
  from html.parser import HTMLParser
  void={'meta','br','hr','img','input','link'}
  class P(HTMLParser):
      def __init__(s): super().__init__(); s.st=[]; s.err=[]
      def handle_starttag(s,t,a):
          if t not in void: s.st.append(t)
      def handle_endtag(s,t):
          if s.st and s.st[-1]==t: s.st.pop()
          else: s.err.append(t)
  p=P(); p.feed(open('docs/index.html',encoding='utf-8').read()); print('unclosed',p.st,'mismatch',p.err)
  "
  ```
- **改 App 侧文案/清单**：`/Users/mac/Documents/apps/PopCritters`、`/Users/mac/Documents/apps/TabCaddy`，两边都有大量未提交改动，改动前先看清 `git status`，只动目标文件。

---

## 8. 约束（沿用站点既有规范）

- 只做整段字符串替换，不重排整页 HTML。
- 不擅自修改其他 App 的段落。
- 不在聊天、日志或文档中输出明文 PAT、API Key、密码。
- 页面文案必须与 App 实际行为一致；改文案前先在代码里找到依据，并记录文件:行号。
- 提交信息用祈使句、一句话。
