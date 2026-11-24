# 70 - GameFeature 插件与内容层（ShooterCore / TopDownArena / ShooterExplorer / ShooterMaps / ShooterTests + 自定义插件）

> 范围：`Plugins/GameFeatures/*`、`/Game` 下的 Experience/PawnData/Input 核心内容、自定义插件 GreenRoom / RedRoom / LyraExtTool。
> 数据来源：磁盘（.uplugin、Config、C++）+ Unreal MCP 只读检查（编辑器已打开，UE 5.8）。MCP 无法读取的部分已在第 8 节"未验证项"中标注。
> 路径约定：`/ShooterCore/...` 即 `Plugins/GameFeatures/ShooterCore/Content/...`；`/Game/...` 即 `Content/...`。

---

## 1. 职责概述

Lyra 把"游戏模式"全部做成数据驱动：

- **Experience (`ULyraExperienceDefinition`, 蓝图子类)** = 一个玩法的完整描述：要启用的 GameFeature 插件、默认 PawnData、一串 `GameFeatureAction`、若干 `ULyraExperienceActionSet`。
- **GameFeature 插件** = 内容/代码的承载包。`ShooterCore` 提供射击玩法共用的一切；`ShooterMaps` 只放地图和 UserFacingExperience（playlist）；`TopDownArena` 是完全独立的第二个游戏（炸弹人）；`ShooterExplorer` 是在 ShooterCore 上加"探索/背包"测试；`ShooterTests` 是自动化测试。
- **UserFacingExperience (`ULyraUserFacingExperienceDefinition`)** = 前端菜单里的"一张牌"：Map + ExperienceID + 标题/图标/最大人数。
- **核心代码 (`LyraGame` 模块)** 只提供通用类（GameState/PlayerState/Pawn/Ability/Input/Team/Bot 组件）；所有"模式差异"都是通过 Experience 里的 `GameFeatureAction_AddComponents / AddAbilities / AddWidgets / AddInputBinding / AddInputContextMapping` 在运行时注入到核心 Actor 上。

启动链路（概览）：`GameDefaultMap = /Game/System/FrontEnd/Maps/L_LyraFrontEnd` -> `B_LyraGameMode` 在 `InitGame` 解析 Experience（优先级高->低，LyraGameMode.cpp:90-165：匹配分配 > URL `?Experience=` > PIE 的 `ULyraDeveloperSettings.ExperienceOverride` > 命令行 `-Experience=` > World Settings 的 DefaultGameplayExperience > 专用服务器登录 > 默认 `B_LyraDefaultExperience`；指定的 ID 找不到时回退默认）-> `ULyraExperienceManagerComponent` 加载 `GameFeaturesToEnable` 对应插件并执行 Actions -> `OnExperienceLoaded` -> 生成 Pawn、机器人、Team 等。

---

## 2. 关键类/资产表

### 2.1 插件总表

| 名称 | 路径 | 职责 / 状态 |
|---|---|---|
| ShooterCore | `Plugins/GameFeatures/ShooterCore` | 射击玩法核心（带 C++ 模块 `ShooterCoreRuntime`）；`BuiltInInitialFeatureState=Registered`，`ExplicitlyLoaded=true`，`EnabledByDefault=false`。依赖 GameplayAbilities / ModularGameplay / GameplayMessageRouter / AsyncMixin / CommonUI / CommonGame / GameSubtitles / EnhancedInput / LyraExampleContent |
| ShooterMaps | `Plugins/GameFeatures/ShooterMaps` | 纯内容：地图 L_Expanse / L_Convolution_Blockout / L_FiringRange_WP / L_ShooterFrontendBackground + 两个 playlist；依赖 ShooterCore、LyraExampleContent；Registered |
| TopDownArena | `Plugins/GameFeatures/TopDownArena` | 第二个游戏"Exploder"（炸弹人风格），带 C++ 模块 `TopDownArenaRuntime`；不依赖 ShooterCore，依赖 GameplayAbilities / LyraExampleContent / Niagara；Registered |
| ShooterExplorer | `Plugins/GameFeatures/ShooterExplorer` | 在 ShooterCore 上叠加：交互/背包/地图 UI/StateTree 跟随 AI 测试；无 C++；依赖 ShooterCore、LyraExampleContent；Registered |
| ShooterTests | `Plugins/GameFeatures/ShooterTests` | 自动化测试（CQTest + Functional Test 蓝图），C++ 模块 `ShooterTestsRuntime`（`TargetConfigurationDenyList: Shipping`）；依赖 ShooterCore、AsyncMessageSystem(+Tests)、CQTest、CQTestEnhancedInput 等；Registered |
| GreenRoom / RedRoom | `Plugins/GreenRoom`, `Plugins/RedRoom` | 非 GameFeature 的极简内容插件（见第 7 节） |
| LyraExtTool | `Plugins/LyraExtTool` | 自定义 Editor 模块 + 一个 Editor Utility Widget（见第 7 节） |

运行时状态（MCP `GetGameFeatureState`）：`ShooterCore / ShooterExplorer / ShooterMaps / ShooterTests / TopDownArena` 当前全部为 `Registered`（尚未 Active，只有进入某个 Experience 才会被 `GameFeaturesToEnable` 激活）。`LyraStarterGame.uproject` 里这 5 个插件均 `Enabled: true`；**GreenRoom / RedRoom / LyraExtTool 没有出现在 uproject 的 Plugins 列表**（LyraExtTool 因位于项目 Plugins 目录且无 `EnabledByDefault:false` 而被自动启用；GreenRoom/RedRoom 带 `ExplicitlyLoaded:true / EnabledByDefault:false`，MCP `find_assets("/GreenRoom/")` 返回空，说明当前未挂载）。

### 2.2 每个插件的 GameFeatureData 注册了什么（MCP 读取）

| 插件 | GFD 资产 | Actions | PrimaryAssetTypesToScan |
|---|---|---|---|
| ShooterCore | `/ShooterCore/ShooterCore` | `AddComponents`（见下）、`DataRegistry`（`/ShooterCore/Accolades/AccoladeDataRegistry`）、`AddGameplayCuePath`（`/GameplayCues` `/Weapons` `/Items`）、`AddInputContextMapping`（`/Game/Input/Mappings/IMC_Default` 与 `/ShooterCore/Input/Mappings/IMC_ShooterGame`，均 `bRegisterWithSettings`） | `LyraExperienceDefinition`(目录 `Experiences`、`System/Experiences`，蓝图类)；`LyraUserFacingExperienceDefinition`(`System/Playlists`, AlwaysCook)；`LyraExperienceActionSet`(`Experiences`、`/System/Experiences`、`Accolades`)；`Map`(`/Maps`)；`PlayerMappableInputConfig`(`/ShooterCore/Input/Configs`、`/Game/Input/Configs`) |
| ShooterMaps | `/ShooterMaps/ShooterMaps` | `AddGameplayCuePath`(`/GameplayCues`) | `Map`(`/Maps`)、`LyraLobbyBackground`(`Items/Backgrounds`)、`LyraUserFacingExperienceDefinition`(`System/Playlists`)、`PrimaryAssetLabel`(`ShooterMaps_Label`) |
| TopDownArena | `/TopDownArena/TopDownArena` | `AddGameplayCuePath`(`/GameplayCues`)、`AddInputContextMapping`(`/Game/Input/Mappings/IMC_Default`) | `LyraExperienceDefinition`(`System/Experiences`)、`Map`(`/Maps`, AlwaysCook)、`PlayerMappableInputConfig`、`LyraUserFacingExperienceDefinition`(`System/Playlists`)、`PrimaryAssetLabel`(`TopDownArena_Label`) |
| ShooterExplorer | `/ShooterExplorer/ShooterExplorer` | 无 Action | `Map`、`LyraExperienceDefinition`(`System/Experiences`)、`LyraExperienceActionSet`(`System/Experiences`) |
| ShooterTests | `/ShooterTests/ShooterTests` | 无 Action | `Map`、`LyraExperienceDefinition`(`System/Experiences`)、`LyraUserFacingExperienceDefinition`(`System/Playlists`) |

**ShooterCore GFD 的 `AddComponents`（只要插件 Active 就生效，与具体 Experience 无关）**：

| Actor 类 | 组件 | Client/Server |
|---|---|---|
| `AGameStateBase` | `B_EliminationFeedRelay` | C+S |
| `ALyraCharacter` | `ULyraEquipmentManagerComponent` | C+S |
| `ALyraPlayerController` | `ULyraIndicatorManagerComponent` | 仅 Client |
| `ALyraPlayerController` | `B_HandleShooterReplays` | 仅 Client |
| `AController` | `ULyraInventoryManagerComponent` | C+S |
| `AController` | `ULyraWeaponStateComponent` | C+S |
| `ALyraGameState` | `B_AimAssistTargetManager` | 仅 Client |

> 即：装备/背包/武器状态机这些"射击专用"组件不在核心代码里写死，而是 ShooterCore 一激活就挂到核心 Actor 上。TopDownArena 里没有这些，所以它没有武器/背包。

### 2.3 ShooterCore 关键 C++（`Source/ShooterCoreRuntime`）

| 类 | 文件 | 职责 |
|---|---|---|
| `UTDM_PlayerSpawningManagmentComponent`（注意拼写 Managment） | `Private/TDM_PlayerSpawningManagmentComponent.{h,cpp}` | 继承 `ULyraPlayerSpawningManagerComponent`；重写 `OnChoosePlayerStart`：遍历所有非旁观、非同队玩家，选"离敌人 Pawn 最远且未被占用(Occupancy<Full)的 `ALyraPlayerStart`"，若全被 Claim 则退而求其次选最远的已 Claim 点；`OnFinishRestartPlayer` 空实现。蓝图子类 `B_TeamSpawningRules` |
| `UAssistProcessor` | `Public/MessageProcessors/AssistProcessor.h` | `UGameplayMessageProcessor`；监听 `Lyra.Damage.Message` 累计每个受害者被各玩家造成的伤害，监听 `Lyra.Elimination.Message` 时给除击杀者外造过伤害的人广播 `Lyra.Assist.Message`（Magnitude=累计伤害） |
| `UElimChainProcessor`（Abstract） | 同上目录 | 连杀(chain)：`ChainTimeLimit=4.5s` 内连续击杀，按 `ElimChainTags: TMap<int32,FGameplayTag>` 广播 2x/3x/4x/5x 勋章；蓝图子类 `B_ElimChainProcessor` |
| `UElimStreakProcessor`（Abstract） | 同上 | 不死连杀(streak)：按 `ElimStreakTags` 广播 5/10/15/20 勋章；蓝图子类 `B_ElimStreakProcessor` |
| `ULyraAccoladeHostWidget` | `Public/Accolades/LyraAccoladeHostWidget.h` | HUD 勋章宿主 Widget：监听 `FLyraNotificationMessage`，用 `FAsyncMixin` 从 DataRegistry 异步加载 `FLyraAccoladeDefinitionRow`（DisplayName/Sound/Icon/DisplayDuration/LocationTag/AccoladeTags/CancelAccoladesWithTag），排队展示。`LocationName` Tag 过滤。BP 事件 `CreateAccoladeWidget/DestroyAccoladeWidget` |
| `ULyraAccoladeDefinition` | `Public/Accolades/LyraAccoladeDefinition.h` | 旧式勋章 DataAsset（现行流程用 DataTable 行 `FLyraAccoladeDefinitionRow` + DataRegistry） |
| `UAimAssistInputModifier` / `UAimAssistTargetComponent` / `UAimAssistTargetManagerComponent` / `IAimAssistTargetInterface` | `Public/Input/` | 手柄辅助瞄准：InputModifier 修改 Look 输入；TargetComponent 挂在可被吸附目标上；ManagerComponent（`B_AimAssistTargetManager`，仅客户端挂到 GameState）收集候选目标 |
| `UShooterCoreRuntimeSettings` | `Public/ShooterCoreRuntimeSettings.h` | `UDeveloperSettings`（config=Game, defaultconfig）；`AimAssistCollisionChannel`（头文件默认 `ECC_EngineTraceChannel5`；`DefaultGame.ini:216-217` 的 `[/Script/ShooterCoreRuntime.ShooterCoreRuntimeSettings]` 段覆盖为 `ECC_GameTraceChannel5`） |
| `ALyraWorldCollectable` | `Public/LyraWorldCollectable.h` | Abstract Blueprintable Actor，实现 `IInteractableTarget` + `IPickupable`；`Option`(交互选项) + `StaticInventory`(`FInventoryPickup`) —— 世界里可交互拾取的物品（ShooterExplorer 的 B_InteractableRock/Chair 等使用） |
| `FLyraControlPointStatusMessage` | `Public/Messages/ControlPointStatusMessage.h` | 占点状态消息结构 `{AActor* ControlPoint; int32 OwnerTeamID}`，用于 `ShooterGame.ControlPoint.Captured.Message` 等通道 |

模块依赖（`ShooterCoreRuntime.Build.cs`）：LyraGame、ModularGameplay、CommonGame、GameplayTags/Tasks/Abilities、GameplayMessageRuntime、CommonUI、UMG、DataRegistry、AsyncMixin、EnhancedInput、GameSubtitles、DeveloperSettings、AIModule。

### 2.4 TopDownArena 关键 C++（`Source/TopDownArenaRuntime`）

| 类 | 职责 |
|---|---|
| `UTopDownArenaAttributeSet : ULyraAttributeSet` | 4 个复制属性：`BombsRemaining`、`BombCapacity`、`BombRange`、`MovementSpeed`（含 Clamp、OnRep） |
| `UTopDownArenaMovementComponent : ULyraCharacterMovementComponent` | 重写 `GetMaxSpeed()`，让移动速度读 `MovementSpeed` 属性（道具加速生效的原因） |
| `ULyraCameraMode_TopDownArenaCamera : ULyraCameraMode` | 自适应整个竞技场的俯视相机：`ArenaWidth/Height/DefaultPivotRotation/BoundsSizeToDistance`；BP 子类 `CM_ArenaFramingCamera` |
| `UTopDownArenaPickupUIData : UGameplayEffectUIData` | 道具 GE 的 UI 元数据（描述/Toast 文案/图标/拾取 VFX/SFX） |

