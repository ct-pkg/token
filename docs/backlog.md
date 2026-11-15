# 包外移入事项登记（EXT-TOK-*）

- 日期：2026-11-15
- 来源：CT 仓移出的「包外事项」清单（原位置 `提案 §10 S4`）。本仓只认领其中属于 `ct-token` 的
  `EXT-TOK-01`、`EXT-TOK-02` 两条；此后这两条的文档完整性与任务完整性由本仓自负，不再回写 CT 仓。
- 范围：只做认领与状态登记。不改 `ct.pkg`、不打 tag、不动版本号；条目只写能力前置，不引用 CT 版本号。
- 状态词：`blocked`（缺前置能力）、`dormant`（仅规格占位，不产包）、`pending-decision`（待 owner 拍板）。

## 1 前置能力

| 编号 | 能力 | 现状 | 阻塞的条目 |
| --- | --- | --- | --- |
| C1 | `ct get` 落 store 并读 `publisher` | 未落地 | EXT-TOK-01 |
| C2 | `ns-root` 进 `-I` | 未落地 | EXT-TOK-01 |
| C3 | `ct build` 默认读 lock | 未落地 | EXT-TOK-01 的构建/安装验收（不阻塞声明本身） |

C1/C2 落地前，`publisher`、`ns-root` 只是清单里的待认字段，读取方无法验证；C3 落地前，构建侧只能
显式传 `-I`，不能作为「装好就能编」的证据。

## 2 认领条目

| ID | 来源位置 | 要做什么 | 前置能力 | 验收方式 | 状态 |
| --- | --- | --- | --- | --- | --- |
| EXT-TOK-01 | 提案 §10 S4 | 补 `publisher`、`ns-root` 两键；依赖声明改用全名 `<publisher>/<name>@<range>` | C1、C2（构建验收另需 C3） | 见 2.1 | `blocked`（C1/C2 未落地） |
| EXT-TOK-02 | 提案 §10 S4 | 二选一：明确「规格占位」并登记 dormant，或补公开单元；仅规格者不得打空包 tag | `ct pkg check` 可用（当前未落地） | 见 2.2 | `dormant`（本仓尚无公开单元） |

### 2.1 EXT-TOK-01 验收

- 正向：`ct.pkg` 同时含 `publisher` 与 `ns-root`；读取方（`ct get` / `ct pkg check`）解析无告警。
- 正向：`ns-root` 指向的目录能作为 `ct build -I <dir>` 的根解析到包内单元（C2 落地后可测）。
- 依赖全名：当前本包零依赖（`grep -n '^dep *=' ct.pkg` 0 命中），全名规则作为将来新增依赖时的硬约束；
  一旦有 `dep`，必须写 `<publisher>/<name>@<range>`，短名即失败。
- 负例 1：删掉 `publisher`，读取方必须非零退出并报缺失键（文案以 C1 实现为准）。
- 负例 2：把 `ns-root` 指到不含单元的目录，`ct build` 必须无法解析包内单元。
- 负例 3：把 `dep` 写成短名（`name@range`），解析必须失败。
- 不在本轮：不改 `ct.pkg`、不提交草稿、不打 tag。

### 2.2 EXT-TOK-02 验收

- 形态 (a)「规格占位」：本文件保留 `dormant` 记录，仓库内 `git tag --list` 不出现本包的空 tag；
  恢复条件写在本条状态栏，改动状态需 owner 拍板。
- 形态 (b)「补公开单元」：包根出现 `*.ct` / `*.h.ct`，`ct pkg check` 退出 0 且识别到至少一个公开单元。
- 负例：在没有任何单元时打 tag → 视为违约，按 Issue 登记（tag 的撤回属对外操作，只由 owner 决定）。
- `ct pkg check` 报缺单元时的处置：维持形态 (a)，只在本文件记录，不补空包、不打 tag。
- 不在本轮：不写实现、不建 `tools/check.sh`。

## 3 待决项：工作区未提交改动

以下改动已存在于工作区，按指令**既不提交也不丢弃**，仅登记：

| 文件 | 未提交内容 | 与认领的关系 | 状态 |
| --- | --- | --- | --- |
| `ct.pkg` | `publisher = ct`、`ns-root = .`、`requires` 下限上调及注释改述 | 属 EXT-TOK-01 的草稿，前置 C1/C2 未落地 | `pending-decision` |
| `docs/requirements.md` | R-01 输入/期望、R-01/R-20 的 `requires` 行、R-01/R-20 两处「依赖 / 阻塞」说明 | 与本轮文档一致性相关，非认领条目本身 | `pending-decision` |

- 本轮提交只含本文件与 `AGENTS.md` 的指针行；不使用 `git add -A`，不带入上述两个文件。
- 草稿是否保留、何时提交，见 §5 owner 拍板点。

## 4 文档自相矛盾（只标出，未修）

| 位置 | 现文 | 冲突对象 | 建议处置 |
| --- | --- | --- | --- |
| `docs/requirements.md:11` | 「本机没有 `ct` / `ct-ast`（`command -v ct ct-ast` 均无输出）」 | 同文件 `:92`、`:398` 已写本机 `ct` 在 `/Users/samxx/.local/bin/ct`（`ct v0.44.1`），只是不在 PATH | 统一为「有 `ct`、需绝对路径调用」 |
| `docs/requirements.md:467` | §7 表「本机 `ct` / `ct-ast`：`command -v ct ct-ast` 无输出，**软阻塞**」 | 同上两条 | 改成「需绝对路径调用；安装验证仍受包源限制」 |
| `docs/requirements.md:375` | R-19「`AGENTS.md` 的 1.9.0 生成块由 `sync-agents.sh` 维护」 | `AGENTS.md:8` 块头为 `version=1.10.0`；`docs/spec/ai-coding-workflow.md:5` 亦为 `1.10.0` | 改为「当前 pin 的生成块」或写 1.10.0 |
| `docs/requirements.md:5` | 「空仓：`main` 零提交」 | 本仓 `main` 已有提交（需求层与规范升级两次） | 改为已落库的实际状态 |

注：`:80`、`:394` 的 `requires` 行与 `ct.pkg` HEAD 的旧下限不一致，原因是工作区有未提交改动（§3），
属待决项而非文档笔误。

## 5 待 owner 拍板

1. `publisher` 取值：草稿写 `ct`，是否就是本包发布者标识？
2. `ns-root` 取值：草稿写 `.`（单元根即包根），是否与本仓后续目录布局一致？
3. EXT-TOK-01 草稿（§3 的 `ct.pkg` 改动）是保留到 C1/C2 落地，还是先回退、等能力齐了重做？
4. EXT-TOK-02 维持 `dormant`（本仓目前无公开单元）是否认可？若要转「补单元」，需另开实现范围。
5. 本轮只登记、不修 §4 的矛盾；是否下一轮单独修文档（会改 `docs/requirements.md`，与 §3 的未提交改动同文件）。
6. `dormant` / `pending-decision` 是否需要同步进 `AGENTS.md` 的 Issue 台账（目前该台账未建）。
