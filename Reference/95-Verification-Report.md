# 95 - 文档事实核查报告（Docs/Lyra 10~80）

**核查日期**: 2026-10-04
**核查方式**: 对磁盘上的 C++ / `.uplugin` / `Config/*.ini` 逐条打开核对（未使用 unreal-mcp，未修改任何源码/配置/资产）。
**核对口径**: `file:line` 允许 ±5 行误差；行号若指向完全不同的函数/语句，或超出文件行数，判为错误。

---

## 汇总

| 文档 | 抽查条目 | 判错 | 已修正 | 准确度评级 |
|---|---|---|---|---|
| 10-CoreFramework | 254 个 `file:line` 自动比对 + 约 30 条关键论断人工核对 | 9 | 9 | **A-** |
| 20-AbilitySystem-Input | 276 个 `file:line` + 约 30 条论断 | 2 | 2 | **A** |
| 30-GameplaySystems | 280 个 `file:line` + 约 30 条论断（含 5 条"坑"） | 2 | 2 | **A** |
| 40-UI | 208 个 `file:line` + 约 25 条论断 | 约 40（集中在 CommonGame 插件，系统性行号偏移） | 40 | **C+**（修正后 A-） |
| 50-Settings-Online-Platform | 209 个 `file:line` + 约 25 条论断 | 19 | 19 | **B** |
| 60-Presentation | 283 个 `file:line` + 约 30 条论断（含争议项 a、b） | 6 | 6 | **A-** |
| 70-GameFeatures-Content | 18 条（.uplugin / uproject / ini / C++） | 2 | 2 | **A-** |
| 80-Build-Config | 16 条（Target.cs / Build.cs / ini） | 4 | 4 | **B+** |

---

## 1. 指定的争议论断

### (a) 60：`ULyraNumberPopComponent_NiagaraText` / `ULyraDamagePopStyleNiagara` 是否"非原版"
**原文错误**。两个类都在 `Source/LyraGame/Feedback/NumberPops/` 下，与 MeshText 版同目录同风格（`LyraNumberPopComponent_NiagaraText.{h,cpp}`、`LyraDamagePopStyleNiagara.h`），代码无改动痕迹（Epic 版权头、`UE_API`/`MinimalAPI` 风格一致）。即原版 Lyra 同时提供 MeshText 与 NiagaraText 两套。

- 60-Presentation.md:434 「若原版 Lyra 没有这两个类，则这是本项目/新版本引入。」→「这两个类属于原版 Lyra 自带(与 MeshText 版同属 Feedback/NumberPops)，并非本项目自定义；代码未见明显改动。」
- 60:391 「MeshText 方案(原版)」→「MeshText 方案(Lyra 自带)」
- 60:434 「与原版 MeshText 方案并存」→「与 MeshText 方案并存(二者均为 Lyra 自带)」
- 仍然正确的部分：当前内容实际使用 Niagara 版（`B_NiagaraNumberPopComponent`），MeshText 版 C++ 存在但资产未用；MeshText 需 `Styles` 且 `bOverrideMesh=true`，否则 `DetermineStaticMesh` 返回空（`LyraDamagePopStyle.h:34/41` 确认 `EditCondition=bOverrideMesh`、`bOverrideMesh = false`）。

### (b) 60：`GetBlendInfo` 读 `CameraModeStack.Last()` 是栈底而非栈顶
**原文正确，已核实**。`LyraCameraMode.cpp:318` `CameraModeStack.Insert(CameraMode, 0)` —— 新模式插在索引 0，所以 `[0]` 是栈顶；`:321` `CameraModeStack.Last()->SetBlendWeight(1.0f)` 明确把"栈底"权重固定为 1；`BlendStack`（:424 起）也是 "Start at the bottom"，从 `StackSize-1` 开始混合。而 `GetBlendInfo`（:449）里 `TopEntry = CameraModeStack.Last()`（:459）取的确实是栈底，变量名 `TopEntry` / 参数名 `OutWeightOfTopLayer` 与头文件 "top layer" 注释都与实现不符。文档 60:157 的行号（.cpp:459）与结论均正确，未改动。

### (c) 30：`FLyraInventoryList::AddEntry(Instance)` 未实现、`LastFireTime` 从未赋值
**两条均正确**。