### 2.5 ShooterTests C++（`Source/ShooterTestsRuntime`）

见第 3.6 节。

### 2.6 核心 /Game 内容

| 名称 | 路径 | 职责 |
|---|---|---|
| DefaultGameData | `/Game/DefaultGameData` | `ULyraGameData`：`DamageGameplayEffect_SetByCaller=GE_Damage_Basic_SetByCaller`、`HealGameplayEffect_SetByCaller=GE_Heal_SetByCaller`、`DynamicTagGameplayEffect=GE_DynamicTag`；由 `[/Script/LyraGame.LyraAssetManager] LyraGameDataPath` 指定 |
| EmptyPawnData | `/Game/Characters/Heroes/EmptyPawnData/DefaultPawnData_EmptyPawn` | `ULyraAssetManager.DefaultPawnData`：所有字段为空（占位，防止无 PawnData 时崩溃） |
| SimplePawnData | `/Game/Characters/Heroes/SimplePawnData/SimplePawnData` | PawnClass=`B_SimpleHeroPawn`，InputConfig=`InputData_SimplePawn`，CameraMode=`CM_ThirdPerson`，无 AbilitySet |
| B_LyraGameMode | `/Game/B_LyraGameMode` | `GlobalDefaultGameMode`；GameState=`ALyraGameState`、PlayerController=`ALyraPlayerController`、PlayerState=`ALyraPlayerState`、HUD=`ALyraHUD`、DefaultPawn=`ALyraCharacter`、GameSession=`ALyraGameSession`、ReplaySpectatorPC=`ALyraReplayPlayerController`（Pawn 由 PawnData 覆盖） |
| B_LyraDefaultExperience | `/Game/System/Experiences/B_LyraDefaultExperience` | 兜底 Experience（见 3.5） |
| B_LyraFrontEnd_Experience | `/Game/System/FrontEnd/B_LyraFrontEnd_Experience` | 前端 Experience（见 3.4） |
| DA_Frontend / DA_ExamplePlaylist | `/Game/System/Playlists/` | 前端、默认 Experience 的 UserFacingExperience |
| TeamDA_Red/Blue/Green/Yellow | `/Game/System/Teams/` | `ULyraTeamDefinition`；`B_TeamSetup_TwoTeams` 用 Red(1)/Blue(2) |
| InputData_Hero | `/Game/Input/InputData_Hero` | `ULyraInputConfig`：Native：Move/Look.Mouse/Look.Stick/Crouch/AutoRun；Ability：Jump/Weapon.Reload/Ability.Heal/Ability.Dash/Weapon.Fire/Weapon.FireAuto |
| IMC_Default | `/Game/Input/Mappings/IMC_Default` | 通用输入映射（ShooterCore、TopDownArena 的 GFD 都会注册） |
| B_PickRandomCharacter / B_CharacterSelection | `/Game/Characters/Cosmetics/` | `ULyraControllerComponent_CharacterParts` 子类：随机 / 轮询(Manny,Quinn) 选角色外观；Experience 把它挂到 `Controller` 上（仅服务器） |
| HeroData_ShooterGame | `/ShooterCore/Game/HeroData_ShooterGame` | ShooterCore 英雄 PawnData（见 3.1） |

---

## 3. 每个 Experience 的完整拆解

### 3.0 通用共享 ActionSet（ShooterCore，`/ShooterCore/Experiences/`）

| ActionSet | 内容 |
|---|---|
| `LAS_ShooterGame_SharedInput` | `AddInputBinding`：`InputData_ShooterGame_AddOns`（ShooterCore 增加的输入：ADS/Drop/Emote/Grenade/Melee/QuickSlot1-3/Cycle/ShowScoreboard 等，见 `/ShooterCore/Input/Actions/`）；`AddInputContextMapping`：`IMC_ShooterGame`（bRegisterWithSettings） |
| `LAS_ShooterGame_StandardComponents` | `AddComponents`：PlayerController<-`B_NiagaraNumberPopComponent`(仅 Client，伤害数字)；Controller<-`B_QuickBarComponent`(C+S，快捷栏)；PlayerController<-`NameplateManagerComponent`(Client)；`B_Hero_ShooterMannequin`<-`NameplateSource`(Client) |
| `LAS_ShooterGame_StandardHUD` | `AddWidgets`：Layout `W_ShooterHUDLayout` -> 层 `UI.Layer.Game`；Widgets 挂槽：`HUD.Slot.EliminationFeed`<-`W_EliminationFeed`；`HUD.Slot.Equipment`<-`W_QuickBar`；`HUD.Slot.TopAccolades`<-`W_AccoladeHostWidget`；`HUD.Slot.Reticle`<-`W_WeaponReticleHost`；`HUD.Slot.PerfStats.Graph/Text`<-`W_PerfStatContainer_GraphOnly/TextOnly`；触屏：`HUD.Slot.LeftSideTouchInputs`<-`W_OnScreenJoystick_Left`，`RightSideTouchInputs`<-`W_OnScreenJoystick_Right`+`W_FireButton`，`RightSideTouchRegion`<-`W_TouchRegion_Right`，`LeftSideTouchRegion`<-`W_TouchRegion_Left` |
| `EAS_BasicShooterAcolades`（`/ShooterCore/Accolades/`） | `AddComponents` 到 `AGameStateBase`：`B_ElimChainProcessor`(仅服务器)、`B_ElimStreakProcessor`(仅服务器)、`UAssistProcessor`(C++，仅服务器)、`B_AccoladeRelay`(C+S)。勋章数据来自 DataRegistry `AccoladeDataRegistry` + `DT_BasicShooterAccolades` |

> 以上 4 个 ActionSet 的 `gameFeaturesToEnable` 为 `ShooterCore`（LAS_* 三个）。

### 3.1 ShooterCore 英雄（PawnData）

`/ShooterCore/Game/HeroData_ShooterGame`（`ULyraPawnData`）：

| 字段 | 值 |
|---|---|
| PawnClass | `B_Hero_ShooterMannequin`（父类 `/Game/Characters/Heroes/B_Hero_Default` <- `Character_Default` <- `ALyraCharacter`） |
| AbilitySets | `AbilitySet_ShooterHero` |
| TagRelationshipMapping | `TagRelationships_ShooterHero` |
| InputConfig | `/Game/Input/InputData_Hero` |
| DefaultCameraMode | `/Game/Characters/Cameras/CM_ThirdPerson`（ShooterCore 另有 `CM_ThirdPersonADS`） |

`AbilitySet_ShooterHero` 授予（技能 / 输入 Tag）：`GA_Hero_Jump`(InputTag.Jump)、`GA_Hero_Death`(无输入)、`GA_Hero_Dash`(InputTag.Ability.Dash)、`GA_Emote`(InputTag.Ability.Emote)、`GA_QuickbarSlots`(InputTag.Ability.Quickslot)、`GA_ADS`(InputTag.Weapon.ADS)、`GA_Grenade`(InputTag.Weapon.Grenade)、`GA_DropWeapon`(InputTag.Ability.Quickslot.Drop)、`GA_Melee`(InputTag.Ability.Melee)、`GA_SpawnEffect`(无输入)、`LyraGameplayAbility_Reset`(C++，无输入)；GE：`GE_IsPlayer`。

`TagRelationships_ShooterHero`：`Ability.Type.Action` 在 `Status.Death.Dead/Dying` 下被阻止；WeaponFire/Melee/Drop/Grenade 激活时 Block+Cancel Emote/Reload；Dash 互斥并取消所有 `Ability.Type.Action`；Reload 阻止 Emote；Emote 在 `Movement.Mode.Falling` 下被阻止。

**初始装备（起手武器）**：写在 Pawn 蓝图 `B_Hero_ShooterMannequin` 的 CDO 变量里：`weaponId = WID_Pistol_C`，`initialInventoryItems = [ID_Pistol_C]`；`AddInitialInventory` / `WaitForInventoryReadiness` 函数（蓝图）在背包就绪后把 `ID_Pistol` 加进 `ULyraInventoryManagerComponent` 并通过 QuickBar 装备。即所有 Shooter 模式（Elimination/Control/Perf/Tests）起手都是手枪；地图上的 `B_WeaponSpawner`（依赖 `WeaponPickupData_*`）发放步枪/霰弹枪。

**武器资产链**（以步枪为例，手枪/霰弹枪完全同构，在 `/ShooterCore/Weapons/<Name>/`）：

```
ID_Rifle (InventoryItemDefinition: EquippableItem / QuickBarIcon / SetStats / PickupIcon / ReticleConfig 五个 Fragment)
  └─ WID_Rifle (EquipmentDefinition): InstanceType=B_WeaponInstance_Rifle,
        AbilitySetsToGrant=[AbilitySet_ShooterRifle],
        ActorsToSpawn=[B_Rifle @ socket weapon_r]
        └─ AbilitySet_ShooterRifle: GA_Weapon_Fire_Rifle_Auto(InputTag.Weapon.FireAuto) +
                                   GA_Weapon_Reload_Rifle(InputTag.Weapon.Reload) + /Game/Weapons/GA_Weapon_AutoReload
WeaponPickupData_Rifle (LyraWeaponPickupDefinition): InventoryItemDefinition=ID_Rifle_C, spawnCoolDownSeconds=10, 拾取/重生音效
```

差异：手枪射击 Tag 为 `InputTag.Weapon.Fire`(GA_Weapon_Fire_Pistol)，步枪/霰弹枪为 `InputTag.Weapon.FireAuto`。另有 `NetShooter_PROTO`(网络射手原型)、`WID_InstantHeal`、`Items/HealthPickup_Unused`。

### 3.2 `B_ShooterGame_Elimination`（团队死斗 TDM）— `/ShooterCore/Experiences/`

被 `DA_Expanse_TDM`（`/ShooterMaps`，地图 `L_Expanse`，16 人，`bIsDefaultExperience=true`）使用。

| 项 | 内容 |
|---|---|
| GameFeaturesToEnable | `ShooterCore` |
| DefaultPawnData | `HeroData_ShooterGame` |
| ActionSets | `LAS_ShooterGame_SharedInput`、`LAS_ShooterGame_StandardComponents`、`LAS_ShooterGame_StandardHUD`、`EAS_BasicShooterAcolades` |
| Actions[0] AddAbilities | `ALyraPlayerState` <- `AbilitySet_Elimination`：`GA_ShowLeaderboard_TDM`(InputTag.Ability.ShowLeaderboard，按住显示 `W_MatchScoreBoard_Elimination`，基于 `GAB_ShowWidget_WhileInputHeld`) + `GA_AutoRespawn`(显示 `W_RespawnTimer`) |
| Actions[1] AddComponents | `ALyraGameState` <- `B_TeamDeathMatchScoring`(C+S，计分/胜负/阶段)、`B_MusicManagerComponent_Elimination`(仅 Client，音乐)、`B_ShooterBotSpawner`(仅 Server，机器人)、`B_TeamSetup_TwoTeams`(仅 Server，创建 Red/Blue 两队)、`B_TeamSpawningRules`(仅 Server，TDM 出生点规则)；`AController` <- `B_PickRandomCharacter`(仅 Server) |
| Actions[2] AddWidgets | `HUD.Slot.TeamScore` <- `W_ScoreWidget_Elimination` |

**TDM 的组成**：队伍(TwoTeams) + 计分(`B_TeamDeathMatchScoring` 继承 `B_ShooterGameScoring_Base`，后者是 `UGameStateComponent`，监听 GameplayMessage 的击杀/死亡，依赖 `B_WeaponSpawner`、`B_Grenade`) + 出生规则(C++ `UTDM_PlayerSpawningManagmentComponent`) + 机器人 + 阶段(Warmup -> Playing -> PostGame) + 记分板/复活/勋章 UI。`B_TeamDeathMatchScoring` CDO：`waitForPlayersDuration=10`、`countDownTime=15`；其资产依赖含 `Phase_Warmup`、`Phase_PostGame`、`GE_DamageImmunity_FromGameMode`（用于开局/终局免伤，`SetDamageEnabled`、`RestartAllPlayers`、`HandleVictory`、`OnEliminationScored`、`GetTeamScore` 为其蓝图函数）。目标分数 `TargetScore`（`OnRep_TargetScore`）的默认值 MCP 未读到（见 8）。

### 3.3 `B_LyraShooterGame_ControlPoints`（占点 Control）

被 `DA_Convolution_ControlPoint`（地图 `L_Convolution_Blockout`，16 人，默认 Experience）和 `DA_ShooterGame_ShooterGym`（`/ShooterCore`，`L_ShooterGym`，8 人）使用。

| 项 | 内容 |
|---|---|
| GameFeaturesToEnable | `ShooterCore` |
| DefaultPawnData | `HeroData_ShooterGame` |
| ActionSets | 与 Elimination 相同的 4 个 |
| Actions[0] AddWidgets | `HUD.Slot.TeamScore` <- `W_CPScoreWidget`；`HUD.Slot.ModeStatus` <- `W_ControlPointStatusWidget` |
| Actions[1] AddAbilities | `ALyraPlayerState` <- `AbilitySet_ControlPoint`：`GA_ShowLeaderboard_CP` + `GA_AutoRespawn` |
| Actions[2] AddComponents | `ALyraGameState` <- `B_ControlPointScoring`(C+S)、`B_MusicManagerComponent_ControlPoint`(Client)、`B_ShooterBotSpawner_ControlPoint`(Server)、`B_TeamSetup_TwoTeams`(Server)、`B_TeamSpawningRules`(Server)；`AController` <- `B_PickRandomCharacter`(Server) |

