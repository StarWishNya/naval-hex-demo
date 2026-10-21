# 舰船数据来源与口径

本游戏的八艘舰艇均为《战舰世界》WG 正式服 X 级舰船。数据快照取自[浩舰战舰世界数据库](https://iwarship.net/wowsdb/index) 2026-10-19 显示的 **WG 服 0.15.9.0**，版本 ID `442`；未使用 WG 公测服或 Lesta 服。离线游戏运行时不访问该站。

原始读取路径：

- [WG 舰名表](https://iwarship.net/wowsdb/data/ship-name/wg)：船只 ID、中文与英文名称。
- [WG 0.15.9.0 分类属性](https://iwarship.net/wowsdb/data/get-ship-detail/442)：`hull`、`artillery`、`penetration`、`torpedoes`、`ability`。响应中的每个分类值仍是 JSON 字符串，需要第二次解析。
- 逐舰详情： [大和](https://iwarship.net/wowsdb/detail/442-PJSB018)、[蒙大拿](https://iwarship.net/wowsdb/detail/442-PASB017)、[得梅因](https://iwarship.net/wowsdb/detail/442-PASC020)、[兴登堡](https://iwarship.net/wowsdb/detail/442-PGSC110)、[米诺陶](https://iwarship.net/wowsdb/detail/442-PBSC110)、[伍斯特](https://iwarship.net/wowsdb/detail/442-PASC210)、[岛风](https://iwarship.net/wowsdb/detail/442-PJSD012)、[克莱贝尔](https://iwarship.net/wowsdb/detail/442-PFSD110)。炮塔逐座射界取页面内的 `component.*_Artillery.turrets[].guns`，相对朝向取 `component.*_Hull.weaponAngles`。舰名表中另有同名旧伍斯特 `PASC016`，本快照使用正式研发舰 `PASC210`。

`data.js` 中的 HP、航速、基础水面隐蔽、烟内开火隐蔽、射程、装填、HE/AP 最大伤害、HE 穿深和起火率均来自上述正式服属性。无舰长技能、升级品、信号旗等加成。米诺陶没有 HE；其烟与雷达同槽，游戏固定选烟。[WG Wiki 米诺陶页](https://wiki.worldofwarships.com/Ship:Minotaur)也确认这两点。`cls` 中 CA/CL 是为棋盘队伍编成手工指定：得梅因、兴登堡为 CA；米诺陶、伍斯特为 CL，原数据只统称 `Cruiser`。

岛风固定 **Type93 mod. 3，12 km**：选用 `PJUT904_D10_TORP_MED`，原数据给出 67 节、1.7 km 鱼雷隐蔽、180 m 起爆距离、单枚最大伤害 23,767。与[WG Wiki 鱼雷表](https://wiki.worldofwarships.com/Ship:Torpedo_Data)相符。标伤取原始字段 `portDamage`；`alphaDamage=66800` 是内部伤害计算值，**不是**游戏面板上的鱼雷标伤。其他鱼雷亦同。棋盘的 `torpedo.count` 表示一次舷侧可发射管数：岛风 15、兴登堡 8、米诺陶 8、克莱贝尔 6；现实游戏中各发射器可独立操作，故棋盘齐射是简化。米诺陶的逐枚发射能力也未在此模型中复刻。

AP `pen` 采样点取数据库 `penetration.penVertAt0/10/14/18/22`。当主炮最大射程不恰好落在样本点，末点用邻近两样本按 `P(d)=P(a)+(P(b)-P(a))×(d-a)/(b-a)` 线性求出；在 22 km 之后则由 18/22 km 两点线性外推。棋盘在相邻已存点之间同样需线性插值。原站的穿深样本本身也是计算值，不是实测值；浩舰仅说明其算法因炮塔高度与精度可能有 **10 mm 内的误差**，没有给出对所有距离都成立的误差保证。线性外推没有验证误差上界，因此全部标 `penEstimated: true`，只能视作**近似穿深**。炮塔原始射界投影为每座炮塔一个对称扇区；蒙大拿 3 号、米诺陶 2 号、伍斯特部分炮塔存在特殊边界或死区，已标 `estimatedArc: true`，棋盘无法完整表达原始射击死区。

`armor` 的船首、船尾、侧面、装甲带四个数是**手工设计的简化代表区**，统一标 `armorApprox: true`，**不是** WG 的整舰装甲剖面，也不能用于历史舰船比较或真实 WoWS 穿甲计算。原始分类属性没有各板件位置、倾角、内部多层结构和动态命中区域。有关真实游戏机制可参看 [WG Wiki 穿甲机制](https://wiki.worldofwarships.com/Ship:Armor_Penetration)：AP 跳弹、强制碾压、过穿、核心区和伤害饱和会改变实际伤害，HE 穿深则按装甲厚度判断。`he.damage`、`ap.damage` 与鱼雷 `damage` 都是*最大面板伤害*，并非每次命中的固定扣血。

`torpedoProtection` 取同一正式服快照的 `hull.flood`，表示船体的**鱼雷防护减伤百分数**：大和 55%、蒙大拿 37%、得梅因 7%、兴登堡 13%、米诺陶 13%、伍斯特 19%、岛风与克莱贝尔 0%。它是舰船防护属性，不是进水概率；鱼雷实际结算还会受命中区域等真实游戏机制影响。

可执行道具按 `ability` 的互斥槽选定：全部舰艇有损害管制；得梅因、伍斯特用水听与雷达；兴登堡用强化水听；米诺陶用水听与烟；岛风用烟与引擎增压；克莱贝尔用引擎增压与主炮装填加速。大和、蒙大拿、得梅因、兴登堡、米诺陶、伍斯特另有可执行 `repair`（由 `repairParty` 映射：次数、时长、冷却、恢复速率、船体/核心区可修复比例）；岛风与克莱贝尔无修理班。蒙大拿结算使用精确速率 `0.66%/s`（快照展示值 `0.7` 仅作记录）。`ap.fuseDelay` 取自同一快照的 `APbulletDetonator`（秒）。`DATA.apDesign` 是棋盘设计参数（落点权重、内层厚度、舰内路径、引信行程系数 `fuseSpeed`），标 `design:true`，不是原版装甲剖面。火灾为四独立火区；点火率使用面板基础值 × Tier X 火灾抗性系数 `0.5005`。完整的飞机、防空、副炮、舰长技能、模块失能和舰艇升级品不在此 MVP 中。

特色交叉核对：[大和](https://wiki.worldofwarships.com/Ship:Yamato) 460 mm 炮与 32 mm 强制碾压；[蒙大拿](https://wiki.worldofwarships.com/Ship:Montana) 12 门 406 mm 炮；[得梅因](https://wiki.worldofwarships.com/Ship:Des_Moines) 快速 203 mm 炮和优异 AP 跳弹角；[兴登堡](https://wiki.worldofwarships.com/Ship:Hindenburg) 51 mm HE 穿深、水听与短程鱼雷；[伍斯特](https://wiki.worldofwarships.com/Ship:Worcester) 12 门快速 152 mm 炮；[岛风](https://wiki.worldofwarships.com/Ship:Shimakaze) 三种雷型与低隐蔽；[克莱贝尔](https://wiki.worldofwarships.com/Ship:Kl%C3%A9ber) 44 节基础航速、无烟、强化引擎和主炮装填加速。烟幕在真实游戏中遮挡视线，但有近距保底发现和烟内开火暴露，见 [WG Wiki 烟幕规则](https://wiki.worldofwarships.com/Ship:Smoke_Screens)。
