# FAC 仓库比较与复用审计
核查日期：2026-10-17。范围：两个仓库的 master；FAC-assembly `12fc8d225b7e9f8e1a9090a6272294467b2f014d`，FAC-CR-NGS `beae0595930517a743cb41dc518dfbda0b69a461`。这是文件与代码静态核查，不是端到端运行验证或生物学验证。

## 已确认的仓库状态
- FAC-assembly：根目录 9 个文件，无子目录。四个 Python 脚本、README、BSD-2-Clause LICENSE，以及 TSV、FASTA、GenBank 各一个。它是 nextgenusfs/FAC-assembly 的 fork；不能把上游代码归为本项目原创。
- FAC-CR-NGS：仅 README.md（152 bytes）和 GPLv3 LICENSE（35,149 bytes）。README 表述筛选大规模 FAC 文库中次级代谢物生物合成基因簇的目标，但没有实现、输入定义或验证证据。
- GitHub API size 分别为 21,864 和 19；它们不能用来判断代码量、成熟度或可靠性。当前树中最大文件 NC_007194.1.final.gbk 为 9,424,382 bytes；四个脚本合计仅 45,463 bytes。
- 分支列表均只有 master；两个仓库 open PR / open issue 集合均为空。FAC-assembly 的 has_issues=false，因此空 Issue 列表也受仓库功能设置影响。
- 两个当前树均无测试、CI、依赖锁文件或端到端原始 reads。