占点特有资产（`/ShooterCore/ControlPoint/`）：`B_ControlPointScoring`（`teamColors` 1红/2蓝/3绿/4粉，`delayBetweenScoring=1`；函数 `GetTeamScoreCP/GetPointStatus/HandleVictory`；依赖 Phase_Warmup/Phase_PostGame）、`/ShooterCore/Blueprint/B_ControlPointVolume`（地图中的占点体积）、UI（`W_ControlPointMarker`、`W_MatchScoreBoard_CP`、`W_SB_PlayerRow_CP`…）、AI（`BT_Lyra_Shooter_ControlPoint_Bot`、`B_AI_Controller_LyraShooter_ControlPoint`）、EQS `EQS_FindControlPoint` + `EQS_Context_ControlPoints`。

### 3.4 `B_ShooterGame_Perf`（性能测试）

被 `DA_ShooterGame_ShooterPerf`（`L_ShooterPerf`，`bShowInFrontEnd=true`，tile 标题 "Test"）使用。与 Elimination 几乎一致（同 AbilitySet_Elimination、同 TDM 计分/音乐、TwoTeams、TeamSpawningRules），区别：机器人组件换成 `B_ShooterBotSpawner_Perf`（`numBotsToCreate=20`、`numActorsToCreate=200`，控制器 `B_AI_Controller_LyraShooter_Passive`，会额外刷 200 个测试 Actor）；ActionSets 与 Elimination 完全相同（4 个，已核实）。

### 3.5 前端与默认 Experience（`/Game`）

**`B_LyraFrontEnd_Experience`**（`/Game/System/FrontEnd/`，`GameFeaturesToEnable` 为空，无 PawnData、无 ActionSet）：

| Action | 内容 |
|---|---|
| `GameFeatureAction_SplitscreenConfig` | `bDisableSplitscreen = true` |
| `GameFeatureAction_AddComponents` | `ALyraGameState` <- `B_LyraFrontendStateComponent`(仅 Client；依赖 `W_LyraStartup`、`W_LyraFrontEnd`，负责启动画面->登录->主菜单状态机)、`B_MusicManagerComponent_FE`(仅 Client) |
| `ApplyFrontendPerfSettingsAction`（CDO 内嵌 UObject，类型来自 LyraGame） | 前端性能设置 |
| `GameFeatureAction_AddWidgets` | `HUD.Slot.PerfStats.Text` <- `W_PerfStatContainer_FrontEnd` |

`DA_Frontend`：Map=`/Game/System/FrontEnd/Maps/L_LyraFrontEnd`，Experience=`B_LyraFrontEnd_Experience`，`MaxPlayerCount=1`。前端地图在 `Config/DefaultGame.ini` AssetManager 中被特别列入（`SpecificAssets`），并在 `DefaultEngine.ini` 设为 `GameDefaultMap`。UI 在 `/Game/UI/Menu/`（`W_LyraFrontEnd`、`Experiences/W_ExperienceList`、`W_ExperienceTile`、`W_ExperienceSelectionScreen`、`W_HostSessionScreen`、会话浏览等）。

**`B_LyraDefaultExperience`**（`/Game/System/Experiences/`）：DefaultPawnData=`SimplePawnData`；唯一 Action：`AddWidgets` Layout `W_DefaultHUDLayout` -> `UI.Layer.Game`。配合 `DA_ExamplePlaylist`（地图 `L_DefaultEditorOverview`）。当 World Settings / URL / 开发者设置都没给 Experience 时由 `ALyraGameMode::InitGame` 回退使用（LyraGameMode.cpp:160）。

### 3.6 TopDownArena Experiences（`/TopDownArena/System/Experiences/`）

| Experience | 说明 |
|---|---|
| `B_TopDownArenaExperience` | `GameFeaturesToEnable=[TopDownArena]`，PawnData=`HeroData_Arena`；Actions：`AddWidgets` Layout `W_TopDownArenaHUDLayout`->`UI.Layer.Game`；`SplitscreenConfig bDisableSplitscreen=true`；`AddComponents`：`ALyraPlayerState`<-`B_PlayerUIComponent`(Client)、`ALyraGameState`<-`B_TeamSetup_FourTeams`(Server)、`ALyraGameState`<-`B_TopDownArena_GameComponent_Lives`(C+S，生命制模式)、`AController`<-`B_PickRandomCharacter`(Server)。Playlist：`DA_TopDownArena_Splode`（地图 `L_TopDownArenaGym`，4 人，"Exploder / Checkered"） |
| `B_TopDownArena_Multiplayer_Experience` | 在上面基础上多一个 `ALyraGameState`<-`B_TopDownArena_GameComponent_SpawnLocalPlayers`(Server，生成本地多人玩家)；Playlist：`DA_TopDownArena_Multiplayer`（地图 `L_TopDown_LocalMultiplayer`，4 人） |

`HeroData_Arena`：PawnClass=`B_Hero_Arena`；AbilitySets=`AbilitySet_Arena`；InputConfig=`InputData_Arena`（Native：IA_Move；Ability：IA_Weapon_Fire -> `InputTag.Weapon.Fire`）；CameraMode=`CM_ArenaFramingCamera`；**无** TagRelationshipMapping。
`AbilitySet_Arena`：`GA_ArenaHero_Death`(无输入)、`GA_DropBomb`(`InputTag.Weapon.Fire`)；授予属性集 `UTopDownArenaAttributeSet`（C++）。
阶段：`/TopDownArena/Game/Modes/Phase_Warmup/Playing/PostGame`（同名但独立于 ShooterCore，另配 `GE_Warmup`）。组件基类 `B_TopDownArena_GameComponent_Base`。炸弹 `B_Bomb_Base/Standard`，GE `GE_Increment/DecrementBombsRemaining`、`GE_Damaged_By_Bomb`，道具 `B_TopDownArena_Pickup` + `GE_BombCountUp/BombRangeUp/MoveSpeedUp/AdditionalHeart`，`GE_Stat_*` 为统计 GE。地图生成：`B_LevelGeneration`/`B_BlockSpawning`，`B_DestructableBlock`。Tag：`TopDownArena.ExtensionPoint.{AbilityBar,Players,StatsBar}`、`GameplayCue.TopDownArenaGame.*`、`Ability.ActivateFail.CellAlreadyContainsBomb`。

### 3.7 ShooterExplorer（`/ShooterExplorer/System/Experiences/`）

`B_TestInventoryExperience`：`GameFeaturesToEnable=[ShooterCore]`（注意自己不在列表中，但其 Content/资产仍可被引用，因为 .uplugin 已启用且 Registered；其 Experience 资产在 GFD 的 `System/Experiences` 被扫描）；PawnData=`HeroData_Explorer`（Pawn=`B_Hero_Explorer`，复用 `AbilitySet_ShooterHero`+`TagRelationships_ShooterHero`，InputConfig=`InputData_InventoryTest`，相机 `CM_ThirdPerson`）；ActionSets：`LAS_ShooterGame_SharedInput`、`StandardComponents`、`StandardHUD`、`LAS_InventoryTest`；本体 Actions：AddWidgets(`W_CPScoreWidget`、`W_ControlPointStatusWidget` — 复制自占点)、AddAbilities(PlayerState<-`AbilitySet_InventoryTest`)、AddComponents(TwoTeams、TeamSpawningRules、PickRandomCharacter，均仅 Server)。
`LAS_InventoryTest`：`AddInputContextMapping`(`IMC_InventoryTest`，priority 1)、`AddInputBinding`(`InputData_InventoryTest`)、`AddAbilities`(`AbilitySet_InventoryTest`：`GA_Interact`/`GA_ToggleMap`/`GA_ToggleInventory`/`GA_ToggleMarkerInWorld`，对应 `InputTag.Ability.Interact/ToggleMap/ToggleInventory/ToggleMarkerInWorld`)。
内容：交互 `GA_Interaction_Collect/Sit`、`B_InteractableChair/Rock`、`TestID_Rock/Tree`、StateTree 跟随 AI(`L_STT_FollowPlayer`, `STT_*`)、DataLayer 测试、地图 `L_InteractionTestMap`/`L_InventoryTestMap`/`L_StateTree_FollowTest`。该插件没有 playlist（无 UserFacingExperience），只能通过地图 World Settings 或 `?Experience=B_TestInventoryExperience` 进入。

### 3.8 ShooterTests（`/ShooterTests/System/Experiences/`）

`B_BasicShooterTest` 与 `B_AutomatedShooterTest`：均为"TDM 精简版"（`GameFeaturesToEnable=[ShooterCore]`，HeroData_ShooterGame，AbilitySet_Elimination，TDM 计分+音乐，TwoTeams，TeamSpawningRules，ScoreWidget_Elimination，同样 4 个 ActionSet），**没有机器人**；区别：`B_AutomatedShooterTest` 把 `B_PickRandomCharacter` 换成 `B_CharacterSelection`（确定性轮询 Manny/Quinn，避免动画测试 flaky），`B_BasicShooterTest` 仍用随机角色。

### 3.9 ShooterMaps playlist 小结

| Playlist | Map | Experience | 人数 | 默认/前端 |
|---|---|---|---|---|
| `DA_Expanse_TDM` | `/ShooterMaps/Maps/L_Expanse` | `B_ShooterGame_Elimination` | 16 | Default + 前端显示；tile 图 `T_UI_MapTile_Expanse`；加载屏 `W_LoadingScreen_TDM` |
| `DA_Convolution_ControlPoint` | `/ShooterMaps/Maps/L_Convolution_Blockout` | `B_LyraShooterGame_ControlPoints` | 16 | Default + 前端；加载屏 `W_LoadingScreen_ControlPoint` |
| `DA_ShooterGame_ShooterGym`（ShooterCore） | `/ShooterCore/Maps/L_ShooterGym` | `B_LyraShooterGame_ControlPoints` | 8 | bShowInFrontEnd 默认 true |
| `DA_ShooterGame_ShooterPerf`（ShooterCore） | `/ShooterCore/Maps/L_ShooterPerf` | `B_ShooterGame_Perf` | 8 | 显示 |
| `DA_TopDownArena_Splode` / `_Multiplayer` | `L_TopDownArenaGym` / `L_TopDown_LocalMultiplayer` | 见 3.6 | 4 | 显示 |
| `DA_ExamplePlaylist`（Game） | `L_DefaultEditorOverview` | `B_LyraDefaultExperience` | 4 | 显示 |
| `DA_Frontend`（Game） | `L_LyraFrontEnd` | `B_LyraFrontEnd_Experience` | 1 | - |

### 3.10 机器人如何生成

1. Experience 通过 `AddComponents` 把 `B_ShooterBotSpawner`（父类 C++ `ULyraBotCreationComponent`）仅挂到服务器的 `ALyraGameState`。
2. `ULyraBotCreationComponent::BeginPlay` 向 `ULyraExperienceManagerComponent` 注册 `CallOrRegister_OnExperienceLoaded_LowPriority`；Experience 加载完成后（`WITH_SERVER_CODE && HasAuthority`）调用 `ServerCreateBots`。
3. 数量：`NumBotsToCreate`（Elimination=3、Control=3、Perf=20）-> 编辑器内可被 `ULyraDeveloperSettings.bOverrideBotCount/OverrideNumPlayerBotsToSpawn` 覆盖 -> 再被 URL 选项 `?NumBots=N` 覆盖。
4. `SpawnOneBot`：`SpawnActor<AAIController>(BotControllerClass)` -> 设置 PlayerName（`RandomBotNames` 随机取，用完后 "Tinplate NNN"）-> `GameMode->GenericPlayerInitialization` + `RestartPlayer`（会走 Team 分配 + `TDM_PlayerSpawningManagment` 选点 + PawnData 生成 Pawn）-> `PawnExtComponent->CheckDefaultInitialization()`。
5. 控制器类：`B_AI_Controller_LyraShooter`（TDM，行为树 `BT_Lyra_Shooter_Bot` + 黑板 `BB_Lyra_Shooter_Bot`，Services：`BTS_Shoot/CheckAmmo/ReloadWeapon/SetFocus`，Decorator `BTDecorator_AllowedToFire`，EQS：FindTarget/FindWeapon/MoveAgainstEnnemy/AIPerceptionEnemy）；占点用 `B_AI_Controller_LyraShooter_ControlPoint`（`BT_Lyra_Shooter_ControlPoint_Bot`）；Perf 用 `_Passive`。

### 3.11 游戏阶段（Game Phase）能力

`/ShooterCore/Experiences/Phases/`（均继承 C++ `ULyraGamePhaseAbility`，`ReplicateNo`、`ServerInitiated`、`ServerOnly`、`InstancedPerActor`，`ActivationPolicy=OnInputTriggered`）：

| 资产 | GamePhaseTag | 参数 / 依赖 |
|---|---|---|
| `Phase_Warmup` | `ShooterGame.GamePhase.Warmup` | `waitForPlayersDuration=30`、`countdownDuration=3`；依赖 `GE_PregameLobby`（预备阶段禁用输入/伤害）、`Phase_Playing`（结束后切入） |
| `Phase_Playing` | `ShooterGame.GamePhase.Playing` | 无额外参数 |
| `Phase_PostGame` | `ShooterGame.GamePhase.PostGame` | `newGameTime=10`；依赖 `GE_BlockAbilityInput`、`GE_DamageImmunity_FromGameMode` |

驱动方：`B_TeamDeathMatchScoring` 与 `B_ControlPointScoring`（GameState 组件）通过 `ULyraGamePhaseSubsystem` 启动 Warmup、胜利时启动 PostGame（资产依赖已核实；具体蓝图图表 DSL MCP 读不出，见 8）。同层级兄弟 Phase 会互相取消，所以 Warmup->Playing->PostGame 自然串联。TopDownArena 有自己的一套 `Phase_*`。

### 3.12 ShooterTests 测试清单与运行方式

C++ CQTest（`Source/ShooterTestsRuntime/Private/`）：