- `LyraInventoryManagerComponent.cpp:110-113`：`void FLyraInventoryList::AddEntry(ULyraInventoryItemInstance* Instance) { unimplemented(); }` —— 确认。`IPickupable.cpp:49-52` 的 `FPickupInstance` 路径确实调 `AddItemInstance` → 该函数。
- `LastFireTime` 全工程仅两处引用：`LyraRangedWeaponInstance.h:194` 声明 `double LastFireTime = 0.0;`、`.cpp:150` `GetWorld()->TimeSince(LastFireTime)`，无任何写入点 —— 确认 `SpreadRecoveryCooldownDelay` 实际不生效。

### (d) 20：`InitializePlayerInput` 只在 `bRegisterWithSettings==true` 时 `AddMappingContext`
**原文正确但不完整，已补全**。`LyraHeroComponent.cpp:256` `if (Mapping.bRegisterWithSettings)` 之内同时包含 `Settings->RegisterInputMappingContext(IMC)`（:259-261）**和** `Subsystem->AddMappingContext(IMC, Priority, Options)`（:266）。flag 为 false 时两件事都不做，该 IMC 被完全跳过（`LoadSynchronous` 仍执行）。另外整个循环嵌在 `if (PawnData->InputConfig)` 内（:250），InputConfig 为空则连循环都不进。`ClearAllMappings()` 在 :244，行号正确。

- 20:404 补充为：「…(flag 为 false 时既不 RegisterInputMappingContext 也不 AddMappingContext，直接跳过；且整个循环还要求 PawnData->InputConfig 非空)」

### (e) 10：客户端经 `CurrentExperience` 的 OnRep 加载 Experience，`LoadState` 不复制
**完全正确**。`LyraExperienceManagerComponent.h:82` `UPROPERTY(ReplicatedUsing=OnRep_CurrentExperience)`，`.h:85` `ELyraExperienceLoadState LoadState` 为普通 private 成员（无 UPROPERTY/复制标记）；`.cpp:377` `DOREPLIFETIME(ThisClass, CurrentExperience)` 是唯一复制项；`.cpp:118-121` `OnRep_CurrentExperience()` → `StartExperienceLoad()`，与服务器 `.cpp:67` 的同一调用并列 —— 两端各自独立加载的描述成立。

---

## 2. 逐文档修正清单

### 10-CoreFramework.md（9 处）

| 行 | 旧 | 新 | 依据 |
|---|---|---|---|
| 417 | `AsyncAction_ExperienceReady.cpp:338` | `:84` | 文件仅 95 行；`Step3_HandleExperienceLoaded` 在 :84 |
| 419 | `BotCreation.cpp:95` | `:32` | `OnExperienceLoaded` 在 :32（:29 注册 LowPriority 回调） |
| 419 | `SpawnOneBot (:161)` | `(:98)` | :161 是 `ServerCreateBots` 的 `#else`(非服务器) 分支尾部 |
| 515 | `PawnExt.cpp:233-245` | `:105-140` | `InitializeAbilitySystem` 在 :105，踢旧 Avatar 在 :119-137 |
| 588 | `PawnExt.cpp:233` | `:119-137` | 同上 |
| 540 | `PawnExt.cpp:398` | `:292` | 文件仅 312 行；`OnAbilitySystemInitialized_RegisterAndCall` 在 :292 |
| 579 | `PawnExt.cpp:368` | `:262` | `HaveAllFeaturesReachedInitState` 调用在 :262 |
| 537 | 「`ControllerCanRestart`(.cpp:218) 是私有非虚」 | 「`ULyraPlayerSpawningManagerComponent::ControllerCanRestart`(.h:48, .cpp:153) 是私有非虚」 | .h:48 private + :50 `friend class ALyraGameMode`；原行号 218 指向无关代码 |
| — | （未改）`GameMode.h:69` 虚函数 `ControllerCanRestart` | 正确 | `LyraGameMode.h:69` |

未能核实/保留：10:36 `LyraEditorEngine.cpp:42` 实际位于 `Source/LyraEditor/`（不在 LyraGame 下），`FirstTickSetup` 确在 :42，行号正确，仅路径标注含糊。

### 20-AbilitySystem-Input.md（2 处）

| 行 | 旧 | 新 | 依据 |
|---|---|---|---|
| 527 | `K2_EndAbility (:544)` | 「引擎侧 `UGameplayAbility::K2_EndAbility`(非 Lyra 代码，原 :544 无此函数)」 | `LyraGameplayAbility.cpp` 共 545 行，:544 是 `EndAbility` 相机清理的收尾花括号 |
| 404 | 见争议项 (d) | 补全 false 分支行为 | `LyraHeroComponent.cpp:250-267` |

