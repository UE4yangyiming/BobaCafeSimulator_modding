# 🃏 《奶茶店模拟器 - 重生之我在冰堡甜城当店长》 Modding 示例 (BobaCafeSimulator Modding Example)

_这是一个使用 **Lua 语言** 编写的 Mod 示例，适用于《卡牌店模拟器 多人联机版》。_  
[中文](README.md)   | [English](README_EN.md)  

---

## 🧩 工作原理概述

游戏会自动扫描并读取以下位置的 Mod：

- `游戏根目录/BobaCafeSimulator/Mods` 📁  
- 从 **Steam 创意工坊** 订阅的物品文件夹 🛠️

当找到满足条件的文件：`main.lua` 与 `preview.png`，即可在 **Mods** 菜单中识别、管理并加载该 Mod。

---

### ⚙️ 规则一：加载与执行
- 进入游戏约 **1 秒** 后，按 Mod 路径顺序加载并依次执行：  
  ```lua
  M.OnInit()   -- 初始化时执行一次
  M.OnTick(dt) -- 每帧执行
  ```

### 🧠 规则二：全局访问
- `UE`：全局变量，可访问 Unreal Engine 暴露的函数集合。  
- `M`：当前 Mod 的信息结构（会在主界面 Mods 列表中显示）。
- `dir`：当前 Mod 的绝对路径。
---

## 📁 Mod 文件夹结构

将 Mod 放入 `游戏根目录/BobaCafeSimulator/Mods/` 目录即可在游戏内识别。

```
BobaCafeSimulator/
└── Mods/
    └── MyMod/   --下面全都不能是中文
        ├── main.lua       # Mod 逻辑（Lua 编写）
        └── preview.png    # 预览图（256×256，正方形）
```

👉 [示例 Mod ](Example_ZH/)

---

## 🧾 `main.lua` 的 `M` 结构

`local M = {}` 建议包含：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | Mod 唯一 ID（英文，作为 Key） |
| `name` | string | 显示名称 |
| `description` | string | 描述 |
| `version` | string | 版本号 |
| `author` | string | 作者 |

> ✅ 你可以在 `M` 旁自由声明本地状态/变量，供 Mod 内部使用。

---

## 🖼️ 奶茶配方添加/替换（示例）

> 💡 ** ID 规则**：建议 `5200–5999`，
> 目前只能只用已有的配方进行组合。
> 咖啡不支持加入其他饮品，只能加入固体小料。
---

## ✅ 完整可运行示例：添加一个配方南瓜橙橙（`main.lua`）

