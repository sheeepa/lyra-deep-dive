# Lyra Starter Game 构建与配置文档

**Unreal Engine 版本**: 5.8  
**最后更新**: 2026-10-04

---

## 1. 模块与依赖

### 项目模块

| 模块名 | 类型 | 加载阶段 | 说明 |
|--------|------|--------|------|
| LyraGame | Runtime | Default | 主游戏模块，包含游戏逻辑与系统 |
| LyraEditor | Editor | Default | 编辑器相关功能与工具 |

### LyraGame 模块依赖

**公共依赖** (PublicDependencyModuleNames):
- Core, CoreOnline, CoreUObject, ApplicationCore, Engine
- PhysicsCore, GameplayTags, GameplayTasks, GameplayAbilities
- AIModule, ModularGameplay, ModularGameplayActors
- DataRegistry, ReplicationGraph, GameFeatures
- SignificanceManager, Hotfix, CommonLoadingScreen
- Niagara, AsyncMixin, ControlFlows, PropertyPath

**私有依赖** (PrivateDependencyModuleNames):
- InputCore, Slate, SlateCore, RenderCore, DeveloperSettings
- EnhancedInput, NetCore, RHI, Projects, Gauntlet
- UMG, CommonUI, CommonInput, GameSettings
- CommonGame, CommonUser, GameSubtitles
- GameplayMessageRuntime, AudioMixer, NetworkReplayStreaming
- UIExtension, ClientPilot, AudioModulation, EngineSettings
- DTLSHandlerComponent, Json, PlatformDLC
- ExternalRpcRegistry, HTTPServer (非Shipping)
- AutomationDriver (非Shipping且Engine非Installed)

**自定义定义**:
- `SHIPPING_DRAW_DEBUG_ERROR=1` - Shipping中DrawDebug会导致编译错误
- `WITH_RPC_REGISTRY=1` (非Shipping)
- `WITH_HTTPSERVER_LISTENERS=1` (非Shipping)
- `WITH_AUTOMATION_DRIVER=1` (非Shipping且Engine非Installed)

### LyraEditor 模块依赖

**公共依赖**:
- Core, CoreUObject, Engine, EditorFramework, UnrealEd
- PhysicsCore, GameplayTagsEditor, GameplayTasksEditor
- GameplayAbilities, GameplayAbilitiesEditor
- StudioTelemetry, LyraGame

**私有依赖**:
- InputCore, Slate, SlateCore, ToolMenus, EditorStyle
- DataValidation, MessageLog, Projects, DeveloperToolSettings
- CollectionManager, SourceControl, Chaos
- ExternalRpcRegistry, HTTPServer (非Shipping)

---

## 2. 构建目标变体

### 基础目标

| 目标 | 类型 | 说明 |
|-----|------|------|
| **LyraGame** | Game | 标准客户端构建，使用OnlineServicesNull |
| **LyraClient** | Client | 纯客户端构建 |
| **LyraServer** | Server | 服务器构建，仅限Server平台 |
| **LyraEditor** | Editor | 编辑器构建，启用RemoteSession插件 |

### EOS集成目标

| 目标 | 基类 | 启用插件 | 自定义配置 |
|-----|------|--------|----------|
| **LyraGameEOS** | LyraGameTarget | OnlineServicesEOS, OnlineSubsystemEOS | EOS |
| **LyraServerEOS** | LyraServerTarget | OnlineServicesEOS, OnlineSubsystemEOS | EOS |

### Steam集成目标

| 目标 | 基类 | 启用插件 | 自定义配置 |
|-----|------|--------|----------|
| **LyraGameSteam** | LyraGameTarget | (无额外) | Steam |
| **LyraServerSteam** | LyraServerTarget | (无额外) | Steam |

### Steam + EOS集成目标

| 目标 | 基类 | 启用插件 | 自定义配置 |
|-----|------|--------|----------|
| **LyraGameSteamEOS** | LyraGameTarget | OnlineServicesEOS, OnlineSubsystemEOS | SteamEOS |
| **LyraServerSteamEOS** | LyraServerTarget | OnlineServicesEOS, OnlineSubsystemEOS | SteamEOS |

