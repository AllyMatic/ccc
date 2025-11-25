# CCC (Claude Code Changelog)

一个使用 Claude AI 自动生成产品更新说明的 GitHub Action。

## 简介

CCC 是一个 GitHub Action，基于官方的 [Claude Code Action](https://github.com/anthropics/claude-code-action) 构建，能够自动分析代码变更并生成面向客户的产品更新说明。

## 特性

- 🤖 **AI 智能生成** - 使用 Claude AI 分析代码变更，生成易读的更新说明
- 📝 **自动化分析** - 自动对比 tag 之间的所有提交
- 🔄 **多种认证方式** - 支持 Anthropic API、AWS Bedrock、Google Vertex AI
- 🎯 **客户友好** - 生成的内容直接面向客户，无技术术语
- ⚡ **开箱即用** - 默认配置即可运行

## 使用方法

### 基础用法

在你的仓库中创建 `.github/workflows/changelog.yml` 文件：

```yaml
name: 生成更新说明
on:
  push:
    tags:
      - 'v*'

jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 需要完整的 git 历史

      - name: 生成更新说明
        id: changelog
        uses: allymatic/ccc@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: 查看结果
        run: echo "${{ steps.changelog.outputs.result }}"
```

### 配合 Release 使用

```yaml
name: 发布新版本
on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 生成更新说明
        id: changelog
        uses: allymatic/ccc@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: 创建 Release
        uses: actions/create-release@v1
        with:
          tag_name: ${{ github.ref_name }}
          release_name: ${{ github.ref_name }}
          body: ${{ steps.changelog.outputs.result }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 使用 AWS Bedrock

```yaml
- name: 生成更新说明
  uses: allymatic/ccc@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    use_bedrock: true
    bedrock_region: 'us-west-2'
    model: 'anthropic.claude-3-5-sonnet-20241022-v2:0'
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 使用 Google Vertex AI

```yaml
- name: 生成更新说明
  uses: allymatic/ccc@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    use_vertex: true
    vertex_project_id: 'your-gcp-project'
    vertex_region: 'us-central1'
  env:
    GOOGLE_APPLICATION_CREDENTIALS: ${{ secrets.GOOGLE_APPLICATION_CREDENTIALS }}
```

## 配置参数

| 参数 | 说明 | 默认值 | 必填 |
|------|------|--------|------|
| `github_token` | GitHub Token | `${{ github.token }}` | 是 |
| `anthropic_api_key` | Anthropic API Key | - | 否* |
| `from_tag` | 起始 tag | 最近的 tag | 否 |
| `to_ref` | 目标引用 | `HEAD` | 否 |
| `model` | Claude 模型 | `claude-sonnet-4-5-20250929` | 否 |
| `use_bedrock` | 使用 AWS Bedrock | `false` | 否 |
| `use_vertex` | 使用 Google Vertex AI | `false` | 否 |
| `bedrock_region` | AWS Bedrock 区域 | `us-east-1` | 否 |
| `vertex_project_id` | GCP 项目 ID | - | 否** |
| `vertex_region` | GCP 区域 | `us-central1` | 否 |

\* 不使用 Bedrock 或 Vertex AI 时必填  
\*\* 使用 Vertex AI 时必填

## 输出

| 输出 | 说明 |
|------|------|
| `result` | 生成的更新说明内容 |
| `from_tag` | 起始 tag |
| `to_tag` | 目标 tag |

## 工作原理

1. 获取两个 tag 之间的所有 commit 信息
2. 使用 Claude AI 分析变更内容
3. 生成面向客户的产品更新说明

## 自定义提示词

提示词模板位于 `prompts/changelog_prompt.md`，你可以 fork 本仓库后自行修改。

## 许可证

MIT License