## 文件级复用判断
| 源文件 | 已确认内容 | 建议 |
|---|---|---|
| [README.md](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/README.md) | 分池组装、Sanger 末端归属、可选迭代扣除的步骤说明 | 提取流程概念与字段说明，重写命令、依赖和输出文件名 |
| [assemble_facs.py](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/assemble_facs.py) | minimap2/samtools 扣除、Shovill 或 Unicycler、回贴和 coverage 汇总 | 重写编排；不能直接作为已验证管线迁移 |
| [get_full_length_facs.py](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/get_full_length_facs.py) | TSV 解析、按 row/column/plate 池名筛选、mappy 比对、方向处理、分类 FASTA | 复用字段与分类思路；修复后移植，分类仅代表计算规则 |
| [filter_seqs.py](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/filter_seqs.py) | 长度过滤、header 字符串物种过滤、每个 origin 保留最长记录 | 最小可先移植模块，但先修复缺失 origin 的行为；不是序列去重或物种鉴定 |
| [antiSMASH2tsv.py](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/antiSMASH2tsv.py) | 读取已有 GenBank 注释和 cluster feature，导出基因/转录本信息 | 重写独立适配器；它不运行 antiSMASH，也不预测 BGC |
| [IGteloFACpooldataNF.txt](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/IGteloFACpooldataNF.txt) | 9 列；595 条数据行、595 个唯一 FAC_Name；前后端序列字段均非空 | 直接复用表头定义；真实数据不默认复制为本项目样本 |
| [pFACvector.fasta](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/pFACvector.fasta) | 1 条 16,040 nt 序列，header 标记 circular=true | 仅在确认当前实验载体相同后按原字节复制并记录校验值；header 不能证明真实环状性 |
| NC_007194.1.final.gbk | 树中可确认路径、大小、blob SHA | 本次接口读取返回空内容，未完成正文核查；不推定物种、注释来源或结果可信度 |
| [LICENSE](https://github.com/hychen86/FAC-assembly/blob/12fc8d225b7e9f8e1a9090a6272294467b2f014d/LICENSE) | BSD 2-Clause，Copyright (c) 2018 Jon Palmer，要求保留声明、条件和免责声明 | 若复制代码/文档，携带原许可全文和来源；保留 FAC-CR-NGS 现有 GPLv3 LICENSE |

可直接迁移的范围限于准确摘录的输入字段、来源/许可声明及经校验的参考文件字节；四个脚本都不应未经修改直接作为生产实现。目标目录结构是本方案新设计，源仓库并没有现成目录架构。

## 影响迁移的具体问题
1. assemble_facs.py:26–34：默认 assembler 为 shovill；默认 --subtract=CP014272.1。158–171 行把参数当作 accession，并寻找 accession.fna 或联网下载，不能把 help 中的“FASTA sequence”理解为可靠的本地路径接口。当前文库是否适合该扣除参考无法确认。
2. assemble_facs.py:193–205：Unicycler 默认质控对原始 args.forward/reverse 执行 Trim Galore，随后用质控产物组装；只有 skip_quality 分支使用 cleanR1/2。应统一为显式的扣除→质控→组装数据流，并检查每个阶段的输入来源。
3. assemble_facs.py:189、232–234：最终输出为 base.assembly.fasta，而非 README 的 base_unicycler.fasta；固定 shovill_out 且存在时追加 --force。basename 截断至第一个下划线，也可能让不同样本共用输出。
4. 多处 subprocess.call / Popen 未检查退出状态；按文件存在性复用中间结果，无法证明完整性。samtools fastq 前使用默认排序，需验证成对 reads 同步；新实现应明确 name-collate/name-sort、孤立 reads 和次级比对策略。大 FASTQ 计数使用 readlines，需改流式处理。
5. get_full_length_facs.py:94 重复检查 fwd 长度，漏检 rev；224 行正常路径 sys.exit(1)。输入短行被静默跳过、重复 FAC 被静默保留首条，需改为显式校验。
6. get_full_length_facs.py:113：仅保留 primary、MAPQ≥40、contig 长度≥20,000（可调）、query coverage≥50%；identity 被记录但未设过滤阈值。这些是代码阈值，不是经证实的实验标准。多候选命中、坐标顺序与歧义需专门处理。
7. get_full_length_facs.py:139–181：同 contig 且相反链按末端区间裁切；异 contig 以 100 个 N 连接，标记 stitched，仅写入 all-contigs。stitched 不代表闭合或完整 FAC；yes 也不能替代实验确认。
8. filter_seqs.py:76–80：无 origin 时 contig 可能未定义，或沿用上一记录的值；物种仅做 header 子串匹配；longest 按 origin 字符串分组，并非按序列唯一性去重。README 的 -l 80000 错，应使用 -m 80000。
9. antiSMASH2tsv.py:417、434、440：表头 name/product/type 对应输出却是 type/name/product；154–155 行生成 1-based inclusive 基因位置，436 行却直接用于 Python 切片，gDNA 起点偏移一位；负链 gDNA 未反向互补。399 行仅识别 cluster feature，不能宣称支持所有 antiSMASH 版本；CDS 首次出现分支未保存 protSeq，也需覆盖测试。
10. 源 README 指向 Python 2，代码含 rU、unicode 与旧位置属性；不能仅修改 shebang 就宣称 Python 3 可用。natsort、interlap 和 Shovill 等依赖在 README 中也未完整列出。

## 可复用的流程及边界
建议流程：输入验证→显式载体/污染参考扣除→质控→分池组装→reads 回贴与覆盖汇总→Sanger 末端定位和 FAC 候选分类→可选的已注释 GenBank 转表。
前六步都有源码对应，但尚未运行验证。antiSMASH 注释生成本身、大规模文库 BGC 筛选/排序、CR 的实验或算法含义、跨池消歧均无充分实现证据。
源 README 的多轮策略可以保留为可选研究方案；首轮通过验证后再实现，记录每轮被扣除的 reads 和候选，避免重叠插入片段被过度扣除。

## 不能由仓库确认的信息
- CR 的全称、实验设计、池编码、真实样本清单、当前载体、宿主/污染参考。
- FASTQ 的读长、质量、接头、配对规则、数据许可与公开范围。
- 工具实际运行版本、参考数据库版本、硬件资源、批量规模、耗时和内存。
- FAC 完整性真值、组装准确度、BGC 阳性/阴性标准、假阳性/假阴性、任何筛选性能或代谢物产量。
- GenBank 正文与生成来源、antiSMASH 版本和数据库；TSV 中的物种标签不构成独立鉴定。
这些字段应保持“待确认”，不填入猜测值，不展示虚构实验结果。

## 交付与提交顺序
第一批文档可直接提交：README.md、docs/repository-audit.md、docs/workflow-spec.md、docs/implementation-plan.md、examples/fac_ends.header.tsv。本报告及这些文件均为新写内容；没有复制源脚本作为可运行实现。
第二批按 implementation-plan.md 实现与测试。尚无运行验证时，不添加“robust/quick”、性能数字、安装成功承诺或全流程运行命令。
