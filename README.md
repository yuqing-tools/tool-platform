# 内部工具平台

部门共享工具平台，纯网页实现，无需服务器。

## 访问地址

```
https://yuqing-tools.github.io/tool-platform/
```

## 功能特点

- **纯网页运行**：不需要安装任何软件，打开网页就能用
- **本地处理**：所有数据在浏览器中处理，不上传到服务器
- **无需登录**：任何人都可以访问使用
- **添加新工具**：提交 Pull Request 审核后上线

## 使用方法

1. 打开网页
2. 选择需要的工具
3. 按提示上传数据文件
4. 点击处理，等待结果
5. 下载处理完成的文件

## 添加新工具

### 1. 创建工具文件夹

在 `tools/` 目录下创建新工具文件夹，例如：
```
tools/my-tool/
```

### 2. 创建 config.json

```json
{
    "id": "my-tool",
    "name": "我的工具名称",
    "icon": "🔧",
    "description": "工具描述",
    "version": "v1.0",
    "author": "作者名",
    "inputHint": "上传文件提示",
    "config": {
        "配置项1": "值1",
        "配置项2": "值2"
    }
}
```

### 3. 提交代码

1. Fork 本仓库
2. 添加你的工具
3. 修改 `index.html` 中的 `TOOLS_CONFIG` 数组，添加你的工具配置
4. 提交 Pull Request
5. 管理员审核后合并上线

## 目录结构

```
tool-platform/
├── index.html              # 主页面
├── README.md               # 说明文档
└── tools/
    └── mengniu-sentiment/ # 舆情工具示例
        └── config.json     # 工具配置
```

## 工具列表

| 工具名称 | 版本 | 作者 | 功能 |
|---------|------|------|------|
| 蒙牛舆情数据整理工具 | v3.6 | EDY | 舆情数据自动分类整理 |

## 技术栈

- 前端：原生 HTML/CSS/JavaScript
- Excel 处理：SheetJS (xlsx)
- 托管：GitHub Pages

## 注意事项

- 工具处理在本地浏览器完成，请放心使用
- 如有配置问题请联系管理员
