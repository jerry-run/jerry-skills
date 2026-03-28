# jerry-skills

个人编制的 Claude Code Agent Skills 合集，用于发布和分享自定义技能。

## 包含技能

### opt-skill

**用途**：审计并优化现有 Agent Skill 的 `SKILL.md` 文件，使其符合最佳实践。

**触发方式**：`/opt-skill <skillpath>`

对目标 `SKILL.md` 进行 12 项最佳实践审计，列出问题并在用户确认后自动重写，覆盖范围包括：内容结构、详细程度、输出模板、多步骤工作流 Checklist、验证循环等。原文件会备份至 `<skillpath>/optimize/original_SKILL.md`。

---

### opt-skill-desc

**用途**：通过迭代 Eval 循环自动优化 `SKILL.md` 的 `description` 字段，使技能触发更精准可靠。

**触发方式**：`/opt-skill-desc <skillpath>`

自动生成 20 条标注查询（10 条应触发 / 10 条不应触发），通过训练集迭代修改描述，直到整体通过率 ≥ 90%，最终展示前后对比并在用户确认后写回 `SKILL.md`。
