电子科技大学 示范性微电子学院
微处理器与嵌入式系统设计 课程实验 2024

# GitHub Pages 部署

- 仓库根目录已添加 `.nojekyll`，避免 Jekyll 处理。
- 已添加工作流文件 `/.github/workflows/pages.yml`，推送到 `main` 或 `master` 分支会自动部署。
- 首次启用：到 GitHub 仓库的 `Settings` -> `Pages`，选择 `Source: GitHub Actions`。
- 部署完成后访问：`https://<你的GitHub用户名>.github.io/<仓库名>/`。
- 当前 `index.html` 使用相对路径，例如 `wq2024/`、`board2024.png`，可直接在子路径下正常工作。

# 本地预览

- 在仓库根目录运行：`python3 -m http.server 8080`，访问 `http://localhost:8080/` 预览。

# 结构说明

- 根页面：`index.html`
- 年度内容：`wq2022/`、`wq2023/`、`wq2024/`，内含静态 `html`/`md` 文件与资源。
- 若仓库名或 Pages 基址变更，保持所有链接为相对路径即可适配。