抽查正确的代表性条目：ASC `ProcessAbilityInput`(:216)/`TryActivateAbilitiesOnSpawn`(:85)/`NotifyAbilityFailed`(:330)/`AddDynamicTagGameplayEffect`(:482)/`NotifyAbilityEnded`(:346)/`WhileInputActive`(:230-245)/`InputReleased`(:295-301)；`LyraAbilitySet.cpp:32/73`；`CanChangeActivationGroup`(:467)、`ChangeActivationGroup`(:497)、`CheckCost`(:202)、`EndAbility`(:195)、`GetHeroComponentFromActorInfo`(:97)、`bClientShouldActivate/bServerShouldActivate`(:451-460)；`LyraHealthSet.cpp:80-82`(免伤)、`:113-122`(GodMode `#if !UE_BUILD_SHIPPING` → MinimumHealth=1)、`:150-155`(Healing 转 Health)；`DefaultInput.ini:49-52`、`DefaultGame.ini:13`。

### 30-GameplaySystems.md（2 处）

| 行 | 旧 | 新 | 依据 |
|---|---|---|---|
| 709 | `ULyraDamageExecution`（`:96`） | `（LyraDamageExecution.cpp:89-98）` | `:96` 落在 `LyraTeamSubsystem.cpp`（同行上下文文件）的无关行；队伤规则在 DamageExecution :89-98 |
| 820 | 「`AddItemDefinition` 新建 Instance（`InventoryManagerComponent.cpp:80-108`）」 | 「`AddItemDefinition` → `FLyraInventoryList::AddEntry(ItemDef)` …（`LyraInventoryManagerComponent.cpp:80-108`）」 | :80 是 `AddEntry(ItemDef)`，`AddItemDefinition` 在 :165 |

抽查正确的代表性条目（含全部"坑"）：`AddEntry(Instance)` unimplemented(:110-113)；`InteractionStatics.cpp:29-32` unimplemented；`CanCauseDamage` 仅 `DifferentTeams` 返回 true + 无队伍但有 ASC 的 `@TODO` 放行（`LyraTeamSubsystem.cpp:345-370`）；`GetTeamTagStackCount` = Public+Private 相加（:310-319）；`SetGenericTeamId` 权威端限制（`LyraPlayerState.cpp:234-250`）与有 Controller 时拒绝（`LyraCharacter.cpp:482-501`）；`bIsTargetDataValid` 硬编码 `true`（`LyraGameplayAbility_RangedWeapon.cpp:495`，服务器命中校验缺失 —— 坑成立）；`QuickBar.cpp:36-41/84/109/169/181`；`AbilityTask_WaitForInteractableTargets.cpp:105-150`；`ULyraAbilityCost_ItemTagStack::ApplyCost` 仅 `IsNetAuthority()`(:41-44)。

### 40-UI.md（约 40 处，系统性问题）

**根因**：CommonGame 插件的行号整体按一个大得多的旧版本标注（例如 `GameUIPolicy.cpp` 实际 207 行，文档引用到 :426/:433；`PrimaryGameLayout.cpp` 实际 132 行，引用到 :230/:261/:270）。逐一改为磁盘实际行号：

