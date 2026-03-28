# 迫降月球 Moon Lander

<img width="1920" height="872" alt="image" src="https://github.com/user-attachments/assets/b4824954-bec2-4748-94c0-964377b6fd2b" />


## 项目简介

《迫降月球》是一款基于 Unity 2D 开发的月球着陆模拟游戏。玩家需要控制登月舱，在有限燃料的条件下完成降落，通过调整推力、旋转角度与下降速度，最终安全降落在指定平台。

项目重点在于“真实物理控制 + 多条件判定 + 动态反馈系统”的实现，包含：

- 推力与旋转控制
- 燃料管理
- 着陆角度与速度判定
- 动态得分系统
- 多关卡切换
- 镜头缩放与跟随
- 粒子、音效与 UI 反馈

---

## 项目预览

### 游戏演示
![ezgif-728f92d468123822](https://github.com/user-attachments/assets/dfc10a29-96bf-4672-a0d4-0a46977cb104)

![ezgif-7f6e0bf4342ed1b5](https://github.com/user-attachments/assets/b47629a5-6dea-436c-b669-d8a2b4955745)


---

## 核心玩法

玩家需要控制飞船在重力影响下平稳着陆。

### 操作方式

| 按键 | 功能 |
| --- | --- |
| W / ↑ | 向上喷射 |
| A / ← | 左旋转 |
| D / → | 右旋转 |
| ESC | 暂停游戏 |

### 成功着陆条件

必须同时满足：

- 落在正确的着陆平台
- 降落速度不能过快
- 飞船角度必须足够平稳

否则将直接坠毁。

---

## 功能特性

### 1. 真实物理飞船控制

项目使用 `Rigidbody2D` 实现真实的飞船运动。

- 上推力控制升力
- 左右控制飞船旋转
- 持续受到重力影响
- 玩家开始操作后才正式进入游戏状态

```csharp
if (GameInput.Instance.IsUpActionPressed()) {
    float force = 700f;
    landerRigidbody2D.AddForce(force * transform.up * Time.deltaTime);
}

if (GameInput.Instance.IsLeftActionPressed()) {
    landerRigidbody2D.AddTorque(+100f * Time.deltaTime);
}
```

---

### 2. 状态机控制

飞船包含三个状态：

```text
WaitingToStart -> Normal -> GameOver
```

- `WaitingToStart`：等待玩家开始
- `Normal`：正常飞行
- `GameOver`：成功或失败后结束

```csharp
public enum State {
    WaitingToStart,
    Normal,
    GameOver,
}
```

这样可以避免不同阶段逻辑混乱，使代码更容易维护。

---

### 3. 多条件着陆判定

着陆时会根据平台、速度与角度综合判断是否成功。

```csharp
float relativeVelocityMagnitude = collision2D.relativeVelocity.magnitude;
if (relativeVelocityMagnitude > 4f) {
    // 速度过快，坠毁
}

float dotVector = Vector2.Dot(Vector2.up, transform.up);
if (dotVector < .90f) {
    // 倾斜角度过大，坠毁
}
```

判定结果包括：

- 成功降落
- 未落在平台
- 速度过快
- 角度过大

---

### 4. 动态评分系统

项目不仅判断成功与失败，还会根据降落表现计算得分。

- 降落越慢，得分越高
- 飞船越垂直，得分越高
- 不同平台拥有不同倍率

```csharp
float landingAngleScore = ...
float landingSpeedScore = ...

int score = Mathf.RoundToInt(
    (landingAngleScore + landingSpeedScore) * landingPad.GetScoreMultiplier()
);
```

---

### 5. 燃料系统

飞船每次喷射都会消耗燃料。

- 喷射时持续扣除燃料
- 碰到补给后恢复燃料
- 燃料耗尽后无法继续控制飞船

```csharp
private void ConsumeFuel() {
    float fuelConsumptionAmount = 1f;
    fuelAmount -= fuelConsumptionAmount * Time.deltaTime;
}
```

---

### 6. 道具系统

场景中包含两种拾取物：

| 道具 | 作用 |
| --- | --- |
| FuelPickup | 恢复燃料 |
| CoinPickup | 增加额外分数 |

玩家需要在保证安全降落的同时，规划路线获取更高分数。

---

### 7. 镜头与反馈系统

项目使用 Cinemachine 实现镜头动态缩放与跟随。

- 游戏开始时镜头拉远展示地图
- 玩家开始操作后镜头自动跟随飞船
- 根据不同情况切换镜头大小

```csharp
cinemachineCamera.Target.TrackingTarget = Lander.Instance.transform;
CinemachineCameraZoom2D.Instance.SetNormalOrthographicSize();
```

同时项目还加入了：

- 推进器粒子效果
- 飞船爆炸效果
- 喷射音效
- 成功降落音效
- 坠毁音效

<img width="1920" height="872" alt="image" src="https://github.com/user-attachments/assets/792fb53f-9056-4237-8807-e62ab208deb6" />

![ezgif-879cd2e2bac77cbb](https://github.com/user-attachments/assets/296f283b-36e2-4e21-b0c0-2b6e3e2746c5)



---

## 项目结构

```text
Scripts
├── Core
│   ├── GameManager.cs
│   ├── GameInput.cs
│   └── SceneLoader.cs
├── Player
│   ├── Lander.cs
│   ├── LanderVisuals.cs
│   └── LanderAudio.cs
├── Level
│   ├── GameLevel.cs
│   ├── LandingPad.cs
│   └── LandingPadVisual.cs
├── UI
│   ├── MainMenuUI.cs
│   ├── StatsUI.cs
│   ├── PausedUI.cs
│   ├── LandedUI.cs
│   └── GameOverUI.cs
├── Audio
│   ├── SoundManager.cs
│   └── MusicManager.cs
└── Pickup
    ├── FuelPickup.cs
    └── CoinPickup.cs
```

---

## 技术亮点

### 事件驱动架构

项目中的多个系统并不是直接耦合，而是通过事件进行通信。

例如：

- 飞船喷射 → 播放喷射音效
- 飞船喷射 → 开启粒子特效
- 飞船降落 → 弹出结算 UI
- 拾取道具 → 更新燃料与得分

```csharp
public event EventHandler OnFuelPickup;
public event EventHandler OnCoinPickup;
public event EventHandler<OnLandedEventArgs> OnLanded;
```

这样做能够让代码结构更清晰，也更符合正式项目开发方式。

---

### 单例管理器

项目中使用多个单例统一管理全局逻辑：

- `GameManager`
- `SoundManager`
- `MusicManager`
- `GameInput`

---

### 模块化设计

飞船相关逻辑被拆分成多个脚本：

```text
Lander.cs          飞船控制与状态
LanderVisuals.cs   粒子与爆炸效果
LanderAudio.cs     喷射音效
```

相比将所有内容写在一个脚本中，可维护性更高，也更便于后续扩展。

---

## 可扩展方向

后续可以继续扩展以下内容：

- 随机地图与多种地形
- 多种登月舱
- 飞船升级系统
- 风力、低重力等特殊环境
- 排行榜与成绩保存
- 教学模式与新手引导
- 更丰富的 UI 与动画效果

---

## 运行方式

```bash
git clone https://github.com/你的用户名/Moon-Lander.git
```

使用 Unity 打开项目后：

1. 打开 `MainMenuScene`
2. 点击 Play
3. 开始游戏

---


