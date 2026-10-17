# 拟议流程与数据约定

状态：设计规范，尚未实现。旧实现与问题见 [审计](repository-audit.md)。本文件中的输出名称是新约定，不是现有脚本保证。

## 输入

1. 新增样本清单 schema：sample_id、pool_id、r1、r2。要求 ID 唯一、路径存在、R1/R2 记录与 read name 对应。首期每条记录是一对 FASTQ；多 lane 合并行为另行定义。
2. FAC 末端 TSV 复用旧表头 9 列：Organism、TeloFAC_Order、FAC_Name、Sanger_fwd_seq、Sanger_rev_seq、Column_Pool、Row_Pool、Plate_Pool、Illumina_seqpools。
3. FAC_Name 必须唯一；空末端、无效字符、短行或未知池返回明确诊断，不能静默丢弃。缺一端可标为单端候选；缺两端无定位证据。空值的具体表示由新校验器固定后写入示例。
4. 旧解析器按第 6–8 列任一匹配选择 FAC；第 9 列不参与其逻辑。保留此说明，不假定它已完成跨池交叉验证。
5. subtraction references 必须明确指定本地 FASTA、用途、来源与校验值；不默认沿用 CP014272.1。载体与宿主参考分别记录。未确认实验载体前不把源 pFACvector.fasta 设为默认。
6. GenBank 转表是独立可选入口，输入必须已包含注释；不自动声称具备 BGC 预测能力。

## 阶段与拟议产物

| 阶段 | 输入→产物 | 验收要求 |
|---|---|---|
| 校验 | manifest/FASTQ/FAC TSV→validation.json | 报告全部错误，不启动分析 |
| 扣除 | paired FASTQ + references→clean_R1/R2.fastq.gz | 两端同步；保留/丢弃和孤立 reads 规则明确 |
| 质控 | clean reads→trimmed_R1/R2.fastq.gz | 后续组装必须消费该阶段产物；skip 时明确消费 clean reads |
| 分池组装 | processed reads→assembly.fasta | 显式选择 assembler；非零退出立即停止 |
| 回贴 | processed reads + assembly→coverage.tsv | 明确回贴的 reads 集；定义分母与比对过滤，不把 coverage 等同准确度 |
| 末端定位 | assembly + FAC TSV→end_hits.tsv、candidates.tsv、candidate FASTA | 记录阈值、坐标、链方向、所有候选与排除原因 |
| 可选注释转换 | annotated GenBank→genes.tsv | 固定 schema/坐标约定；不支持的 feature 格式明确拒绝 |

所有输出置于 results/<sample_id>/，每次独立工作目录。run.json 记录命令参数、工具版本、输入/参考校验值、时间和退出状态。复用中间结果须核对完整记录，不能只检查文件存在。

## 候选判定

采用 0-based half-open 内部坐标；导出时明确坐标约定。旧代码阈值 MAPQ≥40、query coverage≥50%、contig≥20 kb 仅作为待评估起点，全部配置化，不宣称已适用于 FAC-CR-NGS。

- 同 contig、相反方向、区间顺序有效且无未解决歧义：标记 end_supported，说明它只具备末端比对支持。
- 单端：single_end；同向/反序：orientation_conflict；多候选：ambiguous；无命中：unmapped。
- 两端落在不同 contig：split_contig。默认不拼接；如保留旧 100 N 拼接选项，独立标记 stitched_with_gap，不计作完整 FAC。
- 物种名称来自输入元数据，不能当作序列分类结论。

首期不默认启用迭代扣除。BGC 注释运行、跨池整合、候选优先级、实验阳性定义均待补充规范。
