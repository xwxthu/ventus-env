# Bug 分类表说明

本文档说明 `bug-classification.csv` 中各字段的含义。该表用于在开展 Ventus 故障注入实验之前，对数据集中的每个 bug 进行分类。

表中的每一行对应 `bug-dataset/` 中的一条 bug 记录。详细的 bug 元数据仍保留在原始 JSON 文件中；该 CSV 只记录用于选择合适故障注入目标的分类判断。

## 字段

### `bug_id`

六位数字 bug ID，与 `bug-dataset/` 中对应的 JSON 文件名一致。

示例：

```text
000235
```

### `ventus_rtl_mappable`

表示该 bug 是否可以合理映射到当前 Ventus RTL 实现，并在其中进行故障注入。

允许取值：

```text
yes      该 bug 可以映射到一个具体的 Ventus RTL 机制。
partial  Ventus 中存在相似机制，但映射关系是间接的或近似的。
no       Ventus 缺少所需模块/机制，或该 bug 不适合进行 RTL 注入。
unknown  无法通过静态检查可靠判断其映射关系。
```

### `mappability_reason`

对 `ventus_rtl_mappable` 判断的简短原因说明。

允许取值：

```text
module_exists             Ventus 中存在相关或相似的 RTL 机制。
module_absent             Ventus 不包含所需模块或机制。
isa_mismatch              该 bug 依赖当前 Ventus 目标不支持的 ISA 行为。
ai_accelerator_only       该 bug 针对 Ventus 中不存在的 AI 加速器模块。
too_large_refactor        注入该 bug 需要大范围重构，而不是局部修改。
not_functional_behavior   该 bug 主要涉及非功能行为，例如性能或构建行为。
unknown                   无法通过静态检查可靠判断原因。
```

### `diff_detectability`

表示该 bug 在被注入并触发后，是否预期能被当前 GVM 差分检查流程检出。

允许取值：

```text
current_gvm          当该 bug 的影响传播到已检查的事件/状态时，当前 GVM 应该能够检出。
needs_probe_or_pics  该 bug 原则上可检出，但当前 GVM 可能需要额外 probe、PICS 或输出检查。
not_diff_target      该 bug 不适合作为功能差分验证目标。
unknown              无法通过静态检查可靠判断可检出性。
```

`current_gvm` 不要求 bug 必须在错误行为首次出现的那条指令上立即被检出。如果错误状态可以传播到后续指令，并最终出现在当前 GVM 已经比较的状态中，则仍分类为 `current_gvm`。

如果该 bug 只能通过额外插桩，或通过当前 GVM 尚未提供的最终体系结构状态/输出检查来观察，则分类为 `needs_probe_or_pics`。

## 使用解释

对于后续故障注入实验，我们将满足以下条件的行定义为当前可开展实验的 bug：

```text
ventus_rtl_mappable in {yes, partial}
diff_detectability = current_gvm
```

该定义表示：该 bug 与当前 Ventus RTL 存在直接或相似的映射关系，并且在注入故障被触发、其影响传播到已检查事件/状态后，当前 GVM 应该能够检出。

标记为 `needs_probe_or_pics` 的行仍可能有价值，但它们需要额外观测或检查支持，因此不包含在当前可开展实验集合中。标记为 `not_diff_target` 或 `ventus_rtl_mappable = no` 的行，不是当前 Ventus/GVM 故障注入实验的主要候选目标。