### 共享目标设置 (ApplySharedLyraTargetSettings)

**所有配置**:
- `DefaultBuildSettings = BuildSettingsVersion.V7`
- `IncludeOrderVersion = EngineIncludeOrderVersion.Latest`

**Unique构建环境**:
- 启用 `ShadowVariableWarningLevel = Error`
- `bUseLoggingInShipping = true`
- `bTrackRHIResourceInfoForTest = true`

**Shipping + 非Server**:
- `bDisableUnverifiedCertificates = true` (验证HTTPS证书)

**Shipping 或 Test**:
- `bAllowGeneratedIniWhenCooked = false`
- `bAllowNonUFSIniWhenCooked = false`

**非Editor目标**:
- 禁用插件: `OpenImageDenoise`
- `UE_ASSETREGISTRY_INDIRECT_ASSETDATA_POINTERS=1`

**GameFeature插件配置**:
- 由 `ConfigureGameFeaturePlugins()` 根据 `ExplicitlyLoaded`, `EditorOnly`, `RestrictToBranch`, `NeverBuild` 等字段动态启用/禁用

---

## 3. 插件清单表

### 核心插件 (Enabled=true)

| 插件名 | 模块 | 类别 | 说明 |
|--------|------|------|------|
| ActorPalette | - | Editor | Actor选择调板 |
| AESGCMHandlerComponent | - | Networking | AES加密 |
| DTLSHandlerComponent | - | Networking | DTLS处理 |
| GameplayAbilities | GameplayAbilities | Gameplay | 能力系统 |
| Gauntlet | Gauntlet | Testing | 自动化测试框架 |
| CommonLoadingScreen | CommonLoadingScreen | Gameplay | 加载屏幕管理 |
| CommonStartupLoadingScreen | CommonStartupLoadingScreen | Gameplay | 启动加载屏幕 |
| CommonConversation | - | Gameplay | 对话系统 |
| GameFeatures | GameFeatures | Gameplay | 游戏功能系统 |
| ModularGameplay | ModularGameplay | Gameplay | 模块化游戏框架 |
| ModularGameplayActors | ModularGameplayActors | Gameplay | 模块化Actor |
| EnhancedInput | EnhancedInput | Input | 增强输入系统 |
| Volumetrics | - | Rendering | 体积渲染 |
| DataRegistry | DataRegistry | Data | 数据注册表 |
| ReplicationGraph | ReplicationGraph | Networking | 复制图优化 |
| SignificanceManager | SignificanceManager | Gameplay | 对象重要性管理 |
| Niagara | Niagara | Effects | 粒子系统 |
| Water | - | Rendering | 水体系统 |
| CommonUI | CommonUI | UI | 通用UI框架 |
| ControlFlows | ControlFlows | Gameplay | 控制流系统 |
| GameSettings | GameSettings | UI | 游戏设置UI |
| CommonUser | CommonUser | Gameplay | 用户管理 |
| CommonGame | CommonGame | Gameplay | 通用游戏逻辑 |
| GameSubtitles | GameSubtitles | Gameplay | 字幕系统 |
| PocketWorlds | PocketWorlds | Gameplay | 小世界支持 |
| UIExtension | UIExtension | UI | UI扩展框架 |
| AsyncMixin | AsyncMixin | Core | 异步操作混合 |
| Metasound | Metasound | Audio | 元声音合成 |
| OnlineFramework | - | Online | 在线框架 |
| PlayFabParty | PlayFabParty | Online | PlayFab Party (XB1/XSX/WinGDK) |
| OnlineServicesNull | OnlineServicesNull | Online | 空在线服务实现 |
| OnlineServicesOSSAdapter | OnlineServicesOSSAdapter | Online | OSS适配器 |
| OnlineSubsystemSteam | OnlineSubsystemSteam | Online | Steam在线子系统 |
| SocketSubsystemSteamIP | SocketSubsystemSteamIP | Networking | Steam IP套接字 |
| GameplayMessageRouter | GameplayMessageRouter | Gameplay | 游戏消息路由 |
| SteamSockets | SteamSockets | Networking | Steam套接字 |
| AssetReferenceRestrictions | - | Asset | 资源引用限制 |
| ModelingToolsEditorMode | - | Editor | 建模工具 |
| GeometryScripting | GeometryScripting | Scripting | 几何脚本 |
| AnimationLocomotionLibrary | AnimationLocomotionLibrary | Animation | 动画移动库 |
| AudioModulation | AudioModulation | Audio | 音频调制 |
| AudioGameplayVolume | AudioGameplayVolume | Audio | 音频体积 |
| AudioGameplay | AudioGameplay | Audio | 音频游戏系统 |
| SoundUtilities | - | Audio | 声音工具 |
| AnimationWarping | AnimationWarping | Animation | 动画弯曲 |
| MovieRenderPipeline | - | Editor Only | 电影渲染管道 |
| MoviePipelineMaskRenderPass | - | Editor Only | 渲染通道掩码 |
| AssetSearch | - | Asset | 资源搜索 |
| GameplayInsights | - | Insights | 游戏性能分析 |
| Spatialization | - | Audio | 空间音效 |
| ShooterCore | ShooterCoreRuntime | Game Feature | 射击核心功能包 |
| ShooterMaps | - | Game Feature | 射击地图包 |
| TopDownArena | - | Game Feature | 俯视竞技场功能包 |
| FunctionalTestingEditor | - | Editor | 功能测试编辑器 |
| ShooterExplorer | - | Game Feature | 射击探索器 |
| ShooterTests | - | Game Feature | 射击测试包 |
| GameplayInteractions | - | Gameplay | 游戏互动 |
| SmartObjects | SmartObjects | AI | 智能对象 |
| ContextualAnimation | ContextualAnimation | Animation | 情景动画 |
| GameplayBehaviorSmartObjects | - | AI | 行为智能对象 |
| GameplayStateTree | GameplayStateTree | Gameplay | 游戏状态树 |
| GameplayBehaviors | - | Gameplay | 游戏行为 |
| RuntimeTests | RuntimeTests | Testing | 运行时测试 |
| AutomatedPerfTesting | - | Testing | 自动性能测试 |
| Reflex | - | Rendering | NVIDIA Reflex |
| XInputDevice | - | Input (Win64/WinGDK) | XInput设备 |
| GameInputWindows | - | Input (Win64/WinGDK) | Windows GameInput |
| AIAssistant | - | Tools | AI助手 |
| ToolsetRegistry | ToolsetRegistry | Tools | 工具集注册表 |
| AllToolsets | - | Tools | 所有工具集 |
| PlayerInputDebugger | - | Editor Only | 玩家输入调试器 |
| PlatformDLC | - | Deployment | 平台DLC |
| ModelContextProtocol | - | Tools | MCP协议支持 |

