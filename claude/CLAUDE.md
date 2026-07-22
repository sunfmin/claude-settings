# Global Claude config

## sudo

No tty here. Plain `sudo` fail. Use sudoplz askpass.

Prefix every sudo cmd:

```
SUDO_ASKPASS=$HOME/.local/bin/askpass sudo -A <cmd>
```

GUI dialog pop -> user approve per cmd. Deny -> cmd fail, retry not allowed.

Setup (once, user terminal):

```
brew install age
uv tool install sudoplz
sudoplz set
```

Needs: `uv`, `age`, ed25519 key at `~/.ssh/id_ed25519`.

## skills

Installed via `npx skills`. Copies at `~/.agents/skills/<name>/` (symlinked into
`~/.claude/skills/`). **Never edit there** -> `npx skills update` overwrites from
source repo, edits lost.

Source of truth: `~/.agents/.skill-lock.json`. Each entry has `source`
(e.g. `sunfmin/whats-hot`) + `skillPath`. `source` under `sunfmin/` = mine.

Mine live at `~/Developments/<repo>`, `<repo>` = `source` after `sunfmin/`
(`sunfmin/whats-hot` -> `~/Developments/whats-hot`). Git remote = same repo on GitHub.

Change my skill:

1. Lockfile -> get `source` + `skillPath`.
2. Edit `~/Developments/<repo>` (file at `skillPath`, e.g. `SKILL.md`).
3. `git commit` + `git push`.
4. `npx skills update` -> pulls into `~/.agents/skills/`.

`source` not under `sunfmin/` (mattpocock/skills, anthropics/skills,
mvanhorn/cli-printing-press) = third-party, not mine. No edit+push. Surface instead.

## dreamina credits

Every `dreamina` generation call (text2image, image2image, text2video, image2video,
frames2video, multiframe2video, multimodal2video, image_upscale ...) -> after it
returns, report credit spend as a per-step line:

```
第N步 <cmd>: 消耗 <credit_count> credits, 余额 <total_credit>
```

`credit_count` -> from the task result JSON. Balance -> `dreamina user_credit`
(`total_credit`). Multiple calls in a turn -> one line each, plus a total-consumed
summary at the end.

## dreamina video queue (vip = 快队列)

Dreamina video gen has TWO queues, gated by model tier:

- `_vip` models (`seedance2.0_vip`, `seedance2.0fast_vip`) -> **快队列 / priority**:
  reach `queue_status: Generating` in ~1-2 min, finish in a few min.
- non-vip models (`seedance2.0`, `seedance2.0fast`, `seedance2.0mini`) -> **慢队列 / free**:
  can sit at `queue_status: Queueing` for 30-40+ min, sometimes effectively stuck.

Rule: any video that must land promptly -> ALWAYS use a `_vip` model. Non-vip only for
"don't care when it finishes". `_vip` also costs more and is the only tier reaching 1080p/4K.
There is NO CLI cancel -> a slow-queue task, once submitted, can't be aborted (only ignored).

## rg, not grep

Search files -> built-in Grep tool (rg under the hood). Filter output -> pipe to `rg`.
Never shell out to `grep`/`egrep`/`fgrep` -> a global PreToolUse hook denies them.
`pgrep`, `zgrep`, `git grep` still ok.

## 大文件下载

下大文件（模型权重、数据集、release、tarball/zip、ISO、镜像、视频……）前，**先搜确认本地有没有**：

- 查 `~/.cache/huggingface`、`~/.cache/modelscope`、aria2/下载目标目录、已有产物、venv/工具自带缓存等。
- 已有 -> 直接用，**别重复下**。
- 确实缺、真需要下 -> **先和我确认**，得到同意再下。别擅自 `hf download` / `wget` / `curl -O` / aria2 / 工具首次运行触发的隐式拉取就把几个 G 拉下来。

不确定「会不会触发下载」也先说一声，别默默开下。

## auto commit

阶段性任务完成 -> 自动 `git commit`，不用等我开口。这条覆盖默认的「只有我要求才 commit」。

- Git repo only. 非 repo -> skip.
- 每个有意义的阶段 = 一个 commit：一个 feature、一个 fix、一段测试跑通、一步 refactor。别把整个 session 攒成一个大 commit。
- **只 commit，不 push。** Push 仍要我明确说。
- Message 简洁，描述这一步做了啥；保留 `Co-Authored-By` trailer。
- 分支照 harness 默认走（在默认分支上先开 branch 再 commit）。