```lua
-- 必填信息：会显示在 Mods 界面
local M = {
    id          = "NewDrinkPumpkinJuice",
    name        = "增加配方南瓜橙橙",
    description = "增加配方南瓜橙橙",
    version     = "1.0.0",
    author      = "yiming",

}

local function add_new_drink()
    local R = UE.UBoBaFunction.GetDrinkRegistryWS(MOD.GAA.WorldUtils:GetCurrentWorld())
    if not R then
        if MOD and MOD.Logger then MOD.Logger.LogScreen("找不到 UDrinkRegistryWorldSubsystem", 5,1,0,0,1) end
        return
    end
    -- 1) 注册饮品数据（覆盖层优先）
    local D = UE.FDrinkData()
    D.ID = 5200  --需要>5200<5999
    --名称
    D.DisplayName = "南瓜橙橙"
    --图片路径
    D.ImagePath = dir .. "5200.png" --你的Mod目录的图片

    -- 价格（S/M/L）
    D.Value:Add("S", 8.0)
    D.Value:Add("M", 10.0)
    D.Value:Add("L", 12.0)
    -- 配方里完成需要的水类型
    D.DrinkWaterFName = "Drink.PumpkinOrange"  --新的液体类型
    -- 配方里完成需要的物品 四个橙子片
    D.NeedItemID:Add(1103)
    D.NeedItemID:Add(1103)
    D.NeedItemID:Add(1103)
    D.NeedItemID:Add(1103)

    -- 客户会点的甜度
    D.CanSweet = {}
    D.CanSweet:Add("Sweet10")
    D.CanSweet:Add("Sweet7")
    D.CanSweet:Add("Sweet5")
    D.CanSweet:Add("Sweet3")
    D.CanSweet:Add("Sweet0")

    -- 客户会点的温度
    D.CanTemperature = {}
    --D.CanTemperature:Add("Hot")  --Demo目前不支持做除了咖啡的其他热饮
    D.CanTemperature:Add("Normal")
    D.CanTemperature:Add("SmallIce")
    D.CanTemperature:Add("Ice")

    -- 完美需求 -----------------------------------------------------------
    --（需要南瓜汁>0.83-1.00）
    local PN = UE.FPerfectNeed()
    PN.WaterName  = "Drink.PumpkinJuice"
    PN.MinPercent = 0.83
    PN.MaxPercent = 1.00
    D.PerfectNeed:Add(PN)
    -- 完美配方需求物品  橙子片4个
    D.PerfectNeedItem:Add("1103",4)
    ------------------------------------------------------------------

    --提示图标 配方栏的那个小图标
    D.ShowTutorialsItemID:Add(1106) --榨汁机
    D.ShowTutorialsItemID:Add(1033) --南瓜
    D.ShowTutorialsItemID:Add(1103) --橙子片
    --教程 配方界面点开后显示的教程
    D.MakeNeedTutorialText = "榨汁南瓜汁，然后加四个橙子片"

    -- 显示获取方式 配方界面点开后显示的解锁方式
    D.ShowGetWayText = "MOD获得"

    --获得配方之后解锁的物品类型 目前有 Watermelon(西瓜) Orange(橙子) Tea(茶) Pot(煮东西相关）
    --Milk(奶相关 大瓶一箱奶) Pumpkin(南瓜) Boba(珍珠) PaperCup(纸杯,热的需要) Coffee(奶茶)
    D.UnlockedItemID = {}
    D.UnlockedItemID:Add("Pumpkin")
    D.UnlockedItemID:Add("Orange")


    --两种液体混合模式（这个配方不需要）：
    -- 3) （其他）加“加液体后变化”的规则：纯净水 + 咖啡= 南瓜汁 （开玩笑的加法） 目前配方不需要
    ------------------------------------------------------------
    -- R:RegisterCupAddWaterRule(
    --     "Drink.PureWater",    -- CurrentType（杯中原液体）
    --     "Drink.Coffee", -- AddWaterType（加入的液体）
    --     "Drink.PumpkinJuice"  -- ToWaterType（结果）
    -- )

    ------------------------------------------------------------
    -- 4) “加物体后变化”的规则：南瓜汁 + 橙子片 = 南瓜橙橙
    ------------------------------------------------------------
    R:RegisterCupAddItemRule(
        "Drink.PumpkinJuice",      -- CurrentType（杯中原液体）南瓜汁
        "1103",         -- AddItemType（加入的小料类型）柠檬片
        "Drink.PumpkinOrange" --变成的液体类型- 南瓜橙橙
    )

    ------------------------------------------------------------
    -- 5) 饮品颜色
    ------------------------------------------------------------
    local S = UE.FDrinkStyle()
    S.DisplayName = "南瓜橙橙" --需要和配方名称一致
    -- 橙棕配色建议（可调）：亮 → 深
    S.Color1 = UE.FLinearColor(1.00, 0.58, 0.12, 1.0)  -- 明橙棕
    S.Color2 = UE.FLinearColor(1.00, 0.58, 0.12, 1.0)  -- 明橙棕
    R:RegisterDrinkStyle("Drink.PumpkinOrange", S) --填饮品的类型


    -- 注册（覆盖写入）系统
    R:RegisterDrinkData(D.ID, D)

    --直接增加到已经有的配方（不解锁）
    local GS = UE.UGameplayStatics.GetGameState(MOD.GAA.WorldUtils:GetCurrentWorld()) or nil  -- AGameStateBase*
    if GS then
        GS:EvAddDrink(D.ID)
    end

    if MOD and MOD.Logger then MOD.Logger.LogScreen("已注册：南瓜橙橙(5100)", 5,0,1,0,1) end --日志
end


function M.OnInit()
    --初始化
    if MOD and MOD.Logger then  MOD.Logger.LogScreen(("Mod [%s] 开始加载"):format(M.name), 5,1,1,0,1) end --日志
    add_new_drink()
end

function M.OnTick(dt)

end

return M
--附录 目前的液体表：
-- 纯净水	Drink.PureWater
-- 柠檬水	Drink.LemonWater
-- 糖浆 	Drink.Syrup
-- 西瓜汁	Drink.WatermelonJuice
-- 橙汁 	Drink.OrangeJuice
-- 榨橙汁	Drink.SqueezeOrangeJuice
-- 奶	    Drink.Milk
-- 绿茶	    Drink.GreenTea
-- 咖啡	    Drink.Coffee
-- 奶茶	    Drink.MilkTea
-- 柠檬绿茶	Drink.LemonGreenTea
-- 西瓜冰茶	Drink.WatermelonIcedTea
-- 西瓜果奶	Drink.WatermelonMilk
-- 香橙柠檬	Drink.OrangeLemon
-- 南瓜茶茶	Drink.PumpkinTea
-- 南瓜牛乳	Drink.PumpkinMilk
-- 南瓜汁	Drink.PumpkinJuice
-- 热水	    Drink.HotWater

--附录 目前的物品ID表：
--商店物品
-- 1001 一箱柠檬
-- 1002 一罐糖
-- 1014 杯子-小
-- 1003 杯子-中
-- 1015 杯子-大
-- 1004 一箱西瓜
-- 1005 一箱橙子
-- 1006 一箱珍珠
-- 1007 一排牛奶
-- 1008 一箱绿茶
-- 1009 一箱椰果
-- 1010 煮锅
-- 1013 猫粮
-- 1011 棒球棒
-- 1012 榨汁机
-- 1016 大罐糖浆
-- 1017 大瓶牛奶
-- 1018 高速榨汁机
-- 1019 急速榨汁机
-- 1020 小手推车
-- 1021 狼牙棒
-- 1022 桶装矿泉水
-- 1023 一箱牛奶
-- 1024 一箱大瓶牛奶
-- 1025 纸杯-小
-- 1026 纸杯-中
-- 1027 纸杯-大
-- 1028 研磨器
-- 1029 咖啡豆
-- 1030 咖啡冲泡机
-- 1033 南瓜
--判断固态物体：
-- 1102 柠檬片
-- 1103 橙子片
-- 1107 珍珠
--提示图标 配方栏的那个小图标
-- 1105 纯净水工具图片
-- 1106 榨汁机工具图片


```

---

## 📮 更多API接口以及扩展：联系方式
- QQ：780231813  
- 官方QQ群（联系群主）：722792074  
- Email：yangyiming780@foxmail.com  
- Steam 社区留言 / Git issues

---

## 🛡️ 社区准则（简要）
1. 🚫 禁止违法、政治敏感、色情、暴恐等内容。  
2. 🚫 禁止恶意侮辱、引战对立、影射现实人物的内容。  
3. 🚫 禁止未获授权使用受版权保护的资源。  
4. 🚫 禁止以 Mod 形式引导广告、募捐或付费。
   
若在 Steam 创意工坊发布且违反以上条目，可能被直接删除并封禁相关创作者权限。