| 测试对象 | 类别 | 方法 | 地图 | 备注 |
|---|---|---|---|---|
| `InputCrouchAnimationTest`（`ACTOR_ANIMATION_TEST`） | `Project.Functional Tests.ShooterTests.Actor.Animation` | `PlayerCrouched_Forward/Backward/StrafeLeft/StrafeRight/RotateLeft/RotateRight Movement` | `L_ShooterTest_Basic` | 预期 `MM_Pistol_Crouch_*` 动画 |
| `WeaponMeleeAnimationTest`（`..._WITH_FLAGS`，EditorContext+ProductFilter） | 同上 | `WeaponMelee_Pistol/Rifle/Shotgun` | `L_ShooterTest_Basic` | 用 WeaponSpawner 拿武器 |
| `InputAnimationTest`（`ACTOR_ANIMATION_NETWORK_TEST`，仅编辑器） | `...Actor.Replication` | `NetworkPlayers_Jump`、`ServerPlayer_MannyMovement`、`ClientPlayer_QuinnMovement`、`NetworkPlayers_Crouch` | `/ShooterTests/Maps/L_ShooterTest_Basic` | 服务器+客户端 PIE |
| `AbilitySpawnerMapTest`（`TEST_CLASS_WITH_FLAGS`） | `...GameplayAbility` | `PlayerOnDamageSpawner_Eventually_LosesHealth`、`PlayerMissingHealth_OnHealSpawner_RestoresHealth` | `L_ShooterTest_Basic` | 使用 `FMapTestSpawner` |

另有蓝图 Functional Test：`B_Test_AutoRun`（地图 `L_ShooterTest_AutoRun`）、`B_Test_FireWeapon`（`L_ShooterTest_FireWeapon`），基类 `B_Test_Base`；其余地图：`L_AsyncMessagePassing`（`BP_Event_Broadcaster/Listener`、`B_HeavyBroadcaster/Listener`，配合 AsyncMessageSystem）、`L_ShooterTest_DeviceProperties`（`B_DevicePropertyTester`、`B_InputDeviceColor`、手柄触发器/声音震动测试 `Input/TriggerFeedback|TriggerVibration|SoundVibrations`，对应 C++ `ShooterTestsDevicePropertyTester` / `ShooterTestAsyncMessageTestActor`）。辅助 C++：`ShooterTestsActorTestHelper`、`ShooterTestsAnimationTestHelper`、`ShooterTestsInputTestHelper`（`FToggleCrouchTestAction`、`FMeleeTestAction`、`FJumpTestAction`、`FMove*/FLook*TestAction`）、`ShooterTestsActorNetworkTest.h`、`ShooterTestsNetworkComponent.h`。

运行方式：
- 编辑器：Tools -> Session Frontend -> Automation 页签，`Project -> Functional Tests -> ShooterTests`（或搜索 `ShooterTests`）；蓝图 Functional Test 在"所在地图名"下。
- MCP：`AutomationTestToolset`（`DiscoverTests` -> `ListTests` -> `RunTests` -> `GetTestStatus/GetTestResults`）。
- 命令行：`UnrealEditor-Cmd.exe LyraStarterGame.uproject -ExecCmds="Automation RunTests Project.Functional Tests.ShooterTests; Quit" -unattended -nullrhi`（网络类/EditorContext 测试需要有 RHI 的编辑器，不能 nullrhi）。
- 注意：本次只读检查，**未实际运行测试**。

---

## 4. 与核心代码的接口

| 方向 | 接口 | 说明 |
|---|---|---|
| 核心 -> 内容 | `ULyraExperienceDefinition`（`GameFeaturesToEnable: TArray<FString>`、`DefaultPawnData`、`Actions`、`ActionSets`） | Experience 蓝图只是它的子类；`ULyraExperienceActionSet` 是可复用的 Actions 包 |
| 核心 -> 内容 | `ULyraPawnData`（PawnClass/AbilitySets/TagRelationshipMapping/InputConfig/DefaultCameraMode）、`ULyraAbilitySet`、`ULyraInputConfig`、`ULyraAbilityTagRelationshipMapping` | PawnData 决定"这个模式里的玩家长什么样、能做什么" |
| 核心 -> 内容 | `ULyraGamePhaseAbility` / `ULyraGamePhaseSubsystem` | 阶段能力基类 |
| 核心 -> 内容 | `ULyraBotCreationComponent`、`ULyraTeamCreationComponent`、`ULyraPlayerSpawningManagerComponent`、`ULyraTeamDefinition` | 蓝图/ C++ 子类在 ShooterCore/TopDownArena 里 |
| 核心 -> 内容 | `ULyraUserFacingExperienceDefinition`（`MapID/ExperienceID/ExtraArgs/Tile*/LoadingScreenWidget/bIsDefaultExperience/bShowInFrontEnd/bRecordReplay/MaxPlayerCount`） | 前端列表项；`CreateHostingRequest` 把 `Experience=<名>` 加入 URL 参数，`ALyraGameMode::InitGame` 读取 |
| 核心 -> 内容 | `ULyraGameData`（`DefaultGameData`）、`ULyraAssetManager.DefaultPawnData` | 全局默认值 |
| 内容 -> 核心 | GFD 的 `PrimaryAssetTypesToScan` | 让 AssetManager 发现各插件目录下的 Experience / ActionSet / Playlist / Map / LobbyBackground |
| 内容 -> 核心 | GameplayTag：`HUD.Slot.*`（`UIExtension` 槽位）、`UI.Layer.Game`、`InputTag.*`、`ShooterGame.GamePhase.*`、`Lyra.*.Message`(GameplayMessageRouter 通道)、`ShooterGame.Score.*`（Eliminations/Deaths/Assists/ControlPointCapture）、`ShooterGame.ExtensionPoint.AbilityBar` | 各插件 `Config/Tags/*.ini` 定义（ShooterCore/ShooterExplorer/TopDownArena） |
| 内容 -> 核心 | GameplayMessage：`Lyra.Elimination.Message`、`Lyra.Damage.Message`、`Lyra.Assist.Message`、`Lyra.AddNotification.KillFeed`、`ShooterGame.ControlPoint.Captured.Message` | 计分、勋章、击杀信息都靠消息解耦 |
| GFP 策略 | `LyraGameFeaturePolicy`（`DefaultGame.ini GameFeaturesManagerClassName`） | 管理 GFP 加载策略 |

**AssetManager 配置（`Config/DefaultGame.ini`）**：`LyraExperienceDefinition` 目录 `/Game/System/Experiences` + 特例 `B_LyraFrontEnd_Experience`；`LyraUserFacingExperienceDefinition` 目录 `/Game/UI/Temp`、`/Game/System/Playlists`；`LyraExperienceActionSet`/`LyraLobbyBackground` 初始无目录（由各 GFD 追加）；`Map` 特例前端与 DefaultEditorOverview；`LyraGameData`=`/Game/DefaultGameData`。

---

## 5. 数据驱动点

| 想改什么 | 改哪个数据 |
|---|---|
| 模式用哪个插件 | Experience `GameFeaturesToEnable`（或 ActionSet 的同名字段） |
| 玩家是谁/技能/输入/相机 | `LyraPawnData`（Experience `DefaultPawnData`） |
| 给某类 Actor 加组件 | `GameFeatureAction_AddComponents`（Experience / ActionSet / GFD） |
| 给玩家加技能 | `GameFeatureAction_AddAbilities` -> `LyraAbilitySet`（挂在 `ALyraPlayerState`，授予时 ASC 在 PlayerState 上） |
| 加 HUD | `GameFeatureAction_AddWidgets`：Layout 到层(`UI.Layer.*`)，Widget 到槽(`HUD.Slot.*`) |
| 加输入 | `AddInputContextMapping`(IMC) + `AddInputBinding`(`LyraInputConfig`：Tag<->InputAction) |
| 机器人数量/类型 | `B_ShooterBotSpawner*.numBotsToCreate/botControllerClass/randomBotNames`；URL `?NumBots=` |
| 队伍 | `B_TeamSetup_*.teamsToCreate: TMap<int32, LyraTeamDefinition>`（TwoTeams/FourTeams） |
| 阶段时长 | `Phase_Warmup.waitForPlayersDuration/countdownDuration`、`Phase_PostGame.newGameTime`、`B_TeamDeathMatchScoring.waitForPlayersDuration/countDownTime` |
| 起手武器 | `B_Hero_ShooterMannequin` CDO `initialInventoryItems`（`ID_*`） |
| 武器行为 | `ID_*`（Fragments）-> `WID_*`（装备定义）-> `AbilitySet_*`（开火/换弹 GA 与输入 Tag）、`WeaponPickupData_*`（地图拾取点） |
| 勋章 | `DT_BasicShooterAccolades`(DataTable，`FLyraAccoladeDefinitionRow`) + `AccoladeDataRegistry`；连杀阈值在 `B_ElimChainProcessor.ElimChainTags`/`B_ElimStreakProcessor.ElimStreakTags` |
| 前端列表 | `ULyraUserFacingExperienceDefinition` 资产放入任意已注册插件的 `System/Playlists` |
| 全局伤害/治疗 GE | `DefaultGameData` |
| 瞄准辅助碰撞通道 | `ShooterCoreRuntimeSettings`（DefaultGame.ini） |
| 各模式 GameplayCue 搜索路径 | GFD `AddGameplayCuePath`（ShooterCore: `/GameplayCues`,`/Weapons`,`/Items`；ShooterMaps/TopDownArena: `/GameplayCues`） |

---

## 6. 扩展与修改配方

> 全部为编辑器操作 + 少量蓝图；以下对象路径均以本工程现有资产为模板。

### 6.1 新建一个带自己 Experience 与地图的 GameFeature 插件

1. 编辑器 Edit -> Plugins -> Add -> "Game Feature (Content Only)" 或 "(with C++)"（模板见 `DefaultEditor.ini [GameFeaturesEditorSettings]`），放到 `Plugins/GameFeatures/MyMode`。
2. `.uplugin`：保持 `ExplicitlyLoaded:true`、`EnabledByDefault:false`、`BuiltInInitialFeatureState:"Registered"`；若复用 Shooter 内容，在 `Plugins` 数组加 `ShooterCore`（和 `LyraExampleContent`）；带 C++ 则 Build.cs 依赖 `LyraGame`，`LoadingPhase:Default`。
3. 在 `MyMode.uasset`（GameFeatureData）里设置 `PrimaryAssetTypesToScan`：`LyraExperienceDefinition`（`System/Experiences`，`bHasBlueprintClasses=true`）、`LyraUserFacingExperienceDefinition`（`System/Playlists`，AlwaysCook）、`Map`（`/Maps`）、可选 `LyraExperienceActionSet`。**不要漏**，否则 AssetManager 找不到 Experience/Playlist（TopDownArena 即是此范本）。
4. 新建 `System/Experiences/B_MyExperience`（父类 `LyraExperienceDefinition`）：`GameFeaturesToEnable=["MyMode"]`，`DefaultPawnData`，`Actions`/`ActionSets`。
5. 新建地图 `Maps/L_MyMap`（放 `LyraPlayerStart`），World Settings 的 `Default Gameplay Experience` 可直接指到 `B_MyExperience`（编辑器直接 Play 时用）。
6. 新建 `System/Playlists/DA_MyMode`（`LyraUserFacingExperienceDefinition`）：MapID=`Map:/MyMode/Maps/L_MyMap`，ExperienceID=`LyraExperienceDefinition:B_MyExperience`，填 Tile 信息，`bShowInFrontEnd=true`。
7. 项目 uproject 的 `Plugins` 加 `{ "Name": "MyMode", "Enabled": true }`（5 个现有 GFP 都是这么做的）。
8. 要用 Shipping 打包：确认该 playlist 的 `CookRule=AlwaysCook`（GFD 里设）。

### 6.2 在 ShooterCore 里加一个新玩法模式（以"全员 FFA"为例）

最小路径：复制 `B_ShooterGame_Elimination` 为 `B_ShooterGame_FFA`（同目录 `/ShooterCore/Experiences/`，会被 GFD 的 `Experiences` 目录自动扫描）：
1. 保留 `GameFeaturesToEnable=[ShooterCore]`、`DefaultPawnData=HeroData_ShooterGame`、4 个 ActionSet（输入/组件/HUD/勋章可直接复用）。
2. 复制 `B_TeamSetup_TwoTeams` -> `B_TeamSetup_FFA`（`teamsToCreate` 用 4~8 个 TeamDA；或父类换成你自己的 `LyraTeamCreationComponent`），替换 AddComponents 里的队伍组件。
3. 计分：复制 `B_TeamDeathMatchScoring`（父 `B_ShooterGameScoring_Base`）改胜利条件；或新写 `UGameStateComponent` 子类（C++ 放 `ShooterCoreRuntime`）。保留对 `Phase_Warmup/PostGame` 的调用。
4. 出生规则：`B_TeamSpawningRules`（C++ `UTDM_PlayerSpawningManagmentComponent`）按"最远敌人"选点，FFA 通常也适用；如需别的规则，继承 `ULyraPlayerSpawningManagerComponent` 重写 `OnChoosePlayerStart`。
5. HUD：新建/复用记分 Widget，通过 `AddWidgets` 挂 `HUD.Slot.TeamScore`；记分板 `GA_ShowLeaderboard_*` + AbilitySet。
6. 机器人：复制 `B_ShooterBotSpawner` 改数量/控制器。
7. Playlist：`/ShooterMaps/System/Playlists/` 新建 `DA_Expanse_FFA`（ExperienceID=`LyraExperienceDefinition:B_ShooterGame_FFA`）。

### 6.3 新增一个英雄（Hero）

1. Pawn：复制 `B_Hero_ShooterMannequin`（或其父 `B_Hero_Default`）为 `B_Hero_MyHero`；改 Mesh/AnimBP/`B_MannequinPawnCosmetics`；起手装备改 `initialInventoryItems`/`weaponId`。
2. AbilitySet：复制 `AbilitySet_ShooterHero` 为 `AbilitySet_MyHero`，增删技能与 InputTag（InputTag 必须在 InputConfig 中有对应 Ability InputAction，否则按键不会触发）。
3. InputConfig：复制 `/Game/Input/InputData_Hero`（或使用 ShooterCore 的 `InputData_ShooterGame_AddOns` 叠加）。Tag 与 `IMC_*` 里的 InputAction 通过 `LyraInputConfig` 绑定；新按键还需要在某个 IMC（`IMC_ShooterGame` / `IMC_Default`）里绑物理键。
4. PawnData：复制 `HeroData_ShooterGame` 为 `HeroData_MyHero`，填 PawnClass/AbilitySets/TagRelationshipMapping/InputConfig/CameraMode。
5. 在 Experience 的 `DefaultPawnData` 引用它；若希望同一模式有多种英雄，需要自己在 `ALyraGameMode::GetPawnDataForController`（读取 PlayerState 的 PawnData）之前由组件设置 `ALyraPlayerState::SetPawnData`（参考 `LyraPlayerState` / `LyraGameMode.cpp`）。
6. 外观多样性可以只换 `B_PickRandomCharacter` / `B_CharacterSelection` 里的 CharacterParts，而不新建 PawnData。