| 符号 | 旧 | 新 |
|---|---|---|
| `UGameUIPolicy`(类声明) | GameUIPolicy.h:187 | :57 |
| `RootViewportLayouts` | GameUIPolicy.h:226 | :97 |
| `GetRootLayout` | GameUIPolicy.cpp:282 | :45 |
| `NotifyPlayerAdded` / 其 lambda | :288 / :288-314 / :292-302 | :51 / :53-66 / :57-61 |
| `NotifyPlayerRemoved` | :316 | :79 |
| `NotifyPlayerDestroyed` | :345 | :108 |
| `AddLayoutToViewport` / `AddToPlayerScreen(1000)` | :361 / :366 | :124 / :129 |
| `RemoveLayoutFromViewport` | :371 | :134 |
| `OnRootLayoutAddedToViewport` | :388 | :151 |
| `CreateLayoutWidget` / `ensure` / `CreateWidget` / `Emplace` | :426 / :431 / :433 / :434 | :189 / :194 / :196 / :197 |
| `UPrimaryGameLayout::RegisterLayer` | PrimaryGameLayout.cpp:230 / :230-259 | :89 / :89-118 |
| `OnWidgetStackTransitioning` / SuspendInput | :244 / :248 | :103 / :107 |
| `FindAndRemoveWidgetFromLayer` | :261 | :120 |
| `GetLayerWidget` | :270 | :129 |
| 静态 `GetPrimaryGameLayout*` | :158-191 / :165→:170 | :17-45 / :24→:29 |
| Dormant 逻辑 | :198 | :57-87 |
| 转场时长强制 0 的注释 | :235-238 | :97 |
| `UCommonLocalPlayer`(类) / `GetRootUILayout` / `CallAndRegister_OnPlayerStateSet` | CommonLocalPlayer.h:293 / cpp:398 / cpp:362 | h:20 / cpp:70 / cpp:34 |
| `ACommonPlayerController`(类) / `ReceivedPlayer` / `OnPlayerControllerSet.Broadcast` | h:24 / cpp:456 / :462 | h:15 / cpp:16 / :22 |
| `UGameUIManagerSubsystem::Initialize` / `ShouldCreateSubsystem` / `NotifyPlayerAdded` | cpp:67 / :85 / :99 | :13 / :31 / :45 |

**论断本身全部正确**（已逐条核对源码）：`ShouldCreateSubsystem` 对专用服务器返回 false 且只在无子类时创建（GameUIManagerSubsystem.cpp:31-43）；根布局必须等 PC（`CreateLayoutWidget` 的 `if (GetPlayerController)`，:191）；`RegisterLayer` 强制 `SetTransitionDuration(0)`（:97）；`OnRootLayoutAddedToViewport` 仅 `#if WITH_EDITOR` + 主玩家（:153-158）；Dormant/`SingleToggle` 逻辑基本被注释掉（`OnIsDormantChanged` :73-87 全是注释）。
UIExtension / LoadingScreen / Lyra 侧行号（`UIExtensionSystem.cpp:97/133/143/179/184/210/237`、`UIExtensionPointWidget.cpp:90/151`、`LoadingScreenManager.cpp:99`、`LyraUIManagerSubsystem.cpp:40/60`、`LyraActivatableWidget.cpp:34-49`、`SActorCanvas.cpp:562`、`CommonUIExtensions.cpp:55/76/96/134`、`AddWidget.h:79-83`）抽查全部正确。

### 50-Settings-Online-Platform.md（19 处）

| 行 | 旧 | 新 |
|---|---|---|
| 31 | LyraSettingsShared.h:72 | :73 |
| 32 | LyraGameSettingRegistry `.cpp:103` | `.cpp:24(Get)` |
| 32 | `OnInitialize`(.cpp:137) | (.cpp:54) |
| 32 | `SaveChanges`(158) | (75) |
| 43 | LyraSettingsListEntrySetting_KeyboardInput.h:341 | :22 |
| 45 | LyraSettingScreen.h:89 | :15 |
| 54 | Bool OFF/ON (cpp:218-221) | (cpp:193-194) |
| 55 | 缺 DisplayFormat 会 ensure(cpp:80) | ensureAlways(cpp:95) |
| 59 | `FGameSettingEditCondition`/`FGameSettingEditableState` h:159/83 | 补注「159 是 EditCondition，83 是 EditableState」 |
| 86 | LyraHotfixManager `.cpp:83` | `.cpp:15` |
| 87 | LyraRuntimeOptions.h:310 | :28 |
| 88 | LyraTextHotfixConfig.h:362 | :16 |
| 89 | LyraPerformanceSettings.h:163 | :68 |
| 90 | LyraPerformanceSettings.h:216 | :121 |
| 441 | Registry `OnInitialize` :137 | :54 |
| 452 | `IsFinishedInitializing` :119 | :36 |
| 485 / 505 | `SaveChanges` :158 | :75 |
| 610 | `GetAllMappedActionsFromKey` :287 | :224 |
| 634 | `StartHotfixProcess` :267 | :199 |
| 791 | 编辑器内跳过 :269 | :201-205 |

