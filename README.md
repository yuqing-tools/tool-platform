# 内部工具平台

部门共享工具集，统一管理、便捷使用。

## 工具列表

### 蒙牛舆情数据整理工具
- **版本**: v3.6
- **功能**: 自动分类整理舆情数据，支持蒙牛/伊利/竞品/行业分类
- **下载**: [舆情数据整理工具.exe](./tools/mengniu-sentiment/舆情数据整理工具.exe)
- **配置**: [config.json](./tools/mengniu-sentiment/config.json)

## 部署说明

本项目使用 GitHub Pages 托管，静态部署。

### 访问方式
1. 直接打开 `index.html` 文件（本地预览）
2. 部署后访问: `https://[组织名].github.io/[仓库名]/`

### 添加新工具
1. 在 `tools/` 目录下创建新工具文件夹
2. 添加 `config.json` 配置文件
3. 放入 exe 文件
4. 修改 `index.html` 中的 `TOOLS_CONFIG` 配置

## 管理员操作

### 修改工具配置
1. 进入对应工具目录
2. 修改 `config.json` 文件
3. 提交到 GitHub

### 上传新工具
1. 将 exe 放入 `tools/[工具名]/` 目录
2. 更新 `config.json`
3. 提交并推送到 GitHub

## 目录结构

```
tool-platform/
├── index.html              # 主页面
├── tools/
│   └── mengniu-sentiment/ # 舆情工具
│       ├── 舆情数据整理工具.exe
│       └── config.json
└── README.md
```