### 6.4 给某个模式的起手配置添加武器

- **起手背包**：本工程把起手装备写在 Pawn（`B_Hero_ShooterMannequin.initialInventoryItems`）里，因此是"所有用该 PawnData 的模式共享"。若只想改某个模式：
  - 方案 A：新建一个 PawnClass 子类（如 `B_Hero_ShooterMannequin_Rifle`，覆盖 `initialInventoryItems=[ID_Pistol_C, ID_Rifle_C]`）+ 新 PawnData，只给该 Experience 用。
  - 方案 B（不动 Pawn）：新写一个 `GameStateComponent`/`ControllerComponent`（挂到 `AController`，仅 Server），在 Experience 加载后调用 `ULyraInventoryManagerComponent::AddItemDefinition(ID_xxx)`，再用 `ULyraQuickBarComponent` 装槽；用 `AddComponents` 加到该 Experience 即可。
  - 方案 C：`AddAbilities` 授予一个"开局发枪"GA（对 PlayerState 的 ASC），在 GA 中给背包加物品。
- 新武器本身：复制 `Weapons/Rifle/` 全套 -> `ID_/WID_/AbilitySet_/B_WeaponInstance_/B_<Weapon>/GA_Weapon_Fire_/GE_Damage_/GCN_Weapon_*_Fire/W_Reticle_/W_AmmoCounter_/WeaponPickupData_`；`WID` 的 `ActorsToSpawn` 的 `attachSocket=weapon_r`、`AbilitySetsToGrant`；`ID` 的 Fragment（QuickBarIcon/SetStats(弹匣/备弹 Tag `Lyra.ShooterGame.Weapon.MagazineAmmo` 等)/ReticleConfig）；GameplayCue 要放在 `/ShooterCore/Weapons`（已在 AddGameplayCuePath 里）；地图上放 `B_WeaponSpawner` 并指定新的 `WeaponPickupData_*`。
- 快捷栏容量与切换由 `B_QuickBarComponent`（C++ `ULyraQuickBarComponent`）和 `GA_QuickbarSlots` + `IA_QuickSlot*` 控制。

### 6.5 在前端注册新 Experience

1. 把 `LyraUserFacingExperienceDefinition` 资产放到 `<任一已注册 GFP>/Content/System/Playlists/`（该 GFP 的 GFD 必须有 `LyraUserFacingExperienceDefinition` -> `System/Playlists` 的 `PrimaryAssetTypesToScan`，ShooterCore/ShooterMaps/TopDownArena/ShooterTests 都有；ShooterExplorer 没有，需先补）；或放进 `/Game/System/Playlists`（`DefaultGame.ini` 已扫描）。
2. 填 MapID / ExperienceID / Tile 标题、描述、图标（`/Game/UI/Menu/Art/T_UI_MapTile_*`）/ `LoadingScreenWidget` / `MaxPlayerCount`；`bShowInFrontEnd=true`；`bIsDefaultExperience=true` 表示 Quick Play / 专用服务器默认。
3. 前端 `W_ExperienceList`/`W_ExperienceSelectionScreen` 自动枚举所有 `bShowInFrontEnd` 的 playlist，不需要改前端蓝图。
4. 对应 Experience 所在的 GFP 在进入时由 `GameFeaturesToEnable` 激活；该 GFP 必须在 uproject 中启用。
5. Dedicated server 想默认启用：设为 `bIsDefaultExperience`，且服务器的 `L_LyraFrontEnd` 路径会走 `LyraGameMode.cpp:173` 的分支。

### 6.6 其他常用食谱

- **新增一个 ActionSet 复用 HUD/输入**：右键 `LyraExperienceActionSet`，把共用 Actions 放进去，在多个 Experience 的 `ActionSets` 里引用（Shooter 三件套就是这样复用的）。注意其目录需被 GFD 的 `LyraExperienceActionSet` 扫描。
- **新增勋章**：`DT_BasicShooterAccolades` 加一行（Sound/Icon/DisplayDuration/LocationTag/AccoladeTags）；在 Processor 的 `ElimChainTags/ElimStreakTags` 里把计数映射到新 Tag，并在 `Config/Tags` 里声明 Tag。
- **新增 HUD 槽**：在 `W_ShooterHUDLayout` 加 `UIExtensionPointWidget` 并设置 `ExtensionPointTag=HUD.Slot.XXX`，Tag 写入某个 GFP `Config/Tags/*.ini`。

---

## 7. 自定义插件（非 Epic 原版）

### 7.1 LyraExtTool（有 C++，仅 Editor）

| 项 | 内容 |
|---|---|
| 位置 | `Plugins/LyraExtTool` |
| .uplugin | `Category: Other`，`CanContainContent: true`，**无 `EnabledByDefault:false`**（默认启用）；`Modules: [{Name: LyraExtTool, Type: Editor, LoadingPhase: Default}]` -> **只在编辑器中存在，不会进入打包的游戏** |
| Build.cs | 公共依赖 `Core`；私有 `CoreUObject, Engine, Slate, SlateCore` |
| 模块类 | `FLyraExtToolModule`（`StartupModule/ShutdownModule` 为空，纯模板） |
| 唯一功能类 | `UBPFunctionLibrary : UBlueprintFunctionLibrary`（`Public/BPFunctionLibrary.h`），`UFUNCTION(BlueprintCallable, Category="LyraExt") static bool ChangeMeshMaterials(TArray<UStaticMesh*> Mesh, UMaterialInterface* Material)` |
| 实现 | 遍历传入的 StaticMesh：`Modify()` -> 把 `GetStaticMaterials()` 里每个 `FStaticMaterial.MaterialInterface` 都替换成同一个 Material -> `PostEditChange()`；永远返回 `true`；无 null 检查，不 `MarkPackageDirty`/不保存 |
| 内容 | `/LyraExtTool/EUW_MaterialTool`：`EditorUtilityWidget`（Blutility）蓝图，用于在编辑器里批量把选中 StaticMesh 的所有材质槽换成指定材质（推测由 `ChangeMeshMaterials` 驱动；其图表内容 MCP/磁盘未能完整解码，属推断） |
| 集成方式 | 与游戏运行时、Experience、GFP 完全无关；仅是美术/关卡灰盒工具。`DefaultEditor.ini` 的本地化 Gather 配置里把 `Plugins/LyraExtTool/*` 加入了 `ExcludePathWildcards`（不参与本地化收集）。`Binaries/Win64/UnrealEditor-LyraExtTool.dll` 已编译 |

注意：这是一个极小的自定义编辑器工具模块(文件头仍是 Epic 模板版权声明，不能据此判定是否 Epic 官方，原文“唯一一处非 Epic 官方”无法证实)；因为 `UBPFunctionLibrary` 的 `UCLASS` 没有 `LYRAEXTTOOL_API`，其它 C++ 模块无法直接链接调用，只能被蓝图/EUW 调用（够用）。

### 7.2 GreenRoom / RedRoom（纯内容测试房间）

| 项 | GreenRoom | RedRoom |
|---|---|---|
| .uplugin | `Category: Other`，`ExplicitlyLoaded:true`，`EnabledByDefault:false`，无 Modules/依赖 | 同左 |
| 内容 | `GreenRoom.umap` + `Green.uasset`（一个 `Material`） | `RedRoom.umap` + `Red.uasset`（`Material`） |
| 地图内容（从包内字符串解析） | DirectionalLight、SkyLight、AtmosphericFog、SkySphere、StaticMeshActor(Cube)×N、`LyraPlayerStart`、TextRenderActor（房间名字） | 同左（颜色为红） |
| 用途 | 很可能是"色块参照房间"：用于测试材质/光照/相机/碰撞或作为简易空场景 | 同左 |
| 集成 | **不是 GameFeature**，没有 GFD、没有 Experience、没有 playlist，也**没有在 `LyraStarterGame.uproject` 里启用**；MCP `find_assets("/GreenRoom/")` 为空 -> 当前编辑器会话中没有挂载。要打开必须先在 Edit -> Plugins 启用，并在地图 World Settings 指定 `Default Gameplay Experience`（否则 `ALyraGameMode` 回退 `B_LyraDefaultExperience`，只有一个简单 Pawn 和默认 HUD）；或 `?Experience=` 进入 |

二者除颜色不同，结构完全对称。推测来源于 "Green Room / Red Room" 这类 Epic 内部标准测试房（本项目是额外加入的两个小插件）；**用途为推断，仓库内没有任何文档或引用证明**。

---

## 8. 坑与注意事项

1. **GFP 状态**：所有 GFP 是 `Registered`（不是 `Active`）。必须通过 Experience 的 `GameFeaturesToEnable` 激活；只在 ActionSet 里写 `AddComponents` 而忘了在 Experience/ActionSet 的 `GameFeaturesToEnable` 写插件，会出现"插件内容引用不到/ Action 没执行"。`B_TestInventoryExperience` 的 `GameFeaturesToEnable` 只有 ShooterCore，没有 ShooterExplorer（因为地图属于 ShooterExplorer，打开地图本身会使包挂载）。
2. **PrimaryAssetTypesToScan 遗漏**是最常见错误：Experience 放在 GFD 没有扫描的目录就找不到（`FPrimaryAssetId("LyraExperienceDefinition", Name)` 找不到 -> 回退默认体验）。`ShooterCore` 同时扫 `Experiences` 和 `System/Experiences`，其它插件只扫 `System/Experiences`。ShooterExplorer 的 GFD 没有扫 `System/Playlists`，也没有任何 playlist。
3. **Experience 内用到哪个 `Actor Class` 很重要**：`AddComponents/AddAbilities` 针对的是"类"，对子类实例也生效；`Controller`、`GameStateBase`、`LyraGameState` 层级不同：占点/记分挂 `LyraGameState`；勋章处理器挂 `GameStateBase`；技能（含 ASC）挂 `LyraPlayerState`，而不是 Pawn。
4. **Client/Server 勾选**：计分/机器人/队伍/出生规则仅 Server；音乐/HUD 相关仅 Client；漏勾导致 Widget 在服务器报错或客户端缺组件。注意 `bClientComponent/bServerComponent` 默认值，MCP 里 `false` 才显示出来。
5. **3 个 ActionSet 被 4 个 Experience 复用**：修改 `LAS_ShooterGame_*` 会影响 Elimination/Control/Perf/Explorer/Tests。尤其 `LAS_ShooterGame_StandardHUD` 把 HUD Layout 换成 `W_ShooterHUDLayout`，TopDownArena 则用自己的 `W_TopDownArenaHUDLayout`，两者互不兼容。
6. **起手武器写在 Pawn 里**（`B_Hero_ShooterMannequin`），不是在 Experience；改 Experience 的 PawnData 不会换武器，必须改 Pawn CDO 或新增组件/GA。
7. **`B_ShooterGame_Perf` 机器人是"被动型"控制器**，且额外 `numActorsToCreate=200`；不要把它当 TDM 用。
8. **`DA_ShooterGame_ShooterGym` 用的是占点 Experience**（tile 标题 "Control"/"Shooter Gym"），而不是 Elimination；`DA_ShooterGame_ShooterGym` 在 ShooterCore，`bShowInFrontEnd` 保持默认 true（ShooterCore 地图 `/ShooterCore/Maps/L_ShooterGym` 是开发用的测试体育馆）。
9. **ShooterTests 网络测试**：README 指出前端强制 `Force Standalone Net Mode`，新地图做网络测试需要在 World Settings 取消；默认随机 Manny/Quinn 会导致动画测试 flaky，应使用 `B_AutomatedShooterTest`（`B_CharacterSelection` 轮询）。CQTest 里 `EditorContext` 的用例不能在无 RHI 的命令行跑。`ShooterTestsRuntime` 在 Shipping 配置被排除。
10. **TopDownArena 属性与移动**：速度由 `UTopDownArenaAttributeSet::MovementSpeed` + `UTopDownArenaMovementComponent::GetMaxSpeed` 驱动；Pawn `B_Hero_Arena` 必须使用该移动组件，否则道具加速无效。`HeroData_Arena` 没有 TagRelationshipMapping。
11. **命名拼写**：`TDM_PlayerSpawningManagmentComponent`（Managment 缺 e）、`EAS_BasicShooterAcolades`（Acolades 缺 c）是源码原样，引用时不要"纠正"。
12. **自定义插件风险**：LyraExtTool 的 `ChangeMeshMaterials` 会直接改静态网格资产的材质槽且不检查空指针、不保存（需手动保存），请先在版本控制下使用；它是 Editor 模块，运行/打包目标若误引用 `BPFunctionLibrary` 会编译失败（如 `LyraGame` 不能依赖它）。GreenRoom/RedRoom 未在 uproject 启用，引用它们的资产（`/GreenRoom/...`）在未启用时会找不到。
13. **未验证项**（MCP/磁盘均未能读出）：
    - `B_TeamDeathMatchScoring`/`B_ControlPointScoring`/`B_ShooterGameScoring_Base`/`B_Hero_ShooterMannequin` 等蓝图的事件图逻辑（`read_graph_dsl` 返回空）；因此"谁在何时启动 Warmup/PostGame、TargetScore 默认值、胜负判定细节"是基于资产依赖 + 属性推断，未读到图表。
    - `B_ShooterGame_Perf` 的 `numActorsToCreate/actorSpawners` 具体生成的 Actor 类；`B_ShooterBotSpawner*.randomBotNames`。
    - `GameFeatureAction_AddInputBinding` 的全部内部细节、`ApplyFrontendPerfSettingsAction` 的具体设置值。
    - GreenRoom/RedRoom/EUW_MaterialTool 的真实用途（基于包内字符串推断）。
    - 测试是否全部通过（未运行 PIE / 自动化测试）。
    - 各插件 `Config/Tags/*.ini` 只读取了前 60 行 ShooterCore、全部 TopDownArena；ShooterExplorer 的 Tag 文件未单独核对。