### 禁用的插件

| 插件名 | 说明 |
|--------|------|
| MagicLeap | Magic Leap XR支持 |
| MagicLeapMedia | Magic Leap媒体 |
| MagicLeapPassableWorld | Magic Leap通道世界 |
| OpenXREyeTracker | OpenXR眼睛追踪 |
| OpenXRHandTracking | OpenXR手部追踪 |
| OpenXRHMD | OpenXR HMD |
| SteamVR | SteamVR支持 |
| GearVR | Samsung Gear VR |
| MLSDK | Magic Leap SDK |
| ResonanceAudio | Google Resonance音频 |
| RuntimePhysXCooking | PhysX Cooking |

### 游戏功能插件 (GameFeatures/)

所有GameFeature插件设置:
- `ExplicitlyLoaded: true` - 必须在项目启动后显式加载
- `EnabledByDefault: false` - 构建时默认禁用
- `BuiltInInitialFeatureState: "Registered"` - 初始状态

| 插件名 | 模块 | 依赖插件 |
|--------|------|--------|
| **ShooterCore** | ShooterCoreRuntime | GameplayAbilities, ModularGameplay, GameplayMessageRouter, AsyncMixin, CommonUI, CommonGame, GameSubtitles, EnhancedInput, LyraExampleContent |
| **ShooterMaps** | - | ShooterCore |
| **ShooterExplorer** | - | ShooterCore |
| **ShooterTests** | - | ShooterCore |
| **TopDownArena** | - | (需查实) |

---

## 4. 配置文件要点