抽查正确的代表性条目：`LyraSettingsLocal.cpp` 341/353/374/423/452/1221/1314 全部命中；`LyraSettingsShared.cpp` 52/62/72、`:235` `GConfig->SetString("Internationalization","Culture",…, GGameUserSettingsIni)`；`GameSettingRegistry_*` 各 Initialize 入口（Video:140、MouseAndKeyboard:21、Gamepad:21、Gameplay:16、Audio:23、PerfStats:115、Video SettingChanged:127）；`CommonUserSubsystem.cpp:441 ShouldCreateSubsystem`、`:997 TryToLoginForOnlinePlay`；`CommonSessionSubsystem.cpp:254 ConstructTravelURL`；`LyraGameMode.cpp` 88/104/106/167/193/266/289；`LyraGameFeaturePolicy.cpp:72/76`；`LyraUserFacingExperienceDefinition.cpp:40`；`LyraLocalPlayer.cpp:102`；`DefaultEngine.ini:28`(GameUserSettingsClassName)、`:56-57`(HotfixManagerClassName)；`LyraPerformanceSettings.cpp:18/33`(桌面 30..360)；`LyraPerformanceStatSubsystem.cpp:28/158`；`LyraMemoryDebugCommands.cpp:51`。

### 60-Presentation.md（6 处）

| 行 | 旧 | 新 | 依据 |
|---|---|---|---|
| 53 | `LyraCharacterPartTypes.h:91` | :56 | 文件仅 77 行；`FLyraCharacterPart` 在 :56 |
| 54 | `LyraCharacterPartTypes.h:66` | :31 | `FLyraCharacterPartHandle` 在 :31 |
| 163 | `LyraAbilitySystemComponent.cpp:78` | :76 | AnimInstance `InitializeWithAbilitySystem` 分支起点 :76 |
| 391 / 434 | "原版/非原版"措辞 | 见争议项 (a) | — |
| 637 | `SetReplicatedAcceleration (.cpp:101)`、`SimulateMovement …(.cpp:27)` | 均标注为 `LyraCharacterMovementComponent.cpp:101` / `:29` | 上下文文件是 LyraCharacter.cpp，裸 `.cpp` 易误读；:29 才是 `bHasReplicatedAcceleration` 分支 |
| 723 | `ApplyDynamicRangeEffectsChains(bool)` (`.cpp:267`) | (`LyraAudioMixEffectsSubsystem.cpp:267`) | 同行前半是 LyraSettingsLocal.cpp:974，裸 `.cpp:267` 会被读成 LyraSettingsLocal |

抽查正确的代表性条目：`LyraCameraMode.cpp` 82/133/177/229/264/318/321/365/407/449/459、`LyraCameraMode_ThirdPerson.cpp:111`、`LyraCameraComponent.cpp:51-52/120`、`LyraCameraComponent.h:47`；`LyraRangedWeaponInstance.cpp:195-207/203`（ADS tag 比对确在此）；`LyraPawnComponent_CharacterParts.h:62/78`、`.cpp:143/157/319`（:157 `!IsNetMode(NM_DedicatedServer)` → 专用服务器不生成部件 Actor，论断成立）；`LyraNumberPopComponent_MeshText.cpp:49-55`（仅本地 PlayerController 显示，注释明说避免 listen server 主机重复）；GameplayMessage 广播点 `LyraHealthSet.cpp:144`、`LyraHealthComponent.cpp:171-181`、`LyraGameState.cpp:107`、`LyraGameplayAbility.cpp:117`、`LyraGameplayAbility_Reset.cpp:52`、`LyraInventoryManagerComponent.cpp:77`；`LyraSettingsLocal.cpp:974-976`；`LyraUserFacingExperienceDefinition.h:65 / .cpp:45`(bRecordReplay→DemoRec)；`DefaultGame.ini:219-229`(音频总线/HDR-LDR 效果链/LoadingScreenControlBusMix) 全部命中。

### 70-GameFeatures-Content.md（2 处）

| 行 | 旧 | 新 | 依据 |
|---|---|---|---|
| 73 | 「`AimAssistCollisionChannel`（默认 `ECC_EngineTraceChannel5`），`DefaultGame.ini` 中有 `[…ShooterCoreRuntimeSettings]` 段」 | 「头文件默认 `ECC_EngineTraceChannel5`；`DefaultGame.ini:216-217` 的该段**覆盖**为 `ECC_GameTraceChannel5`」 | `ShooterCoreRuntimeSettings.h:30` 默认值与 ini 实际值不同，原文容易误导 |
| 397 | 「这是整个项目里**唯一一处**非 Epic 官方的 C++ 代码」 | 「这是一个极小的自定义编辑器工具模块(文件头仍是 Epic 模板版权声明，不能据此判定是否 Epic 官方，原文"唯一一处非 Epic 官方"无法证实)」 | `BPFunctionLibrary.cpp/.h` 头部均为 "Copyright Epic Games, Inc."；无证据支持"非官方"且"唯一" |

