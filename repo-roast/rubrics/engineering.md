# 工程化评分标准

## S — 标杆级

- 测试覆盖率 >80%，断言有意义
- 所有 I/O 边界有错误处理
- 文档全面：README + API 文档 + 架构文档 + CONTRIBUTING 指南
- 配置全部外部化，有校验
- 语义化版本 + 详细 CHANGELOG
- Git 历史干净：conventional commits，PR 粒度合理

## A — 优秀

- 测试覆盖率 >60%
- I/O 边界有错误处理
- README 完善 + 基本 API 文档
- 环境变量/配置文件管理
- 有版本号和 CHANGELOG

## B — 良好

- 有测试但覆盖率 <40%
- 非关键路径错误处理有遗漏，偶有空 catch
- README 覆盖 setup 和 usage
- 配置部分外部化
- 使用 git tag 管理版本

## C — 及格

- 测试很少（<10% 覆盖）
- 错误处理不一致，有空 catch 块
- 只有 README，且内容不完整
- 存在硬编码配置值
- 发版不规则，无 CHANGELOG

## D — 不及格

- 零测试
- 异常被系统性吞掉
- 无文档
- 全部硬编码
- 无版本管理
- commit message 像 "fix"、"update"、"wip"