### DefaultGame.ini

**[/Script/EngineSettings.GeneralProjectSettings]**
```
ProjectID=F767DCEB4F32E6A743A92AAFB109EB4C
ProjectName=Lyra
Description=Sample starter game for Unreal Engine 5
```

**[/Script/EngineSettings.GameMapsSettings]**（注意：实际位于 `Config/DefaultEngine.ini:51-54`，不在 DefaultGame.ini；DefaultGame.ini 只含 GeneralProjectSettings 等段）
```
GlobalDefaultGameMode=/Game/B_LyraGameMode.B_LyraGameMode_C
GameInstanceClass=/Game/B_LyraGameInstance.B_LyraGameInstance_C
GameDefaultMap=/Game/System/FrontEnd/Maps/L_LyraFrontEnd.L_LyraFrontEnd
EditorStartupMap=/Game/System/DefaultEditorMap/L_DefaultEditorOverview.L_DefaultEditorOverview
```

**[/Script/LyraGame.LyraAssetManager]**
```
LyraGameDataPath=/Game/DefaultGameData.DefaultGameData
DefaultPawnData=/Game/Characters/Heroes/EmptyPawnData/DefaultPawnData_EmptyPawn.DefaultPawnData_EmptyPawn
```

**[/Script/Engine.AssetManagerSettings]**
- **PrimaryAssetTypesToScan** (8项 `+` 条目，另有 2 条 `-` 移除引擎默认项):
  - `Map`: `/Game/Maps`, AlwaysCook, 包含前端和编辑器默认地图
  - `LyraGameData`: 默认游戏数据资源, AlwaysCook
  - `LyraExperienceDefinition`: `/Game/System/Experiences`, AlwaysCook
  - `LyraUserFacingExperienceDefinition`: UI Temp和Playlists目录, AlwaysCook
  - `LyraLobbyBackground`: 大厅背景
  - `LyraExperienceActionSet`: 体验动作集
  - `GameFeatureData`: `/Game/Unused`, AlwaysCook

**[/Script/GameplayAbilities.AbilitySystemGlobals]**
```
AbilitySystemGlobalsClassName=/Script/LyraGame.LyraAbilitySystemGlobals
GlobalGameplayCueManagerClass=/Script/LyraGame.LyraGameplayCueManager
+GameplayCueNotifyPaths=/Game/GameplayCueNotifies
+GameplayCueNotifyPaths=/Game/GameplayCues
ActivateFailCooldownTag=(TagName="Ability.ActivateFail.Cooldown")
```

**[/Script/GameFeatures.GameFeaturesSubsystemSettings]**
```
GameFeaturesManagerClassName=/Script/LyraGame.LyraGameFeaturePolicy
```

**[/Script/LyraGame.LyraUIManagerSubsystem]**
```
DefaultUIPolicyClass=/Game/UI/B_LyraUIPolicy.B_LyraUIPolicy_C
```

**[/Script/CommonLoadingScreen.CommonLoadingScreenSettings]**
```
LoadingScreenWidget=/Game/UI/Foundation/LoadingScreen/W_LoadingScreen_Host.W_LoadingScreen_Host_C
```

**[/Script/CommonUI.CommonUISettings]**
```
DefaultThrobberMaterial=/Game/UI/Foundation/Materials/M_UI_Throbber_Base.M_UI_Throbber_Base
DefaultRichTextDataClass=/Game/UI/Foundation/RichTextData/CommonUIRichTextData.CommonUIRichTextData_C
```

**打包设置 [/Script/UnrealEd.ProjectPackagingSettings]**
```
BuildConfiguration=PPBC_Development
BuildTarget=LyraGame
UsePakFile=True
bGenerateChunks=true
bCompressed=True
PackageCompressionFormat=Oodle
PackageCompressionMethod=Kraken
InternationalizationPreset=All
CulturesToStage: en, ar, es, es-419, fr, it, ja, pl, pt-BR, ru, tr, zh-Hans, ko, de
```

### DefaultEngine.ini

