# 原理图API、扇出与增量ECO

## 1. 调用前核对

下列方法来自既有执行经验与EasyEDA官方API，部分处于BETA。先核对实际客户端可用签名、返回结构与坐标，方法不存在时保留具体缺项。通用接口查询和连接由API Skill负责。

| 操作 | 现有接口与注意点 |
| --- | --- |
| 通过商城编号定位器件 | `eda.lib_Device.getByLcscIds(...)`；从返回对象取得真实UUID/libraryUUID，不能将商城编号当UUID |
| 放置器件 | `eda.sch_PrimitiveComponent.create(...)`；保留器件引用后显式设置位号并等待提交 |
| 读取引脚 | `eda.sch_PrimitiveComponent.getAllPinsByPrimitiveId(id)`；移动、旋转、替换后重读 |
| 电源/地/信号端口 | `createNetFlag(...)`、`createNetPort(...)`；依真实网络和电气方向选择 |
| 导线 | `eda.sch_PrimitiveWire.create(...)`；查询中曾使用`.line`和`.net`，先验证字段，不调用不存在的`getState_Coordinates()` |
| 分区矩形/文字 | `sch_PrimitiveRectangle.create(...)`、`sch_PrimitiveText.create(...)`；检查透明填充、锚点和字号 |

异步位号设置的既有模式：

```javascript
const asyncObject = component.toAsync();
asyncObject.setState_Designator(designator);
await asyncObject.done();
```

执行后读取位号，不能只相信调用返回。Bridge通常只注入 `eda`，TypeScript文档里的枚举名不一定成为全局变量。必要时用经文档和小样例核对的数字映射，禁止猜枚举。

## 2. 扇出适用范围

引脚扇出用于主控边界或远端连接。先逐脚确定网络和方向，再计算引脚朝外的短线与标签锚点。用当前符号真实引脚端点、旋转/镜像和一处可见样例校准向量，不能把所有符号的0/90/180/270度含义硬编码为同一种朝向。

本地两器件相连时直接创建端点间短线，端口只在有命名或跨区价值时生成。NC、EP、安装脚和重复电源脚按器件说明分别处理，不以“没有名称”或“机械脚”统一跳过。

`getAll()`可能包含真实器件、网络标志和端口。统计优先使用可用类型字段，位号为空只作为辅助条件，不作为删除授权。

## 3. 所有权与删除

同步范围与人工修改处理统一遵循[需求补全与ECO](intent-and-eco.md)。这里的局部删除保护不限制已授权的同板全差异合并，也不否定用户在执行中新增的修改。

需要清理旧电路时保存该组件的引脚端点、专属导线ID和专属端口ID。共享电源干线或连接多器件的同网导线不得随着一个器件删除。

替换流程：读取旧状态与网表 → 建立旧新脚号映射 → 放置/修改目标器件 → 重读新脚端点 → 重接所属支路 → 删除已证实无引用的旧线和标签 → 比对网表 → 保存与截图。

位置靠近、落在同一矩形或没有位号都不能独立证明对象归属。出现无法区分的共享线时，保留原对象并只重建已知专属支路。

## 4. 调用规模与读回

按照一个完整功能块或一组可验证连接组织操作。历史一次数百操作成功及其耗时不能作为默认批量参数。每组检查位号、脚号、网络与几何，遇到客户端量化或部分成功时从读回状态生成剩余操作。

几何修正后还要比较网络成员。网名数量相等、器件总数相等都不能发现两个引脚互换，因此比较键至少包含“位号＋引脚号＋网络”。

## 5. API依据

核验日期：2026-09-14；BETA接口在运行前检测。

- 器件与引脚：`https://prodocs.lceda.cn/cn/api/reference/pro-api.sch_primitivecomponent.html`
- 引脚读取：`https://prodocs.lceda.cn/cn/api/reference/pro-api.sch_primitivecomponent.getallpinsbyprimitiveid.html`
- 原理图单位示例：`https://prodocs.easyeda.com/en/api/reference/pro-api.sch_primitivecircle.getall.html`

## MCP 1.6 逻辑焊盘目标

`expectedAfter.pads`的每行二选一：`{primitiveId, net}`或`{componentUniqueId, padNumber, net}`。源组件uniqueId可从真实组件或`pcb_inspect_pinmap`读回。第二种身份经当前组件到焊盘的明确父关联解析，适合物理ID可能重建的流程，不代替源端引脚功能核对。未知字段、重复目标、组件删除与其焊盘存在要求冲突时在导入前拒绝。

只有postconditionsVerified=true才表示已提供的目标全部满足；未提供目标仍为null。真实换封装、源端改针、增删器件的工程验证不能由合成ID变更测试代替。
