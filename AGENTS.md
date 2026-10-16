# Repository Guidelines

## Project Structure & Module Organization

Firefly is an Astro 7 site with Svelte islands and TypeScript configuration. Main source code lives in `src/`: routes in `src/pages`, layouts in `src/layouts`, reusable UI in `src/components`, styles in `src/styles`, content in `src/content`, helpers in `src/utils`, and Markdown/HTML plugins in `src/plugins`. Site configuration is split across `src/config` with matching type definitions in `src/types`; prefer imports from `@/config` when available. Static files served directly belong in `public`, source-managed images in `src/assets`, docs in `docs` and `Firefly-Docs`, and automation in `scripts`.

## Build, Test, and Development Commands

Use `pnpm`; the `preinstall` script enforces it.

- `pnpm dev` or `pnpm start`: run the local Astro dev server.
- `pnpm check`: run Astro diagnostics.
- `pnpm type-check`: run TypeScript with `--noEmit`.
- `pnpm format`: format `src` with Biome.
- `pnpm lint`: run Biome checks and safe fixes on `src`.
- `pnpm build`: generate icons, LQIPs, the Astro build, font subsets, and Pagefind search output in `dist`.
- `pnpm preview`: preview the production build locally.
- `pnpm new-post`: scaffold a new content post.

## Coding Style & Naming Conventions

Biome is the formatter and linter. It uses tabs for indentation and double quotes for JavaScript/TypeScript strings. Keep Astro and Svelte components in `PascalCase` (`PostCard.astro`, `Search.svelte`), config modules in `camelCase` ending with `Config.ts`, and utilities in descriptive kebab case such as `date-utils.ts`. Keep `src/types` aligned with `src/config`. Avoid unrelated formatting churn.

## Testing Guidelines

There is no dedicated unit-test framework configured. Before submitting changes, run `pnpm check`, `pnpm type-check`, and `pnpm build` for rendering, content, or generated asset work. For visual or interactive changes, verify with `pnpm dev` or `pnpm preview` and include screenshots in the PR. Name future tests near the feature they cover, using the local file name as the stem.

## Commit & Pull Request Guidelines

Use Conventional Commits, matching the current history: `feat: ...`, `fix: ...`, and `chore: ...`. Keep commits and PRs focused on one concern. PRs should include a concise summary, linked issues when relevant, validation commands run, and screenshots for UI changes. Discuss major features or design changes in an issue or discussion before implementation.

## Security & Configuration Tips

Do not commit secrets, tokens, or service keys in config files. Keep deployment-specific settings in the target platform environment, and review generated files such as `dist`, `src/constants/lqips.json`, and `src/constants/icons.ts` before committing them.

## 部署链路（本项目定制，2026-10 起）

Cloudflare **Workers Build（Git 集成）已被账号封禁**，故改用 **GitHub Actions + `wrangler@4`** 直传静态资源。两段式：

1. **源仓库 `orchidsd/Firefly`**（master）push → `.github/workflows/deploy-artifacts.yml`：`pnpm install` → `pnpm run build` → 把 `dist/`（含复制进去的 `wrangler.toml` 与 `.github/workflows/deploy.yml`）用 `peaceiris/actions-gh-pages` 推到**产物仓库 `orchidsd/firefly-artifacts`**（`publish_branch: main`、`force_orphan: true`、`exclude_assets: ''`、`personal_token`）
2. **产物仓库 `orchidsd/firefly-artifacts`** 的 `.github/workflows/deploy.yml`：`npx wrangler@4 deploy` → Cloudflare Worker **`firefly-blog`**，自定义域名 `fire.weiguang.eu.org`（绑在 Worker 名上，重部署自动保留，**无需改 DNS**）

所需 secrets：
- 源仓库：`ARTIFACTS_REPO`（值 `orchidsd/firefly-artifacts`）、`ARTIFACTS_REPO_TOKEN`（PAT，可推产物仓）、`CF_ACCOUNT_ID`、`CF_API_TOKEN`
- 产物仓库：`CF_ACCOUNT_ID`、`CF_API_TOKEN`

> ⚠️ 两个坑：① 产物仓的部署工作流必须**随产物一起推送**（`.github/artifacts/deploy.yml` 复制进 `dist/`，`exclude_assets: ''`），否则产物仓没有部署任务；② 推送必须用 **PAT**（`personal_token`），用默认 `GITHUB_TOKEN` 推不会触发产物仓 Actions。
> `.github/workflows/deploy.yml`（GitHub Pages）为遗留任务，与本链路无关且常失败，可忽略。
> 本机查 Actions：直连 `api.github.com` 返回 `000`（被墙），须用环境变量 `GITHUB_PERSONAL_ACCESS_TOKEN` + `HTTPS_PROXY=http://127.0.0.1:10808` 走代理。线上验证带 cache-bust：`curl.exe -sL "https://fire.weiguang.eu.org/<路径>?cb=<随机>"`。