---

## 附：资产引用图（asset map，精简）

```
DefaultEngine.ini  GameDefaultMap ─▶ L_LyraFrontEnd ◀─ DA_Frontend ─▶ B_LyraFrontEnd_Experience
                                                                   ├─ B_LyraFrontendStateComponent ─▶ W_LyraStartup / W_LyraFrontEnd
                                                                   └─ B_MusicManagerComponent_FE, W_PerfStatContainer_FrontEnd
GlobalDefaultGameMode ─▶ B_LyraGameMode ─(InitGame: URL ?Experience= / WorldSettings / dev settings / fallback)─▶ Experience
DefaultGame.ini  LyraAssetManager ─▶ DefaultGameData, DefaultPawnData_EmptyPawn

W_ExperienceList (前端) ─枚举─▶ UserFacingExperience(DA_*) ─MapID─▶ L_* 地图
                                                        └ExperienceID─▶ B_*Experience
  DA_Expanse_TDM(ShooterMaps) ───────────▶ L_Expanse           ─▶ B_ShooterGame_Elimination
  DA_Convolution_ControlPoint(ShooterMaps)▶ L_Convolution_Blockout─▶ B_LyraShooterGame_ControlPoints
  DA_ShooterGame_ShooterGym(ShooterCore) ▶ L_ShooterGym         ─▶ B_LyraShooterGame_ControlPoints
  DA_ShooterGame_ShooterPerf(ShooterCore)▶ L_ShooterPerf        ─▶ B_ShooterGame_Perf
  DA_TopDownArena_Splode / _Multiplayer ─▶ L_TopDownArenaGym / L_TopDown_LocalMultiplayer ─▶ B_TopDownArena(_Multiplayer)_Experience

B_ShooterGame_Elimination ─ GFP: ShooterCore
   ├ PawnData: HeroData_ShooterGame ─▶ B_Hero_ShooterMannequin(▶ID_Pistol▶WID_Pistol▶AbilitySet_ShooterPistol▶GA_Weapon_Fire_Pistol)
   │                                  ├ AbilitySet_ShooterHero ─▶ GA_Hero_Jump/Death/Dash, GA_Emote, GA_QuickbarSlots, GA_ADS, GA_Grenade, GA_DropWeapon, GA_Melee, GA_SpawnEffect, GE_IsPlayer
   │                                  ├ TagRelationships_ShooterHero
   │                                  └ InputData_Hero (+ InputData_ShooterGame_AddOns via LAS_SharedInput)
   ├ ActionSets: LAS_SharedInput / LAS_StandardComponents / LAS_StandardHUD / EAS_BasicShooterAcolades
   ├ AddAbilities(PlayerState): AbilitySet_Elimination ─▶ GA_ShowLeaderboard_TDM(▶W_MatchScoreBoard_Elimination), GA_AutoRespawn(▶W_RespawnTimer)
   ├ AddComponents(GameState): B_TeamDeathMatchScoring(▶Phase_Warmup▶GE_PregameLobby,Phase_Playing; ▶Phase_PostGame▶GE_BlockAbilityInput,GE_DamageImmunity)
   │                           B_MusicManagerComponent_Elimination, B_ShooterBotSpawner(▶B_AI_Controller_LyraShooter▶BT_Lyra_Shooter_Bot)
   │                           B_TeamSetup_TwoTeams(▶TeamDA_Red,TeamDA_Blue), B_TeamSpawningRules(C++ UTDM_PlayerSpawningManagmentComponent)
   │                (Controller): B_PickRandomCharacter
   └ AddWidgets: HUD.Slot.TeamScore ▶ W_ScoreWidget_Elimination
ShooterCore GFD ─▶ AddComponents(EquipmentManager/InventoryManager/WeaponState/IndicatorManager/AimAssistTargetManager/EliminationFeedRelay/HandleShooterReplays)
               ─▶ DataRegistry: AccoladeDataRegistry(DT_BasicShooterAccolades) ─▶ GameplayCuePath ─▶ IMC_Default + IMC_ShooterGame

TopDownArena GFD / B_TopDownArenaExperience ─▶ HeroData_Arena ─▶ B_Hero_Arena(UTopDownArenaMovementComponent), AbilitySet_Arena(GA_ArenaHero_Death, GA_DropBomb, UTopDownArenaAttributeSet),
                                               InputData_Arena, CM_ArenaFramingCamera(ULyraCameraMode_TopDownArenaCamera)
   AddComponents: B_PlayerUIComponent, B_TeamSetup_FourTeams, B_TopDownArena_GameComponent_Lives (+SpawnLocalPlayers in Multiplayer)

ShooterExplorer: B_TestInventoryExperience ─▶ HeroData_Explorer ─▶ B_Hero_Explorer, InputData_InventoryTest ; LAS_InventoryTest ─▶ IMC_InventoryTest, AbilitySet_InventoryTest(GA_Interact/ToggleMap/ToggleInventory/ToggleMarkerInWorld)
ShooterTests: B_BasicShooterTest / B_AutomatedShooterTest (TDM 精简版, 无 bot) ─ L_ShooterTest_* ─ ShooterTestsRuntime(CQTest) / B_Test_* (Functional Test)
LyraExtTool: UBPFunctionLibrary::ChangeMeshMaterials ◀─ EUW_MaterialTool (Editor Utility Widget)     GreenRoom/RedRoom: 孤立测试地图（无引用）
```

---

## 9. 资产引用链（逐 Experience / 逐武器；除标注"推断"外均经 MCP 读取属性或 get_dependencies 核实）

> 编号说明：模板第 8 节是"坑与注意事项"，协调者追加的"资产引用链 / 速查问答 / 设计意图"顺延为第 9 / 10 / 11 节。
> 路径简写：SC=/ShooterCore、SM=/ShooterMaps、TD=/TopDownArena、SE=/ShooterExplorer、ST=/ShooterTests、G=/Game。

### 9.1 前端链（游戏启动）

```
DefaultEngine.ini GameDefaultMap
  └ G/System/FrontEnd/Maps/L_LyraFrontEnd  (对应 DA_Frontend 的 MapID)
      └ ALyraGameMode(B_LyraGameMode) InitGame 解析 Experience
          └ G/System/FrontEnd/B_LyraFrontEnd_Experience   (GameFeaturesToEnable 为空, 无 PawnData)
              ├ GameFeatureAction_SplitscreenConfig (bDisableSplitscreen=true)
              ├ AddComponents[GameState] G/UI/B_LyraFrontendStateComponent (Client)
              │      └ G/UI/Menu/W_LyraStartup -> G/UI/Menu/W_LyraFrontEnd
              │             └ G/UI/Menu/Experiences/W_ExperienceSelectionScreen / W_ExperienceList / W_ExperienceTile / W_HostSessionScreen
              │                    └ 枚举 bShowInFrontEnd 的 LyraUserFacingExperienceDefinition -> CreateHostingRequest -> travel ?Experience=<名>
              ├ AddComponents[GameState] G/Audio/Blueprints/B_MusicManagerComponent_FE (Client)
              ├ ApplyFrontendPerfSettingsAction (CDO 内嵌 UObject)
              └ AddWidgets HUD.Slot.PerfStats.Text <- G/UI/PerfStats/W_PerfStatContainer_FrontEnd
```

### 9.2 通用 Shooter 玩家链（所有 ShooterCore 系 Experience 共享）

```
HeroData_ShooterGame  (SC/Game/HeroData_ShooterGame, ULyraPawnData)
 ├ PawnClass  SC/Game/B_Hero_ShooterMannequin -> G/Characters/Heroes/B_Hero_Default -> G/Characters/Character_Default -> ALyraCharacter
 │     ├ 依赖: G/Characters/Cosmetics/B_MannequinPawnCosmetics (Manny/Quinn Invis 网格规则 + PA_Mannequin)
 │     ├ 依赖: G/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base, Meshes/SKM_Manny_Invis, Rig/PA_Mannequin
 │     ├ 依赖: SC/Game/B_TeamColor_DeviceProperty, G/Effects/Materials/PostProcess/MPP_Outline_Inst, G/Audio/Sounds/Emotes/BI_EmoteSoundInterface
 │     ├ 依赖: SC/Input/Actions/IA_QuickSlot1/2/3/CycleForward/CycleBackward (Pawn 内直接绑定)
 │     └ 起手: weaponId=SC/Weapons/Pistol/WID_Pistol ; initialInventoryItems=[SC/Weapons/Pistol/ID_Pistol]
 ├ AbilitySets SC/Game/AbilitySet_ShooterHero
 │     ├ G/Characters/Heroes/Abilities/GA_Hero_Jump (InputTag.Jump), GA_Hero_Death
 │     ├ SC/Game/Dash/GA_Hero_Dash (+W_DashCooldown, GCNL_Dash), SC/Game/Emote/GA_Emote
 │     ├ SC/Game/GA_QuickbarSlots
 │     ├ SC/Input/Abilities/GA_ADS, GA_Grenade (+SC/Weapon/Grenade/B_Grenade, GE_Damage_Grenade, GE_Grenade_Cooldown), GA_DropWeapon
 │     ├ SC/Game/Melee/GA_Melee (+GE_Damage_Melee, GCN_Weapon_Melee/MeleeImpact)
 │     ├ SC/Game/Respawn/GA_SpawnEffect (+GE_SpawnIn), LyraGameplayAbility_Reset (C++)
 │     └ GE: G/GameplayEffects/GE_IsPlayer
 ├ TagRelationshipMapping SC/Game/TagRelationships_ShooterHero
 ├ InputConfig G/Input/InputData_Hero  (+ SC/Input/Actions/InputData_ShooterGame_AddOns, 由 LAS_SharedInput 追加)
 │     └ IA_* 在 G/Input/Actions 与 SC/Input/Actions; 物理键位在 G/Input/Mappings/IMC_Default 与 SC/Input/Mappings/IMC_ShooterGame(+_KBM, IMC_ADS_Speed)
 └ DefaultCameraMode G/Characters/Cameras/CM_ThirdPerson (ADS 时 SC/Camera/CM_ThirdPersonADS)
角色外观: Controller 上的 G/Characters/Cosmetics/B_PickRandomCharacter (随机 B_Manny / B_Quinn;
        staticGameplayTags 分别为 Cosmetic.AnimationStyle.Masculine / Feminine + Cosmetic.BodyStyle.Medium)
        -> ULyraControllerComponent_CharacterParts -> B_MannequinPawnCosmetics 选 Invis 网格
        -> 武器动画层按 Cosmetic.AnimationStyle 标签选 ABP_<Weapon>AnimLayers 或 _Feminine 变体
```

### 9.3 B_ShooterGame_Elimination 全链

```
SM/System/Playlists/DA_Expanse_TDM -> Map SM/Maps/L_Expanse ; W_LoadingScreen_TDM ; tile T_UI_MapTile_Expanse
 └ SC/Experiences/B_ShooterGame_Elimination
    ├ GameFeaturesToEnable: ShooterCore
    ├ DefaultPawnData: HeroData_ShooterGame (见 9.2)
    ├ ActionSets: LAS_ShooterGame_SharedInput / StandardComponents / StandardHUD / SC/Accolades/EAS_BasicShooterAcolades
    ├ AddAbilities[PlayerState]: SC/Elimination/AbilitySet_Elimination
    │      ├ GA_ShowLeaderboard_TDM (InputTag.Ability.ShowLeaderboard) -> SC/Input/Abilities/GAB_ShowWidget_WhileInputHeld -> SC/Elimination/UI/W_MatchScoreBoard_Elimination
    │      │       -> W_ScoreBoard_HeaderRow_EliminationMode, W_SB_PlayerRow_TDM -> SC/UserInterface/ScoreBoard/PlayerRow/{W_BasePlayerRow, W_SB_PlayerName, W_SB_PlayerStat, W_SB_PlayerIcon}, W_SB_PlayerList, W_SB_TeamStat
    │      └ GA_AutoRespawn -> SC/Game/Respawn/W_RespawnTimer
    ├ AddComponents[GameState]:
    │      ├ SC/Elimination/B_TeamDeathMatchScoring (父 SC/Game/B_ShooterGameScoring_Base, 一个 GameStateComponent)
    │      │       ├ 依赖 SC/Experiences/Phases/Phase_Warmup (-> GE_PregameLobby, Phase_Playing)
    │      │       ├ 依赖 Phase_PostGame (-> G/GameplayEffects/GE_BlockAbilityInput, GE_DamageImmunity_FromGameMode)
    │      │       └ Base 依赖 SC/Blueprint/B_WeaponSpawner 与 SC/Weapon/Grenade/B_Grenade (Warmup 结束时重置)
    │      ├ SC/Elimination/B_MusicManagerComponent_Elimination (Client)
    │      ├ SC/Bot/B_ShooterBotSpawner (Server; ULyraBotCreationComponent, numBotsToCreate=3, controller=B_AI_Controller_LyraShooter)
    │      │       -> SC/Bot/BT/BT_Lyra_Shooter_Bot + BB_Lyra_Shooter_Bot + BTDecorator_AllowedToFire + Services(BTS_Shoot/CheckAmmo/ReloadWeapon/SetFocus) + SC/Bot/EQS/*
    │      ├ SC/Game/B_TeamSetup_TwoTeams (Server; {1:G/System/Teams/TeamDA_Red, 2:TeamDA_Blue})
    │      └ SC/Game/B_TeamSpawningRules (Server; C++ UTDM_PlayerSpawningManagmentComponent)
    ├ AddComponents[Controller]: G/Characters/Cosmetics/B_PickRandomCharacter (Server)
    └ AddWidgets: HUD.Slot.TeamScore <- SC/Elimination/UI/W_ScoreWidget_Elimination
ShooterCore GFD 常驻: Equipment/Inventory/WeaponState/Indicator/AimAssist/EliminationFeedRelay/HandleShooterReplays + DataRegistry(AccoladeDataRegistry) + GameplayCue 路径 + IMC
勋章: EAS_BasicShooterAcolades -> B_ElimChainProcessor / B_ElimStreakProcessor / AssistProcessor(C++) / B_AccoladeRelay
      -> (消息 Lyra.ShooterGame.Accolade.*) -> W_AccoladeHostWidget(ULyraAccoladeHostWidget) -> DataRegistry -> DT_BasicShooterAccolades
击杀信息: B_EliminationFeedRelay -> Lyra.AddNotification.KillFeed -> W_EliminationFeed -> W_EliminationFeedEntryWidget / ElimFeedEntry
```