核实正确的条目（18 抽查中 16 正确）：5 个 GFP 的 `.uplugin` 三连（`ExplicitlyLoaded:true`、`EnabledByDefault:false`、`BuiltInInitialFeatureState:"Registered"`）全部一致；ShooterCore 依赖列表与文档完全吻合（GameplayAbilities / ModularGameplay / GameplayMessageRouter / AsyncMixin / CommonUI / CommonGame / GameSubtitles / EnhancedInput / LyraExampleContent）；ShooterTests 的 `TargetConfigurationDenyList: ["Shipping"]` 与依赖（ShooterCore / AsyncMessageSystem(+Tests) / CQTest / CQTestEnhancedInput）正确；ShooterExplorer、ShooterMaps 均依赖 ShooterCore+LyraExampleContent 且无 Modules；TopDownArena 依赖 GameplayAbilities / LyraExampleContent / Niagara 且不依赖 ShooterCore；GreenRoom/RedRoom 的 `Category:Other`+`ExplicitlyLoaded:true`+`EnabledByDefault:false`+无 Modules 正确；LyraExtTool 无 `EnabledByDefault` 键、`Modules:[{LyraExtTool, Editor, Default}]`、Build.cs 公共 `Core` / 私有 `CoreUObject,Engine,Slate,SlateCore`、`FLyraExtToolModule` 空实现、`ChangeMeshMaterials` 的 `Modify()`→替换全部材质槽→`PostEditChange()`→恒返回 true 且无 null 检查 —— 全部与源码一致；`uproject` 里 5 个 GFP `Enabled:true`、GreenRoom/RedRoom/LyraExtTool 确实不在列表；`DefaultEditor.ini` 本地化 `ExcludePathWildcards` 含 `Plugins/LyraExtTool/*` 正确；AssetManager 的 `SpecificAssets`（L_LyraFrontEnd + L_DefaultEditorOverview；B_LyraFrontEnd_Experience）与 `DefaultGame.ini:49 GameFeaturesManagerClassName` 正确。

未能核实（已在原文第 8 节标注为推断，保留）：`EUW_MaterialTool` 蓝图图表内容、各 Experience 资产内部字段（.uasset 二进制，未用 MCP）。

### 80-Build-Config.md（4 处）

| 位置 | 旧 | 新 | 依据 |
|---|---|---|---|
| 234 节 `[GameMapsSettings]` | 列在 "### DefaultGame.ini" 之下 | 加注「实际位于 `Config/DefaultEngine.ini:51-54`，不在 DefaultGame.ini」 | `GlobalDefaultGameMode/GameInstanceClass/GameDefaultMap/EditorStartupMap` 全在 DefaultEngine.ini |
| 258 | `PrimaryAssetTypesToScan` **（6项）** | 「(8 项 `+` 条目，另有 2 条 `-` 移除引擎默认项)」 | DefaultGame.ini:56-65：2 条 `-`、8 条 `+`（Map / LyraGameData / PrimaryAssetLabel / GameFeatureData / LyraExperienceDefinition / LyraUserFacingExperienceDefinition / LyraLobbyBackground / LyraExperienceActionSet） |
| 628 ReplicationGraph「启用以优化网络复制性能」 | — | 「模块已依赖，但 `bDisableReplicationGraph` 默认 `true`（LyraReplicationGraphSettings.h:24；LyraReplicationGraph.cpp:145 据此跳过），默认并不启用；当前 DefaultGame.ini:232 也显式写了 `=True`」 | 原文暗示默认启用，与代码/配置相反 |
| 640 | 「16种语言已配置」 | 「14种语言（`CulturesToStage`: en, ar, es, es-419, fr, it, ja, pl, pt-BR, ru, tr, zh-Hans, ko, de）」 | DefaultGame.ini:136-149 共 14 个 `+CulturesToStage` |