**[/Script/Engine.Engine]**
```
GameEngine=/Script/LyraGame.LyraGameEngine
UnrealEdEngine=/Script/LyraEditor.LyraEditorEngine
EditorEngine=/Script/LyraEditor.LyraEditorEngine
GameViewportClientClassName=/Script/LyraGame.LyraGameViewportClient
AssetManagerClassName=/Script/LyraGame.LyraAssetManager
WorldSettingsClassName=/Script/LyraGame.LyraWorldSettings
LocalPlayerClassName=/Script/LyraGame.LyraLocalPlayer
GameUserSettingsClassName=/Script/LyraGame.LyraSettingsLocal
```

**[/Script/Engine.RendererSettings]** (部分关键设置):
```
r.SkinCache.CompileShaders=True
r.DefaultFeature.AutoExposure.ExtendDefaultLuminanceRange=True
r.VirtualTextures=True
r.CustomDepth=3
r.GenerateMeshDistanceFields=True
r.AntiAliasingMethod=4
r.RayTracing=True
r.Lumen.HardwareRayTracing=True
r.Substrate=1
```

**[/Script/Engine.CollisionProfile]**
- 5个自定义TraceChannel (ECC_GameTraceChannel1-5)
- 8个自定义CollisionProfile，见下表

**[/Script/Engine.PhysicsSettings]**
```
+PhysicalSurfaces=(Type=SurfaceType1,Name="Character")
+PhysicalSurfaces=(Type=SurfaceType2,Name="Concrete")
+PhysicalSurfaces=(Type=SurfaceType3,Name="Glass")
DefaultDegreesOfFreedom=Full3D
```

**[OnlineServices]**
```
DefaultServices=Null
```

**[/Script/Engine.AudioSettings]**
```
DefaultSoundClassName=/Game/Audio/Classes/Overall.Overall
DefaultSoundConcurrencyName=/Game/Audio/Concurrency/SCon_Default.SCon_Default
MasterSubmix=/Game/Audio/Submixes/MainSubmix.MainSubmix
VoiPSoundClass=/Game/Audio/Classes/VoiceChat.VoiceChat
```

**[/Script/WindowsTargetPlatform.WindowsTargetSettings]**
```
DefaultGraphicsRHI=DefaultGraphicsRHI_DX12
+D3D12TargetedShaderFormats=PCD3D_SM6
+VulkanTargetedShaderFormats=SF_VULKAN_SM6
AudioSampleRate=48000
```

### DefaultInput.ini

**[/Script/Engine.PlayerInput]**
```
DefaultPlayerInputClass=/Script/LyraGame.LyraPlayerInput
DefaultInputComponentClass=/Script/LyraGame.LyraInputComponent
DefaultPreferredInputAPIList=XInput,WinDualShock
```

**[/Script/EnhancedInput.EnhancedInputDeveloperSettings]**
```
bEnableUserSettings=True
UserSettingsClass=/Script/LyraGame.LyraInputUserSettings
DefaultPlayerMappableKeyProfileClass=/Script/LyraGame.LyraPlayerMappableKeyProfile
```

**[/Script/CommonUI.CommonUIInputSettings]**
```
bLinkCursorToGamepadFocus=True
UIActionProcessingPriority=10000
+InputActions=(ActionTag=UI.Action.Escape,KeyMappings=((Key=Escape),(Key=Gamepad_Special_Right)))
```

### DefaultGameplayTags.ini

**[/Script/GameplayTags.GameplayTagsSettings]**
```
ImportTagsFromConfig=True
WarnOnInvalidTags=True
FastReplication=True
+GameplayTagTableList=/Game/ContextEffects/DT_AnimEffectTags.DT_AnimEffectTags
+GameplayTagTableList=/Game/ContextEffects/DT_SurfaceTypes.DT_SurfaceTypes
```

**主要Tag分类** (样本):
- `Ability.Type.Action.*` (ADS, Dash, Drop, Emote, Grenade, Jump, Melee, Reload, WeaponFire)
- `Ability.Type.Passive.*` (AutoReload, AutoRespawn, ChangeQuickbarSlot)
- `Ability.Type.StatusChange.*` (Death, Spawning)
- `GameplayCue.Character.*` (DamageTaken, Dash, Death, Heal, Melee)
- `GameplayCue.Weapon.*` (Fire, Impact, Grenade, etc.)
- `GameplayEffect.DamageType.*` (Basic, Grenade, Melee, Pistol, Rifle, Shotgun)
- `InputTag.Ability.*` / `InputTag.Weapon.*` (输入映射)
- `Platform.Trait.*` (输入、音频、功能特征)
- `HUD.Slot.*` (UI布局插槽)

