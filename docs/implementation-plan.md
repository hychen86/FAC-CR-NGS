# 文件级实施计划

本次第一批仅提交文档和表头模板；下表中的程序文件均为后续建议，尚未创建。不复制大体积历史数据或将旧脚本直接标为生产管线。

## 第一批：可直接提交

| 文件 | 操作 | 内容与验收 |
|---|---|---|
| README.md | 替换 | 明确设计阶段、目标、文档入口和未确认事项；移除未经验证的 robust/quick 表述 |
| docs/repository-audit.md | 新增 | 固定源 SHA、逐文件复用判断、源码问题和证据边界 |
| docs/workflow-spec.md | 新增 | 输入输出约定、数据流、候选状态、版本与来源记录要求 |
| docs/implementation-plan.md | 新增 | 本计划 |
| examples/fac_ends.header.tsv | 新增 | 仅旧格式表头，ASCII 空白规范化；没有虚构样本 |
| LICENSE | 保留 | 不替换现有 GPLv3 全文 |

建议提交标题：docs: define evidence-based FAC workflow and migration plan

## 第二批：先做可独立验证的模块

| 目标文件 | 来源/处理 | 必须通过的检查 |
|---|---|---|
| src/fac_cr_ngs/inputs.py | 新写；借鉴旧 parseInput 字段映射 | 重复 ID、短行、空两端、无效字符和未知池明确报告；验证 R1/R2 同步 |
| src/fac_cr_ngs/filter_fasta.py | 修订移植 filter_seqs.py | 长度边界；缺失 origin 明确处理；按 origin 最长；物种字段精确匹配；不声称序列去重 |
| tests/test_inputs.py、tests/test_filter_fasta.py | 新写 | 使用标注为 synthetic 的小输入验证上述行为 |
| pyproject.toml | 新写 | Python 3 范围与依赖必须经过实际验证；不要凭源 README 填版本 |
| LICENSES/FAC-assembly-BSD-2-Clause.txt、THIRD_PARTY_NOTICES.md | 首次复制源代码时新增 | 原 BSD 全文、Jon Palmer 版权、源 URL/SHA、迁入文件及修改清单 |

## 第三批：重写组装编排

| 目标文件 | 修改要求 | 验收 |
|---|---|---|
| src/fac_cr_ngs/assembly.py | 显式 reference/assembler/outdir；扣除产物接质控再接组装；检查每个子进程；流式计数；验证参考和 paired reads | mocked command 测试确认数据流；注入子进程失败时中止；sample_id 不碰撞；隔离输出 |
| src/fac_cr_ngs/provenance.py | 参数、输入/参考 hash、版本和退出状态 | 参考或参数改变时拒绝旧缓存 |
| config/analysis.example.yaml | 待接口实现后新增 | 必填项用 null 并令校验器拒绝未配置运行；不设置未经确认的默认宿主 accession |
| environment.yml 或容器定义 | 选择实际验证的一种方式 | 固定真正跑通的工具版本并保存一次环境构建证据 |
| tests/integration/test_assembly.py | 新写 | 小型合成 paired reads 验证扣除与配对；assembler 集成测试单独运行，不以 mock 代替完整验证 |

先实现和验证一种 assembler，再扩展第二种。覆盖统计应明确用哪一阶段的 reads，并给出已知小输入的期望值。

## 第四批：末端定位与注释转换

| 目标文件 | 修改要求 | 验收 |
|---|---|---|
| src/fac_cr_ngs/end_mapping.py | 修复双端长度检查与成功退出；阈值可配；保留多命中；校验方向与区间；把定位和候选分类分离 | 正反链、同向、反序、单端、空命中、多命中、split-contig 和阈值边界 |
| tests/test_end_mapping.py | 新写 | 确保 100 N 拼接从未归为完整；合成真值核对切片与方向 |
| src/fac_cr_ngs/antismash_tsv.py | 重写表头/schema、坐标与负链序列；明确支持的 GenBank feature 模式；与 annotation runner 分离 | 表头逐列一致；首碱基与负链提取；CDS 先出现、缺失字段、多记录/重复 locus_tag、多个 cluster 重叠 |
| tests/test_antismash_tsv.py | 新写 | 用来源明确的最小 GenBank fixtures；未支持 feature 不能静默当作无 BGC |
| .github/workflows/tests.yml | 模块可运行后新增 | Python 单元测试；外部工具集成测试独立标记，记录版本 |

## 后续决策门槛

收到 CR 含义、实验池图、真实载体/宿主、样本清单与已有真值后，再制定跨池匹配、antiSMASH 运行和 BGC 筛选模块。真实数据应使用可追溯下载地址与校验值，不默认入 Git。至少一次完整真实数据运行及真值对照后才能讨论筛选准确率、速度或“robust”能力。
