# 村民大战Grox (Village vs Grox)

## 简介
一款基于 Python + Pygame 开发的塔防游戏。梗源于知名 MC 主播 **GroxMC**（"村民克星"），玩家需要合理布置村民防御塔，阻挡 Grox 军从左至右进攻村庄。包含 5 波预设 + 无尽模式、绿宝石经济系统、6 种功能各异的防御塔、3 种敌人类型，以及隐藏的 Grox Boss 战（暂未加入）。

## 开发主体
EndoveStudio 宇鸽工作室

## 开发者名单
| 名称 | 归属 | 职责 |
|------|------|------|
| PanBeiChen | EDS | 策划、程序、美术整合、测试 |

## 版本线
| 阶段 | 版本号 |
|------|--------|
| 上个版本 | *null* |
| **本版本** | **v1.26.11** |
| 下个版本 | v1.26.x（计划中） |

---

## 更新日志

### [v1.26.11] — 2026-11-18

#### 新增
- **完整塔防核心循环**：5 行 8 列网格地图，敌人从左向右推进，到达终点扣除村庄生命值
- **5 种防御塔**：
  - `Farmer` — 经济塔，定期产出绿宝石，不攻击
  - `Archer` — 远程射手，射程 180px，中等攻速
  - `Iron Golem` — 近战肉盾，高血量高伤害，范围 60px
  - `Cleric` — 治疗塔，定期回复范围内友方塔血量
  - `Nitwit` — 自爆 AOE，放置后接触敌人触发范围爆炸
- **3 种敌人 + Boss(未加入）**：
  - `Pillager` — 基础远程兵，速度快血量低
  - `Ravager` — 重型冲锋兵，高血量高伤害
  - `Vindicator` — 近战突击兵，攻速快
  - `Grox`（Boss）— 第 5 波登场，超高血量 + 群体冲锋（未加入）
- **绿宝石经济系统**：初始 200 绿宝石，每 3 秒 +25，击杀敌人掉落奖励
- **5 波预设 + 无尽轮回**：前 5 波固定配置，第 6 波起循环递增难度（每轮回血量/数量 ×1.3）
- **选卡 UI 面板**：底部 5 张塔卡片，鼠标点击或快捷键 1-5 选塔
- **HUD 系统**：实时显示绿宝石数量、村庄生命值、当前波次
- **飘字反馈**：伤害数字、绿宝石产出、破防提示等浮动文字
- **调试碰撞箱**（`DEBUG_HITBOX = True`）：绿色圈=碰撞体，蓝色=射程，橙色=爆炸范围，红色=近战范围
- **中文字体支持**：`simhei.ttf` 加载器 + 缓存机制，兼容 Python 3.14 + pygame-ce
- **资源路径自适应**：`path_util.py` 统一处理开发/打包双环境资源定位
- **PyInstaller 打包配置**：一键生成单文件 exe，无需 Python 环境即可运行

#### 修复
- **Python 3.14 兼容**：`pygame.font.SysFont(None, n)` 触发 TypeError，全量替换为 `Font(None, n)` + 系统字体兜底
- **`NameError: grid`**：`game_state.update()` 签名补全 `grid` 参数传递
- **敌人不死**：`enemy_base.update()` 补充 `hp <= 0 → dead = True` 判定
- **`IronGolem` 无 `range`**：`__init__` 补 `self.range = d["range"]`
- **`FloatText` 未导入**：`game_state.py` 顶部补 `from game.ui.float_text import FloatText`
- **`Entity` 未导入**：`enemy_base.py` 顶部补 `from game.entities.entity import Entity`
- **`farmer` KeyError 连环炸**：`hp` / `gen_amount` / `gen_interval` 全量改为 `.get(key, default)` 兜底
- **`priest` KeyError**：`heal_amt` 字段对齐 config，`d.get("heal_amt", d.get("heal", 8))` 双兼容
- **`nitwit` key 不一致**：类内 `TOWER_DEFS["nitwit_mine"]` → `"nitwit_idle"` 与配置统一
- **放置逻辑坐标偏移**：`try_place` 加入 `GRID_ORIGIN_X/Y` 偏移计算，防止点击越界
- **遍历中删除列表元素**：敌人/子弹更新改为 `for x in list[:]` 拷贝遍历 + 延迟移除
- **`pygame.font` 模块顶层初始化崩溃**：`fonts.py` 移除顶层 `Font()` 调用，改为 `get_font()` 首次调用时延迟加载
- **PowerShell 打包命令报错**：`^` 续行符改为单行命令或 PowerShell 反引号续行

#### 调整
- 窗口默认 1000×600，60FPS
- 塔费用平衡：Farmer(50) / Archer(75) / Golem(150) / Cleric(100) / Nitwit(25)
- 敌人速度/血量按波次递增
- 卡片面板英文命名（MC 原版职业名），避免中文乱码
- 碰撞箱调试开关默认 `False`，发布版关闭

---

## 项目结构

```
village_vs_grox/
├── main.py                    # 程序入口，主循环
├── config.py                  # 全局配置（塔/敌人数据、网格参数、常量）
├── vvg_1.26.11.py            # 主程序别名（便于打包）
├── assets/                    # 资源文件
│   ├── fonts/
│   │   └── simhei.ttf        # 中文字体
│   ├── towers/               # 塔图片（farmer_idle.png 等）
│   ├── enemies/              # 敌人图片（pillager.png 等）
│   └── farmers/              # 农民变体图片
├── game/
│   ├── game_state.py         # 游戏状态管理（经济/生命/实体列表）
│   ├── grid.py               # 网格系统与塔放置逻辑
│   ├── wave_manager.py       # 波次生成与无尽轮回
│   ├── entities/
│   │   ├── entity.py         # 实体基类
│   │   ├── towers/           # 防御塔（farmer/archer/iron_golem/cleric/nitwit）
│   │   ├── enemies/          # 敌人（pillager/ravager/vindicator/enemy_base）
│   │   ├── projectiles/      # 抛射物（arrow）
│   │   └── float_text.py     # 飘字
│   ├── ui/
│   │   ├── card_panel.py     # 选卡面板
│   │   ├── hud.py            # 头顶信息显示
│   │   └── float_text.py     # 飘字 UI
│   └── utils/
│       ├── fonts.py          # 字体加载器（中文字体+缓存）
│       ├── loader.py         # 图片资源加载器
│       └── path_util.py      # 开发/打包双环境路径适配
├── data/                      # 存档数据（预留）
└── docs/                      # 文档（预留）
```

---

## 运行要求

- **Windows 7 及以上**
- **Python 3.8+**（源码运行）或直接使用打包版 exe
- **依赖**：`pygame>=2.0.0`（推荐 `pygame-ce>=2.5.0`）

## 操作说明

| 操作 | 功能 |
|------|------|
| `1-5` | 选择防御塔 |
| `鼠标左键` | 放置选中的塔 / 点击选卡 |
| `鼠标移动` | 预览塔放置位置 |
| `绿宝石` | 击杀敌人或 Farmer 自动产出 |
| `村庄生命` | 敌人到达右端扣除，归零则失败 |

## 许可

MIT License — 自由使用、修改、分发。MC 相关素材版权归 Mojang Studios 所有，本项目仅作非商业同人创作。