### 自定义配置目录

**Config/Custom/EOS/DefaultEngine.ini** 和 **DefaultGame.ini**
- EOS目标使用此配置覆盖

**Config/Custom/Steam/DefaultEngine.ini** 和 **DefaultGame.ini**
- Steam目标使用此配置覆盖

**Config/Custom/SteamEOS/DefaultEngine.ini**
- SteamEOS目标使用此配置覆盖

### 平台特定配置

| 平台目录 | 文件 | 说明 |
|---------|------|------|
| **Windows/** | WindowsEngine.ini, WindowsGame.ini | 桌面Windows配置 |
| **Android/** | AndroidEngine.ini, AndroidGame.ini, AndroidInput.ini, AndroidScalability.ini | Android平台 |
| **IOS/** | IOSEngine.ini, IOSGame.ini, IOSScalability.ini | iOS平台 |
| **Mac/** | MacEngine.ini, MacGame.ini | macOS平台 |
| **Linux/** | LinuxEngine.ini, LinuxGame.ini | Linux平台 |
| **TVOS/** | TVOSGame.ini | tvOS平台 |
| **Localization/** | Game_*和EngineOverrides_* | 本地化配置 |

---

## 5. 碰撞通道 / 物理表面 / 输入

### 碰撞通道 (Trace Channels)

| 通道 | ECC索引 | TraceType | 名称 | 用途 |
|-----|--------|-----------|------|------|
| ECC_GameTraceChannel1 | 1 | Yes | `Lyra_TraceChannel_Interaction` | 交互对象射线 |
| ECC_GameTraceChannel2 | 2 | Yes | `Lyra_TraceChannel_Weapon` | 武器射线 |
| ECC_GameTraceChannel3 | 3 | Yes | `Lyra_TraceChannel_Weapon_Capsule` | 武器胶囊体 |
| ECC_GameTraceChannel4 | 4 | Yes | `Lyra_TraceChannel_Weapon_Multi` | 武器多目标 |
| ECC_GameTraceChannel5 | 5 | Yes | `Lyra_TraceChannel_AimAssist` | 瞄准辅助 |

### 物理表面 (Physical Surfaces)

| 类型 | 名称 |
|-----|------|
| SurfaceType1 | Character |
| SurfaceType2 | Concrete |
| SurfaceType3 | Glass |

### 自定义碰撞配置文件 (Collision Profiles)

| 名称 | 启用 | ObjectType | 说明 |
|-----|------|-----------|------|
| **LyraPawnMesh** | QueryOnly | Pawn | 人物网格，忽略大部分碰撞，与Lyra_TraceChannel_Weapon_Multi重叠 |
| **LyraPawnCapsule** | QueryOnly | Pawn | 人物胶囊体，仅与Lyra_TraceChannel_Weapon_Capsule交互 |
| **Interactable_OverlapDynamic** | QueryOnly | PhysicsBody | 交互对象，与Lyra_TraceChannel_Interaction重叠 |
| **Interactable_BlockDynamic** | QueryAndPhysics | WorldDynamic | 交互物体，阻挡并与交互通道重叠 |
| **AimAssist_OverlapDynamic** | QueryOnly | PhysicsBody | 瞄准辅助区域，仅与Lyra_TraceChannel_AimAssist重叠 |

### 增强输入系统 (EnhancedInput)

**玩家输入类**:
- `DefaultPlayerInputClass=/Script/LyraGame.LyraPlayerInput`
- `DefaultInputComponentClass=/Script/LyraGame.LyraInputComponent`

**输入用户设置**:
- `UserSettingsClass=/Script/LyraGame.LyraInputUserSettings`
- `DefaultPlayerMappableKeyProfileClass=/Script/LyraGame.LyraPlayerMappableKeyProfile`

**控制器死区** (GamePad):
- Left/Right Stick: DeadZone=0.25, Sensitivity=1.0
- L/R Trigger: DeadZone=0.0, Sensitivity=1.0

**鼠标灵敏度**:
- MouseX/MouseY: DeadZone=0.0, Sensitivity=0.07

---

## 6. Content 目录地图

| 目录 | 主要内容 |
|-----|---------|
| **Audio/** | 音频资源：音效、音乐、SoundCue、Metasound资源 |
| **Characters/** | 角色资源：骨骼网格、动画蓝图、人物数据、动画集 |
| **ContextEffects/** | 情景效果：DT_AnimEffectTags、DT_SurfaceTypes |
| **Effects/** | 特效：粒子系统、Niagara效果、视觉反馈 |
| **Environments/** | 环境资源：地形、建筑、道具、天空盒 |
| **Feedback/** | 反馈资源：力反馈、屏幕振动、特效触发 |
| **GameplayCueNotifies/** | GameplayCue通知 (动态生成的Cue播放器) |
| **GameplayEffects/** | 游戏效果资源：伤害、治疗、状态效果 |
| **Input/** | 增强输入数据：InputAction、InputMappingContext |
| **Legal/** | 法律文本、许可证、使用条款 |
| **Localization/** | 多语言翻译数据 |
| **PhysicsMaterials/** | 物理材料：表面摩擦力、反弹等 |
| **System/** | 系统资源：游戏实例、游戏模式、经验定义、默认数据 |
| **Tools/** | 工具和编辑器相关资源 |
| **UI/** | UI资源：通用UI、菜单、HUD、小部件、样式 |
| **Weapons/** | 武器资源：武器蓝图、射线武器数据、视觉效果 |

### System/ 子目录重点

| 子目录 | 用途 |
|-------|------|
| **System/Experiences/** | 体验定义 (LyraExperienceDefinition) |
| **System/Playlists/** | 播放列表与用户侧体验定义 |
| **System/FrontEnd/** | 前端菜单、主界面、地图 |
| **System/DefaultEditorMap/** | 编辑器默认场景 |

---

## 7. 修改菜谱 (How-To)

### 添加模块依赖

1. 编辑 `Source/LyraGame/LyraGame.Build.cs`
2. 在 `PublicDependencyModuleNames.AddRange()` 或 `PrivateDependencyModuleNames.AddRange()` 中添加模块名
3. 重新生成项目文件并编译

```csharp
PublicDependencyModuleNames.AddRange(
    new string[] {
        // 已有的模块...
        "MyNewModule"  // 新添加
    }
);
```

### 添加碰撞通道

1. 编辑 `Config/DefaultEngine.ini`
2. 在 `[/Script/Engine.CollisionProfile]` 下添加 `+DefaultChannelResponses` 条目
3. 编辑需要与此通道交互的碰撞配置文件，更新 `CustomResponses`

```ini
+DefaultChannelResponses=(Channel=ECC_GameTraceChannel6,DefaultResponse=ECR_Ignore,bTraceType=True,bStaticObject=False,Name="MyCustomTraceChannel")
```

### 更改默认地图/体验

**游戏默认地图**:
- 编辑 `Config/DefaultEngine.ini` 的 `[/Script/EngineSettings.GameMapsSettings]`
- 修改 `GameDefaultMap=/Game/System/FrontEnd/Maps/L_LyraFrontEnd.L_LyraFrontEnd`

**编辑器默认地图**:
- 修改 `EditorStartupMap=/Game/System/DefaultEditorMap/L_DefaultEditorOverview.L_DefaultEditorOverview`

**默认体验**:
- 编辑 `Config/DefaultGame.ini`
- 修改 `[/Script/LyraGame.LyraAssetManager]` 中的 `LyraGameDataPath`

### 添加新 Primary Asset Type

1. 编辑 `Config/DefaultGame.ini`
2. 在 `[/Script/Engine.AssetManagerSettings]` 下添加 `+PrimaryAssetTypesToScan` 条目

```ini
+PrimaryAssetTypesToScan=(PrimaryAssetType="MyAssetType",AssetBaseClass="/Script/MyModule.MyAssetClass",bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game/MyAssets")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
```

### 添加原生 Gameplay Tag

1. 在代码中定义标签（例如在 `LyraGameplayTags.h` 中）
2. 或编辑 `Config/DefaultGameplayTags.ini`
3. 在 `+GameplayTagList` 中添加条目

```ini
+GameplayTagList=(Tag="MyProject.MyTag",DevComment="My tag description")
```

### 添加 INI Tag (不依赖编译)

1. 编辑 `Config/DefaultGameplayTags.ini`
2. 确保 `ImportTagsFromConfig=True` (默认已启用)
3. 添加 `+GameplayTagList` 条目
4. 在编辑器中点击 "Refresh" 或重新启动

---

## 8. 注意事项

### 构建配置

1. **目标选择**: 
   - 客户端开发使用 `LyraGame` 或 `LyraClient`
   - 服务器开发使用 `LyraServer` 或 `LyraServerEOS`
   - EOS集成时使用对应的 `*EOS` 目标

2. **CustomConfig**:
   - EOS/Steam/SteamEOS 目标使用 `Config/Custom/{ConfigName}/` 中的配置
   - 这些配置覆盖基础 `DefaultEngine.ini` 和 `DefaultGame.ini`

3. **SharedTargetSettings**:
   - 所有非Editor目标共享 `ApplySharedLyraTargetSettings()`
   - 此函数配置GameFeature插件的启用/禁用逻辑

### 插件架构

1. **模块化**: CommonUI/CommonGame/GameSettings 等构成通用基础层
2. **游戏功能**: ShooterCore/TopDownArena/等通过GameFeatures系统动态加载
3. **限制**: GameFeature插件必须设置 `ExplicitlyLoaded=true` 和 `EnabledByDefault=false`

### 资源管理

1. **AssetManager**:
   - 配置的Primary Asset Type会被预扫描和烹饪
   - 关键资源（Map、LyraGameData、Experience）设置 `CookRule=AlwaysCook`

2. **GameplayCue**:
   - 两个搜索路径: `/Game/GameplayCueNotifies` 和 `/Game/GameplayCues`
   - 建议使用数据表 (DataTable) 存储Cue定义

### 网络配置

1. **在线服务**: 
   - 默认 `OnlineServices=Null` (单机/本地联网)
   - EOS/Steam 目标启用对应的在线子系统

2. **ReplicationGraph**:
   - 模块已依赖，但 `ULyraReplicationGraphSettings::bDisableReplicationGraph` 默认 `true`（LyraReplicationGraphSettings.h:24；LyraReplicationGraph.cpp:145 据此跳过），默认并不启用复制图；当前 DefaultGame.ini:232 也显式写了 `bDisableReplicationGraph=True`；要启用需在 `[/Script/LyraGame.LyraReplicationGraphSettings]`（DefaultGame.ini:231）设为 false
   - 配置在 `LyraGame.Build.cs` 的公共依赖中

3. **网络带宽**:
   - `[/Script/Engine.GameNetworkManager]` 设置:
     - `TotalNetBandwidth=200000`
     - `MaxDynamicBandwidth=40000`
     - `MinDynamicBandwidth=20000`

### 本地化

1. **多语言支持**: 14种语言已配置 (`CulturesToStage`: en, ar, es, es-419, fr, it, ja, pl, pt-BR, ru, tr, zh-Hans, ko, de)
2. **本地化配置**: 在 `Config/Localization/` 中维护导入/导出规则
3. **UIScaleRule**: 设置为 `ScaleToFit` 以支持不同屏幕分辨率

### 性能优化

1. **Nanite虚拟化**: `r.VirtualTextures=True`
2. **Lumen全局光照**: `r.Lumen.HardwareRayTracing=True`
3. **Substrate**: `r.Substrate=1` (新材料系统)
4. **LOD系统**: 启用 `r.GenerateMeshDistanceFields=True`

### 调试与开发

1. **RPC框架**: 非Shipping配置启用，Shipping时剥离以消除漏洞
2. **自动化驱动**: `WITH_AUTOMATION_DRIVER=1` (非Shipping且本地引擎)
3. **Debug绘制**: `SHIPPING_DRAW_DEBUG_ERROR=1` (Shipping中编译错误)