### 9.4 B_LyraShooterGame_ControlPoints 全链

```
SM/System/Playlists/DA_Convolution_ControlPoint (L_Convolution_Blockout, W_LoadingScreen_ControlPoint)
SC/System/Playlists/DA_ShooterGame_ShooterGym (L_ShooterGym)
 └ SC/Experiences/B_LyraShooterGame_ControlPoints
    ├ PawnData / ActionSets 同 Elimination
    ├ AddWidgets: HUD.Slot.TeamScore <- SC/ControlPoint/UI/W_CPScoreWidget ; HUD.Slot.ModeStatus <- W_ControlPointStatusWidget (消息 FLyraControlPointStatusMessage)
    ├ AddAbilities[PlayerState]: SC/ControlPoint/AbilitySet_ControlPoint -> GA_ShowLeaderboard_CP -> W_MatchScoreBoard_CP (W_SB_PlayerRow_CP, W_ScoreBoard_HeaderRow_CP) ; GA_AutoRespawn
    ├ AddComponents[GameState]: B_ControlPointScoring (-> Phase_Warmup, Phase_PostGame ; teamColors, delayBetweenScoring=1) ; B_MusicManagerComponent_ControlPoint ;
    │      SC/ControlPoint/AI/B_ShooterBotSpawner_ControlPoint (numBots=3, controller=B_AI_Controller_LyraShooter_ControlPoint -> BT_Lyra_Shooter_ControlPoint_Bot, EQS_FindControlPoint) ;
    │      B_TeamSetup_TwoTeams ; B_TeamSpawningRules
    └ 地图内依赖 SC/Blueprint/B_ControlPointVolume (占领体积, 广播 ShooterGame.ControlPoint.Captured.Message), W_ControlPointMarker
```

### 9.5 其它 Experience 链

| Experience | 链 |
|---|---|
| SC/Experiences/B_ShooterGame_Perf | DA_ShooterGame_ShooterPerf -> L_ShooterPerf -> 同 Elimination，但 Bot=B_ShooterBotSpawner_Perf（20 bot + 200 actor，B_AI_Controller_LyraShooter_Passive）；测试 Actor 在 SC/Blueprint/B_PerfTestActor、B_PerfTestSpawner |
| TD/System/Experiences/B_TopDownArenaExperience | DA_TopDownArena_Splode -> L_TopDownArenaGym；HeroData_Arena -> B_Hero_Arena（含 UTopDownArenaMovementComponent）/ AbilitySet_Arena（GA_ArenaHero_Death、GA_DropBomb->B_Bomb_Standard->GE_Damaged_By_Bomb、UTopDownArenaAttributeSet）/ InputData_Arena（IA_Move、IA_Weapon_Fire）/ CM_ArenaFramingCamera（ULyraCameraMode_TopDownArenaCamera）；组件 B_TopDownArena_GameComponent_Lives（+Phase_Warmup/Playing/PostGame、GE_Warmup）、B_TeamSetup_FourTeams、B_PlayerUIComponent（W_PlayerTile、W_Powerup_Stat）；HUD W_TopDownArenaHUDLayout（W_RoundTimer、W_TopDownPlayers）；道具 B_TopDownArena_Pickup -> GE_BombCountUp/BombRangeUp/MoveSpeedUp/AdditionalHeart（UTopDownArenaPickupUIData） |
| TD/.../B_TopDownArena_Multiplayer_Experience | 同上 + B_TopDownArena_GameComponent_SpawnLocalPlayers；DA_TopDownArena_Multiplayer -> L_TopDown_LocalMultiplayer |
| SE/System/Experiences/B_TestInventoryExperience | 无 playlist；HeroData_Explorer -> B_Hero_Explorer；LAS_InventoryTest -> IMC_InventoryTest + InputData_InventoryTest + AbilitySet_InventoryTest -> GA_Interact（GA_Interaction_Collect/Sit）/ GA_ToggleInventory / GA_ToggleMap（W_MapScreen）/ GA_ToggleMarkerInWorld；世界物体 B_InteractableChair/Rock（推断基于 C++ ALyraWorldCollectable），物品 TestID_Rock/Tree；UI W_InteractionPrompt、ItemAcquiredToastEntry |
| ST/System/Experiences/B_BasicShooterTest、B_AutomatedShooterTest | 同 Elimination 去掉 bot；Automated 使用 G/Characters/Cosmetics/B_CharacterSelection；地图 L_ShooterTest_* |
| G/System/Experiences/B_LyraDefaultExperience | DA_ExamplePlaylist -> L_DefaultEditorOverview；PawnData SimplePawnData -> B_SimpleHeroPawn + InputData_SimplePawn + CM_ThirdPerson；W_DefaultHUDLayout |

### 9.6 武器链（Rifle / Pistol / Shotgun，下表数值均由 MCP 读到）

通用骨架：

```
ID_<W>  (SC/Weapons/<W>/ID_<W>, ULyraInventoryItemDefinition, 5 个 Fragment)
 ├ InventoryFragment_EquippableItem  -> EquipmentDefinition = WID_<W>
 ├ InventoryFragment_QuickBarIcon    -> brush=G/UI/Hud/Art/T_UI_Icon_RangedWeapons_<W>, ammoBrush=..._Ammo, displayNameWhenEquipped
 ├ InventoryFragment_SetStats        -> initialItemStats { Lyra.ShooterGame.Weapon.MagazineSize / MagazineAmmo / SpareAmmo }
 ├ InventoryFragment_PickupIcon      -> skeletalMesh (G/Weapons/<W>/Mesh/SK_*), displayName, padColor (拾取台颜色)
 └ InventoryFragment_ReticleConfig   -> [W_Reticle_<W>, W_AmmoCounter_<W>]
WID_<W> (ULyraEquipmentDefinition)
 ├ InstanceType = B_WeaponInstance_<W>  (ULyraRangedWeaponInstance: 散布/热量曲线/距离衰减/弱点倍率/装备与近战蒙太奇/动画层)
 ├ AbilitySetsToGrant = AbilitySet_Shooter<W>
 └ ActorsToSpawn = B_<W> @ socket weapon_r
AbilitySet_Shooter<W>: GA_Weapon_Fire_<W>(输入 Tag) + GA_Weapon_Reload_<W>(InputTag.Weapon.Reload) + G/Weapons/GA_Weapon_AutoReload(无输入)
GA_Weapon_Fire_<W> (ULyraGameplayAbility_RangedWeapon 子类, BP)
 ├ gE_Damage = GE_Damage_<W> ; fireDelayTimeSecs / autoRate / traceMaxDistance / object Types ; characterFireMontage
 ├ additionalCosts = LyraAbilityCost_ItemTagStack(tag=Lyra.ShooterGame.Weapon.MagazineAmmo, quantity=1, failureTag=Ability.ActivateFail.Cost -> "No Ammo!" + DryFire 蒙太奇)
 ├ abilityTags = Ability.Type.Action.WeaponFire ; activationOwnedTags = Event.Movement.WeaponFire ; sourceBlockedTags = Ability.Weapon.NoFiring ; activationPolicy=WhileInputActive
 └ GameplayCue: GameplayCue.Weapon.<W>.Fire / .Impact -> SC/Weapons/<W>/GCN_Weapon_<W>_Fire (SC GFD 的 AddGameplayCuePath 含 /Weapons)
GE_Damage_<W>: Instant ; Execution = LyraDamageExecution(C++) ; CapturedAttribute LyraCombatSet.BaseDamage(Source, AddBase)=数值 ;
               标签 GameplayEffect.DamageType.<W> / Basic / DamageTrait.Instant ; Cue GameplayCue.Character.DamageTaken(magnitude=LyraHealthSet.Damage)
```

| 武器 | 开火 GA / 输入 Tag | 伤害 GE（BaseDamage） | fireDelay / autoRate / traceMax | 弹匣 / 备弹 | 散布 min~max / 每发弹丸 / 弱点倍率 | 其它 |
|---|---|---|---|---|---|---|
| Pistol | SC/Weapons/Pistol/GA_Weapon_Fire_Pistol / InputTag.Weapon.Fire | G/Weapons/Pistol/GE_Damage_Pistol（18） | 0.12 / 1 / 10000 | 12 / 48 | 2.5~12 / 1 / x2.0；扫掠半径 6 | 换弹 G/Weapons/Pistol/GA_Weapon_Reload_Pistol；Montage G/Weapons/Pistol/Animations/AM_MM_Pistol_Fire |
| Rifle | SC/Weapons/Rifle/GA_Weapon_Fire_Rifle_Auto / InputTag.Weapon.FireAuto | SC/Weapons/Rifle/GE_Damage_RifleAuto（12） | 0.12 / 1 / 10000 | 30 / 60 | 2.5-15 / 1 / x1.5；扫掠半径 5.5；maxDamageRange=25000；距离衰减曲线 0-2800 为 1，2801 起 0.5 | 换弹 GA_Weapon_Reload_Rifle；动画层 ABP_RifleAnimLayers（Feminine 变体按 Cosmetic.AnimationStyle.Feminine）；装备/近战蒙太奇 AM_MM_Rifle_Equip / AM_MM_Rifle_Melee |
| Shotgun | SC/Weapons/Shotgun/GA_Weapon_Fire_Shotgun / InputTag.Weapon.FireAuto | SC/Weapons/Shotgun/GE_Damage_Shotgun（12/颗） | 0.5 / 2.2 / 1000 | 8 / 16 | 6~20 / 9 颗 / x1.75；扫掠半径 0.5 | 换弹 GA_Weapon_Reload_Shotgun（另有 GA_Weapon_Reload_NetShooter）；Montage AM_MM_Shotgun_Fire |