核实正确的条目：`LyraGame.Build.cs` 公共/私有依赖列表、`SHIPPING_DRAW_DEBUG_ERROR=1`、Shipping 下 `WITH_RPC_REGISTRY=0`/`WITH_HTTPSERVER_LISTENERS=0`/`WITH_AUTOMATION_DRIVER=0`、非 Shipping 且 `!bIsEngineInstalled` 才加 `AutomationDriver`、`SetupIrisSupport`；`ApplySharedLyraTargetSettings`（`BuildSettingsVersion.V7`、`IncludeOrderVersion.Latest`、Unique 下 `ShadowVariableWarningLevel=Error` + `bUseLoggingInShipping=true`）；各 Target 的 `Type`/`CustomConfig`（EOS/Steam/SteamEOS）、EOS 与 SteamEOS 才 `EnablePlugins(OnlineServicesEOS, OnlineSubsystemEOS)`、LyraGameSteam 无额外插件、`LyraServer` 的 `[SupportedPlatforms(Server)]`、`LyraEditor` 的 `EnablePlugins.Add("RemoteSession")`；`[OnlineServices] DefaultServices=Null`（DefaultEngine.ini:73-74）；`GameNetworkManager` 的 200000/40000/20000；打包设置 `PPBC_Development`/`BuildTarget=LyraGame`/`UsePakFile=True`/`bGenerateChunks=true`/`Oodle`/`Kraken`/`InternationalizationPreset=All`；`DefaultEngine.ini` 的 Engine 类覆盖（GameEngine / UnrealEdEngine / EditorEngine / GameViewportClientClassName / AssetManagerClassName / WorldSettingsClassName / LocalPlayerClassName / GameUserSettingsClassName）；`DefaultInput.ini:32-40`；`DefaultGame.ini` 的 ASG / GameplayCueNotifyPaths / `ActivateFailCooldownTag` / UIPolicy / LoadingScreenWidget / CommonUI 材质与 RichTextData。

---

## 3. 未能核实的事项

1. 一切 `.uasset`/`.umap` 内部字段（Experience 的 Actions 列表、PawnData 字段、`ID_Rifle` 的 StatTags 初值、`B_NiagaraNumberPopComponent` 的 Style 指向、`W_*` 控件树、DataTable 行等）：按任务要求未使用 unreal-mcp，二进制资产无法可靠读取。30/60/70 中涉及资产内部值的论断（如 MagazineSize=30/SpareAmmo=60、`B_TeamSpawningRules` 挂载、`DA_Frontend` 的 MaxPlayerCount=1）仅在其 C++/ini 侧可核的部分得到印证。
2. 10:36「`LyraEditorEngine` 强制 Standalone」等 LyraEditor 模块论断只核对了行号（`FirstTickSetup` 在 :42），具体 PIE 行为未逐句验证。
3. 70 第 8 节自述「各插件 Config/Tags/*.ini 只读取了前 60 行」的部分未补全核对；ShooterCore 仅有一个 `Config/Tags/ShooterCoreTags.ini`（其中 `Lyra.ShooterGame.Weapon.MagazineAmmo/MagazineSize/SpareAmmo` 的 tag 定义本身未逐行读到此文，保留其"待核"状态）。
4. 80 中与打包产物/`Binaries` 时效相关的描述（仅核对了 `Plugins/LyraExtTool/Binaries/Win64/UnrealEditor-LyraExtTool.dll` 确实存在）。
5. "是否为 Epic 原版"类判断（60 争议项 a 之外）无上游 Lyra 仓库可对比，只能依据代码风格/版权头与目录归属判断，该局限已在 70:397 处注明。

---

## 4. 方法与可重复性

- 用脚本从每篇文档抽出所有 `文件名.{cpp,h}:行号`（含 `.cpp:NNN` 的省略写法，按同行上文的文件名解析），在 `Source/`、`Plugins/`、`Config/` 下定位文件，取 `行号±6` 的窗口与同行反引号内的符号名做匹配，不匹配者进入人工复核；再对每篇文档的第 3/7/8 节与 Q&A 逐条人工打开源码确认语义（尤其 Server/Client 分支与"坑"类论断）。
- 自动比对命中率（未命中者含大量"符号名在窗口外但行号正确"的误报，已人工复核剔除）：10 → 228/254，20 → 231/276，30 → 227/280，40 → 148/208，50 → 157/209，60 → 223/283，70 → 16/18，80 → 12/16。
- 本报告只修改 `Docs/Lyra/*.md`，未触碰任何源码、配置或资产。
