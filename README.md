# KG PDF Project（糊弄版）

基于 `knowledge_graph_skill.md` 的 PDF 课程知识图谱生成项目。

目标是把课程 PPT / PDF 讲义直接转换为 **可适配 KT-SQEP 知识图谱工具** 的最终 XML，而不是先生成中间 schema 再二次转换。

## 项目内容

- [knowledge_graph_skill.md](/C:/Users/asus/Desktop/KG_PDF_Project/knowledge_graph_skill.md)
  最新版 skill 规则
- [ppt](/C:/Users/asus/Desktop/KG_PDF_Project/ppt)
  待处理 PDF
- [output](/C:/Users/asus/Desktop/KG_PDF_Project/output)
  生成后的 XML

## Skill 定位

`knowledge_graph_skill.md` 的作用是：

- 从 PDF 中优先抽取内容型实体 `KA / KU / KP / KD`
- 抽取并组织每个节点内部的 `K / T / Z / Q / E / P`
- 默认不生成资源型实体 `VD / PT / PD`
- 默认不生成 `RESOURCE` 边
- 默认直接输出 **工具兼容版 XML**
- 默认保存到指定目录，文件名为 `PDF文件名.xml`

## 默认输出规则

### 1. 输出结构

默认直接生成：

```xml
<KG>
  <entities>...</entities>
  <relations>...</relations>
</KG>
```

不再默认生成：

- `<graph><nodes><edges>`
- `content_ordered` 中间 schema
- 资源节点和 `RESOURCE` 关系

### 2. 节点类型

- `KA` -> `知识领域:KA`
- `KU` -> `知识单元:KU`
- `KP` -> `知识点:KP`
- `KD` -> `关键知识细节:KD`

### 3. 节点固定字段

每个内容节点都应包含：

- `id`
- `class_name`
- `classification=内容方法型节点`
- `identity=知识`
- `level`
- `attach`
- `opentool=无`
- `content`
- `K`
- `T`
- `Z`
- `Q`
- `E`
- `P`
- `x`
- `y`

### 4. attach 编码

`attach` 固定为 6 位，顺序是：

`K / T / Z / Q / E / P`

规则：

- 有内容记为 `1`
- 无内容记为 `0`

例如：

- `110111` 表示该节点有 `K/T/Q/E/P`，没有 `Z`
- `111111` 表示六类内容都存在

### 5. 关系映射

- `CONTAINS -> 包含关系`
- `ORDER -> 次序：次序关系`
- `KEY_ORDER -> 次序：次序关系`

关系固定字段：

- `name=包含`
- `mask=知识连线`
- `head_need=内容方法型节点`
- `tail_need=内容方法型节点`

## 布局规则

为保证在 KT-SQEP 工具中直接打开时不被遮挡，默认布局要求：

- 最小 `x >= 320`
- 最小 `y >= 180`
- 图整体不要偏左
- 图整体不要偏上
- 层次清晰，尽量减少交叉
- `KA` 通常位于顶部中心
- `KU` 在第二层横向展开
- `KP` 在第三层展开
- `KD` 在更下方并尽量与所属 `KP` 对齐

## 节点规模建议

- 总节点数不少于 `40`
- 优先建议 `45~70`
- 默认主干结构：

```text
KA -> KU -> KP -> KD
```

推荐下限：

- `KA >= 1`
- `KU >= 3`
- `KP >= 10`
- `KD >= 10`

## 推荐使用方式

### 标准提示词

处理新 PDF 时，推荐直接使用下面这段提示词：

```text
请使用 `knowledge_graph_skill.md` 的最新规则处理我上传的 PDF，并直接生成可适配 KT-SQEP 知识图谱工具的最终 XML。

要求如下：
1. 直接生成工具兼容版 XML，不要先生成中间 graph-schema XML。
2. 输出结构必须为 `<KG><entities><relations>`。
3. 默认不要生成资源型实体（VD/PT/PD），也不要生成 RESOURCE 边。
4. 节点只保留内容型实体：KA、KU、KP、KD。
5. 总节点数不少于 40。
6. 每个内容型实体内部必须包含并严格按以下顺序输出：
   `content`、`K`、`T`、`Z`、`Q`、`E`、`P`
7. 必须为每个实体生成 `attach`，按 `K/T/Z/Q/E/P` 顺序用 6 位 0/1 编码表示内容是否存在。
8. 字段必须符合工具兼容格式：
   - `class_name`
   - `classification=内容方法型节点`
   - `identity=知识`
   - `level`
   - `attach`
   - `opentool=无`
   - `content`
   - `K/T/Z/Q/E/P`
   - `x`
   - `y`
9. 关系只使用：
   - `CONTAINS -> 包含关系`
   - `ORDER -> 次序：次序关系`
   - `KEY_ORDER -> 次序：次序关系`
10. 布局要避免遮挡：
   - 最小 `x >= 320`
   - 最小 `y >= 180`
   - 整体不要偏左、不要偏上
   - 层次尽量清晰，减少交叉
11. 输出文件直接保存到 `output/` 目录。
12. 文件名使用“PDF文件名 + .xml”。
13. 写入文件的内容必须是纯 XML，不要把解释文字写进文件。
14. 最终只告诉我生成完成后的文件路径，以及节点数、边数。
```

### 简短版提示词

```text
请按 `knowledge_graph_skill.md` 的最新规则，直接把我上传的 PDF 生成成 KT-SQEP 工具可导入的最终 XML，保存到 `output/` 下，文件名使用 PDF 同名 `.xml`。不要生成资源型实体，不要走中间 schema。节点不少于 40，每个内容节点必须带 `content + K/T/Z/Q/E/P + attach + x/y`，布局避免左侧和顶部遮挡。最终只回复生成文件路径、节点数和边数。
```

## 输出文件命名规则

默认命名规则：

```text
PDF文件名 + .xml
```

例如：

- `2020ERP&SCM第1讲-引论.pdf`
  -> `2020ERP&SCM第1讲-引论.xml`
- `2020ERP&SCM第2讲-理解典型企业过程-船舶制造示例(1).pdf`
  -> `2020ERP&SCM第2讲-理解典型企业过程-船舶制造示例(1).xml`

默认保存位置：

```text
output/
```

## 推荐工作流

声明：本工作流目前只在 Codex 中验证过有效性。

在放入 PDF 之前，先建立如下目录结构：

```text
KG_PDF_Template/
├─ knowledge_graph_skill.md
├─ ppt/
└─ output/
```

1. 把 PDF 放入 `ppt/` 目录。
2. 用标准提示词调用 skill。
3. 检查 `output/` 中生成的 XML。
4. 导入 KT-SQEP 工具验证布局与内容。
5. 若导入后发现偏左、偏上或字段兼容性问题，再小幅调整坐标或字段映射。

## 已验证的经验规则

根据当前项目的调试结果，以下规则对 KT-SQEP 工具兼容性最重要：

- 使用 `<KG><entities><relations>` 而不是旧的 `<graph>` 结构
- 节点内部直接写 `K/T/Z/Q/E/P`
- `content` 只放节点标题
- `attach` 必须和 `K/T/Z/Q/E/P` 的存在情况一致
- 默认移除资源节点
- 节点整体右移、下移，避免图谱贴左上角