- 伤害最终值（Source/LyraGame/AbilitySystem/Executions/LyraDamageExecution.cpp 约 117-136 行）：DamageDone = max(BaseDamage x DistanceAttenuation x PhysicalMaterialAttenuation x DamageInteractionAllowedMultiplier, 0)，写入 LyraHealthSet.Damage 元属性；DistanceAttenuation 来自武器实例的 DistanceDamageFalloff 曲线；PhysicalMaterialAttenuation 来自 MaterialDamageMultiplier（Gameplay.Zone.WeakSpot）；友伤/自伤由 DamageInteractionAllowedMultiplier（队伍系统）决定。LyraHealthSet 默认 Health=MaxHealth=100（LyraHealthSet.cpp:22-23）。
- WeaponPickupData_<W>（ULyraWeaponPickupDefinition）-> InventoryItemDefinition=ID_<W>，spawnCoolDownSeconds=10（Rifle），displayMesh、拾取/重生音效；被地图中的 SC/Blueprint/B_WeaponSpawner 引用；治疗拾取 SC/Items/HealthPickup/WeaponPickupData_Heal_* -> GE_InstantHeal_*。
- 准星：W_WeaponReticleHost（HUD.Slot.Reticle）读取 ReticleConfig 的 Widget；弹药 HUD 由 W_AmmoCounter_* 读取 MagazineAmmo / SpareAmmo Tag 堆叠。
- 另有 SC/Weapons/NetShooter_PROTO/*（B_NetShooter、GA_WeaponNetShooter、ID/WID_NetShooter）网络测试原型武器，WID_InstantHeal 为治疗装备。

---

## 10. 速查问答（改什么在哪里改）

1. **改步枪伤害在哪？** SC/Weapons/Rifle/GE_Damage_RifleAuto -> Executions[0] -> CalculationModifiers 里 LyraCombatSet.BaseDamage 的 ScalableFloat（现 12）。手枪是 G/Weapons/Pistol/GE_Damage_Pistol（18），霰弹枪 SC/Weapons/Shotgun/GE_Damage_Shotgun（12 x 9 颗）。
2. **改步枪射速？** SC/Weapons/Rifle/GA_Weapon_Fire_Rifle_Auto 的 fireDelayTimeSecs（0.12s）与 autoRate（1）。霰弹枪是 0.5s / 2.2。
3. **改步枪弹匣/备弹？** SC/Weapons/Rifle/ID_Rifle -> InventoryFragment_SetStats.initialItemStats：Lyra.ShooterGame.Weapon.MagazineSize=30、MagazineAmmo=30、SpareAmmo=60（三项一起改）。
4. **改步枪散布/后座？** SC/Weapons/Rifle/B_WeaponInstance_Rifle：HeatToSpreadCurve、HeatToHeatPerShotCurve、HeatToCoolDownPerSecondCurve、SpreadAngleMultiplier_Aiming/StandingStill/Crouching/JumpingOrFalling、bulletsPerCartridge。
5. **改射程/距离衰减/爆头倍率？** 同上资产：DistanceDamageFalloff（0~2800=1，之后 0.5）、MaterialDamageMultiplier（Gameplay.Zone.WeakSpot=1.5）；射线长度在 GA 的 traceMaxDistance（10000，霰弹枪 1000）。
6. **每发消耗几发子弹？** GA 的 additionalCosts -> LyraAbilityCost_ItemTagStack（Tag=MagazineAmmo, quantity=1）。
7. **Elimination 的胜利条件和计分在哪？** SC/Elimination/B_TeamDeathMatchScoring（GameState 组件，父类 B_ShooterGameScoring_Base）：targetScore（复制变量，OnRep_TargetScore）、team0Score / team1Score、HandleVictory、OnEliminationScored，监听 Lyra.Elimination.Message。CDO 里 targetScore 读到 0，实际目标分在事件图/运行时设置，图表未能经 MCP 读取（见第 8 节未验证项）。
8. **Control（占点）的计分？** SC/ControlPoint/B_ControlPointScoring：delayBetweenScoring=1（占点方周期加分）、targetScore、teamColors；占点体积 SC/Blueprint/B_ControlPointVolume；消息 ShooterGame.ControlPoint.Captured.Message。
9. **热身/开局倒计时、赛后等待？** SC/Experiences/Phases/Phase_Warmup（waitForPlayersDuration=30、countdownDuration=3）、Phase_PostGame（newGameTime=10）；TDM 组件自身另有 waitForPlayersDuration=10、countDownTime=15。
10. **Bot 数量在哪配？** SC/Bot/B_ShooterBotSpawner.numBotsToCreate（3；Control 版 SC/ControlPoint/AI/B_ShooterBotSpawner_ControlPoint 也是 3；Perf 20）。临时覆盖：URL ?NumBots=N；编辑器 PIE：ULyraDeveloperSettings 的 bOverrideBotCount + OverrideNumPlayerBotsToSpawn。
11. **Bot 的 AI 在哪？** 控制器 SC/Bot/B_AI_Controller_LyraShooter -> 行为树 SC/Bot/BT/BT_Lyra_Shooter_Bot + 黑板 BB_Lyra_Shooter_Bot；射击/换弹/聚焦在 SC/Bot/BT/Services/BTS_*；寻敌 EQS 在 SC/Bot/EQS/；名字池 randomBotNames。
12. **怎么新增一个地图到菜单？** 新建 ULyraUserFacingExperienceDefinition（放任一 GFP 的 System/Playlists，如 SM/System/Playlists/DA_MyMap）：MapID=Map:/ShooterMaps/Maps/L_MyMap，ExperienceID=LyraExperienceDefinition:B_ShooterGame_Elimination，填 Tile*，bShowInFrontEnd=true；地图放进该 GFP 的 Maps/ 目录（GFD 的 Map 扫描项 /Maps）。
13. **地图里至少要放什么？** LyraPlayerStart、B_WeaponSpawner（拾取点）、导航网格（Bot）、占点模式需要 B_ControlPointVolume；直接 PIE 地图时在 World Settings 设 Default Gameplay Experience。
14. **怎么直接 PIE 某个 Experience？** 地图 World Settings 的 Default Gameplay Experience；或 ULyraDeveloperSettings.ExperienceOverride；或命令行 -Experience=B_ShooterGame_Elimination；或 URL ?Experience=（优先级见第 1 节）。
15. **默认英雄/玩家技能在哪加？** SC/Game/AbilitySet_ShooterHero（所有 Shooter 模式的玩家技能），或模式专属的 AbilitySet_Elimination / AbilitySet_ControlPoint（经 Experience 的 AddAbilities 挂到 ALyraPlayerState）。新技能的 InputTag 必须在 InputConfig 有对应 InputAction，并在 IMC_* 绑键。
16. **按键绑定在哪？** G/Input/Mappings/IMC_Default（移动/视角/跳/开火/换弹）、SC/Input/Mappings/IMC_ShooterGame 与 IMC_ShooterGame_KBM（ADS/手雷/近战/表情/快捷栏/排行榜）；由 LAS_ShooterGame_SharedInput 与 ShooterCore GFD 注册（bRegisterWithSettings=true 才会出现在设置的键位页）。
17. **起手武器怎么换？** SC/Game/B_Hero_ShooterMannequin CDO 的 initialInventoryItems（现为 ID_Pistol）和 weaponId。
18. **血量/移动速度？** 血量：ULyraHealthSet 默认 100（C++），可用 GE 覆盖；跳 / 冲刺：GA_Hero_Jump、SC/Game/Dash/GA_Hero_Dash；移动参数在 Pawn 的 CharacterMovement（G/Characters/Character_Default）。TopDownArena 速度来自 UTopDownArenaAttributeSet::MovementSpeed。
19. **复活逻辑？** SC/Game/Respawn/GA_AutoRespawn（PlayerState 上的技能，倒计时 UI W_RespawnTimer）；出生特效 GA_SpawnEffect + GE_SpawnIn；出生点选择 UTDM_PlayerSpawningManagmentComponent。
20. **队伍数量/颜色？** 队伍组件 SC/Game/B_TeamSetup_TwoTeams（teamsToCreate: {1:TeamDA_Red, 2:TeamDA_Blue}），TeamDA 在 G/System/Teams/；TopDownArena 用 TD/Game/B_TeamSetup_FourTeams。做 FFA/三队，复制该组件改 teamsToCreate。
21. **勋章/连杀怎么配？** SC/Accolades/B_ElimChainProcessor.ElimChainTags（2x..5x，ChainTimeLimit=4.5s）与 B_ElimStreakProcessor.ElimStreakTags（5/10/15/20）；展示配置在 DataTable DT_BasicShooterAccolades（FLyraAccoladeDefinitionRow），经 AccoladeDataRegistry；UI 为 W_AccoladeHostWidget / W_AccoladeToast。
22. **助攻怎么算？** C++ UAssistProcessor：累计每个受害者被各人造成的伤害，击杀时给非击杀者发 Lyra.Assist.Message；计分 Tag ShooterGame.Score.Assists。
23. **击杀提示 UI？** SC/Accolades/B_EliminationFeedRelay（ShooterCore GFD 常驻挂到 GameState）-> W_EliminationFeed（槽 HUD.Slot.EliminationFeed，由 LAS_ShooterGame_StandardHUD 注入）。
24. **HUD 怎么加一个 Widget？** 在 Experience/ActionSet 的 GameFeatureAction_AddWidgets 加条目：WidgetClass + SlotID（HUD.Slot.*，由 W_ShooterHUDLayout 里的 UIExtensionPointWidget 承载）；整页布局用 Layout + LayerID=UI.Layer.Game。
25. **装备/背包组件是在哪挂的？** ShooterCore GFD 的 GameFeatureAction_AddComponents（装备/背包/武器状态/指示器/回放/辅助瞄准/击杀 feed）以及各 Experience 的 AddComponents，不在核心 C++ 默认子对象里。
26. **怎么新增 GameFeature 并让前端看到？** 见 6.1 / 6.5：GFD 要有 LyraExperienceDefinition、LyraUserFacingExperienceDefinition、Map 的 PrimaryAssetTypesToScan，uproject 启用，playlist bShowInFrontEnd=true。
27. **前端的 Experience 到底是什么？** G/System/FrontEnd/B_LyraFrontEnd_Experience：无 PawnData，仅挂 B_LyraFrontendStateComponent（启动/登录/主菜单状态机）、前端音乐、禁分屏。
28. **怎么跑测试？** 编辑器 Session Frontend -> Automation -> Project.Functional Tests.ShooterTests（Actor.Animation、Actor.Replication、GameplayAbility）；或 MCP AutomationTestToolset；或命令行 -ExecCmds="Automation RunTests Project.Functional Tests.ShooterTests"。
29. **GreenRoom / RedRoom / LyraExtTool 是什么？** 见第 7 节：前两者是未启用的纯内容测试房（红/绿材质 + 灯光 + LyraPlayerStart）；LyraExtTool 是编辑器模块，提供 UBPFunctionLibrary::ChangeMeshMaterials 与 EUW_MaterialTool 批量换静态网格材质。
30. **改了 GFD 的 Action 为什么没生效？** 检查：该 GFP 是否 Active（Experience 的 GameFeaturesToEnable 是否包含）；Action 所在资产是否被 Experience/ActionSet 引用；bClientComponent / bServerComponent 是否勾对；目标 ActorClass 是否正确（PlayerState vs Pawn vs Controller vs GameState）。

---

## 11. 设计意图（为什么这样拆）

### 11.1 为什么拆成 ShooterCore / ShooterMaps / TopDownArena 三类 GameFeature

- ShooterCore = "玩法"：武器、技能、HUD、模式组件、Bot、勋章、C++（Aim Assist、Processor）。它不含正式地图（只有 L_ShooterGym / L_ShooterPerf 两个开发测试图）。
- ShooterMaps = "关卡内容"：几张地图 + 大量网格/材质 + 两个 playlist。依赖 ShooterCore（.uplugin 声明），反向不依赖。
- TopDownArena = "另一款游戏"：不依赖 ShooterCore，不使用装备/背包/武器组件，有自己的 Pawn、属性集、相机、阶段、HUD，证明核心框架（LyraGame）与具体玩法解耦。
- 具体场景：
  1. 美术/关卡团队只改地图：他们只动 ShooterMaps（体积大、二进制资产多、冲突多），不会碰 ShooterCore 的蓝图，减少二进制文件锁冲突；程序/设计改 ShooterCore 时关卡组不受阻。
  2. 按需下载 / DLC：新增赛季地图包只是新 GFP + 新 playlist，不改主包。（GFP 设计目标；本项目未配置 Chunk / 下载，ShooterMaps / TopDownArena 里的 PrimaryAssetLabel 属预留，属推断。）
  3. 玩法实验隔离：TopDownArena 可整体从 uproject 移除而不影响射击玩法，反之亦然。
  4. 按构建配置裁剪：ShooterTests 的 C++ 模块 TargetConfigurationDenyList 含 Shipping，测试代码和测试地图不进正式包。
  5. 编译边界：ShooterCoreRuntime 单独成模块，改它只重编该 DLL；但它依赖 LyraGame，LyraGame 头文件改动仍会级联。
- 代价：跨插件引用必须单向（Maps -> Core -> Game）。ShooterCore 硬引用 ShooterMaps 内容会形成循环依赖；ShooterCore 引用 /Game（IMC_Default、B_PickRandomCharacter、UI）是允许的，但 /Game 不应引用 GFP 内容（核心必须能在 GFP 缺失时运行）。

### 11.2 为什么 ActionSet 在多个 Experience 之间共享

- Elimination、Control、Perf、ShooterExplorer、ShooterTests 都需要同一套输入、HUD、标准组件和勋章。做成 LAS_ShooterGame_SharedInput / StandardComponents / StandardHUD 与 EAS_BasicShooterAcolades，各 Experience 只需列出 4 个引用，再叠加自己的差异 Actions（计分、bot、队伍、模式 HUD）。
- 场景：
  1. 给 HUD 新增一个 Widget（例如新的准星宿主）：改一个 LAS_ShooterGame_StandardHUD，5 个模式同时生效。
  2. 做新模式（FFA）：复制 Elimination，只改队伍/计分，不会漏掉 QuickBar、Nameplate、Accolade 等标配。
  3. 测试模式（ShooterTests）复用同样的 ActionSet，保证测试环境与线上模式 HUD / 输入一致，测试结果有代表性。
- 代价：隐式耦合，改动影响面大，回归要覆盖所有引用它的 Experience。本项目 B_TestInventoryExperience 把占点 HUD 的两个 Widget 直接复制进自己的 Actions，是"复制粘贴替代共享"的反例（后续同步成本高）。ActionSet 资产还必须放在 GFD 扫描的目录里（Experiences、Accolades）。

### 11.3 为什么 UserFacingExperience 与 Experience 分离

- Experience 是"运行时玩法配方"（服务器和客户端都要加载，包含 Action / PawnData）；UserFacingExperience 是"菜单与会话入口"（地图 + 用哪个 Experience + 展示信息 + 最大人数 + 额外 URL 参数 + 加载界面 + 是否录回放）。
- 场景：
  1. 同一 Experience 多张地图：B_ShooterGame_Elimination 可被 DA_Expanse_TDM 以及将来任何 DA_xxx_TDM 复用，只换 MapID / 图标 / 文案。反过来，B_LyraShooterGame_ControlPoints 同时被 DA_Convolution_ControlPoint（16 人）与 DA_ShooterGame_ShooterGym（8 人）使用。
  2. 同一地图多种玩法：新增一个 playlist 指向另一个 Experience 即可。
  3. 前端只加载轻量数据：菜单遍历 ULyraUserFacingExperienceDefinition（非蓝图类 PrimaryDataAsset），不必把整个 Experience 蓝图及其引用的 Widget / 技能拉进内存；点击后才由 ?Experience= 触发服务器加载。
  4. 专用服务器 / 快速开始：bIsDefaultExperience 决定无参数启动或 Quick Play 的默认玩法（LyraGameMode.cpp 约 221-243 行），与 Experience 本身无关。
  5. 灰度/开关：隐藏 playlist（bShowInFrontEnd=false）即可下线入口，玩法资产不动。
  6. 调试：URL ?Experience= 与命令行 -Experience= 可绕过 playlist 直接指定 Experience，说明二者数据层确实独立。
- 代价：多一层间接，新同事容易找不到"某个菜单项到底跑哪个 Experience"（需追 ExperienceID）；ExtraArgs、MaxPlayerCount 分散在 playlist 而不是 Experience 里。

### 11.4 这套布局的整体得失

- 得：
  - 玩法是数据，增删模式不改引擎 / 核心 C++（TopDownArena 证明可以整体换一个游戏）。
  - 内容按职责分包，多人并行、可裁剪、可单独激活（Registered + 按需 Active，内存只为当前模式付费）。
  - 服务器 / 客户端可在 Action 级别区分（bClientComponent / bServerComponent），天然适合多人。
  - 测试用 Experience（ShooterTests）可以无前端地复现模式环境。
- 失：
  - 调试门槛高：一个行为可能由 GFD、Experience、ActionSet、PawnData、AbilitySet、C++ 多处共同决定，需要看第 9 节的资产引用链才能定位。
  - 蓝图逻辑集中在 B_*Scoring 等 GameStateComponent 里，文本工具难读，二进制 diff 不可读，代码审查困难。
  - 硬引用与加载时序风险：Action 里直接引用的类会同步加载；GFP 状态机（Registered -> Loaded -> Active）出错时表现为"组件没挂上"而非明确报错。
  - 目录约定很重要（System/Experiences、System/Playlists、Experiences、Accolades）：目录放错 = 资产扫描不到。
