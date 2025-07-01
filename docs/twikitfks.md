# twikitFKS: 基于 twikit 2.3.1 的改进版本

## 项目概述

### 关于 twikit

[twikit](https://github.com/d60/twikit) 是一个优秀的 Twitter API 爬虫库，它的核心优势在于**无需 API Key**即可访问 Twitter 功能。通过网页爬虫技术，twikit 实现了：

- 发布推文和媒体内容
- 搜索推文和用户
- 获取热门话题趋势
- 发送私信
- 用户信息获取
- 书签管理等功能

这种无 API Key 的设计让开发者可以快速构建 Twitter 相关应用，而无需申请官方 API 权限。

### twikitFKS 的诞生

在使用原版 twikit 的过程中，我们发现了一个影响数据完整性的问题：**对话分割（conversation spit）**。当 Twitter 返回对话形式的推文时，原版在处理 `profile-conversation` 类型的数据结构时存在解析缺陷，导致部分推文数据丢失。

**twikitFKS** 正是为了解决这个问题而生的改进版本。我们不仅修复了这个关键 bug，还优化了项目结构，使其更符合现代 Python 项目的最佳实践。

## 项目信息

- **项目名称**: twikitFKS
- **版本**: 2.3.1
- **Python 版本要求**: >= 3.12
- **PyPI 包**: [twikitfks](https://pypi.org/project/twikitfks/)
- **GitHub 仓库**: [Liaofushen/twikitFKS](https://github.com/Liaofushen/twikitFKS)
- **许可证**: MIT

## 主要改动

### 1. 修复对话分割问题 (Conversation Spit Fix)

在 commit `b860e7a4570fdaa0cd3c08eb574ef5f8827bcc7c` 中，我们修复了一个影响数据完整性的关键问题。

**问题分析**: 
Twitter 的 GraphQL API 在返回用户推文时，对于对话类型的推文会使用特殊的 `profile-conversation` 结构。原版 twikit 在处理这种嵌套数据结构时，只取第一个结果，导致对话中的其他推文被忽略。

**技术方案**: 
- 重构 `tweet_from_data` 函数，增加 `index` 参数以支持多推文解析
- 针对 `profile-conversation` 类型实现专门的遍历逻辑
- 添加类型过滤，确保只处理有效的 Tweet 对象

```python
# 修改前
def tweet_from_data(client: GuestClient, data: dict) -> Tweet:
    tweet_data_ = find_dict(data, 'result', True)
    if not tweet_data_:
        return None
    tweet_data = tweet_data_[0]

# 修改后  
def tweet_from_data(client: GuestClient, data: dict, index: int = 0) -> Tweet:
    tweet_data_ = find_dict(data, 'result', index == 0)
    if tweet_data_ and index > 0:
        tweet_data_ = [ _i for _i in tweet_data_ if _i.get('__typename') == 'Tweet']
    if not tweet_data_ or len(tweet_data_) <= index:
        return None
    tweet_data = tweet_data_[index]
```

### 2. 项目结构现代化

为了提升项目的可维护性和开发体验，我们进行了以下改进：

- **Python 版本管理**: 添加 `.python-version` 文件，明确指定 Python 3.12+ 要求
- **构建系统升级**: 使用 `pyproject.toml` 替代传统的 setup.py，符合 PEP 518 标准
- **Git 配置优化**: 更新 `.gitignore` 规则，排除 `__pycache__` 等临时文件
- **包管理器升级**: 集成 uv 包管理器，使用 `uv sync` 命令进行依赖同步，提供更快的依赖解析和安装速度

### 3. 依赖管理标准化

采用 `pyproject.toml` 作为统一的依赖管理配置：

```toml
[project]
name = "twikitfks"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "beautifulsoup4>=4.13.4",
    "filetype>=1.2.0",
    "httpx[socks]>=0.28.1",
    "js2py-3-13>=0.74.1",
    "lxml>=5.4.0",
    "m3u8>=6.0.0",
    "pyotp>=2.9.0",
    "webvtt-py>=0.5.1",
]
```

## 安装和使用

### 从 PyPI 安装

```bash
pip install twikitfks
```

### 从源码安装

```bash
git clone https://github.com/Liaofushen/twikitFKS.git
cd twikitFKS
uv sync
```

## 使用示例

```python
import asyncio

from twikit.guest import GuestClient

client = GuestClient()


async def main():
    # Activate the client by generating a guest token.
    await client.activate()

    # Get user by screen name
    user = await client.get_user_by_screen_name('elonmusk')
    print(user)
    # Get user by ID
    user = await client.get_user_by_id('44196397')
    print(user)


    user_tweets = await client.get_user_tweets('44196397')
    print(user_tweets)

    tweet = await client.get_tweet_by_id('1519480761749016577')
    print(tweet)

asyncio.run(main())
```

## 与原版 twikit 的技术差异

| 特性 | twikit 原版 | twikitFKS |
|------|-------------|-----------|
| 对话分割处理 | ❌ 存在数据丢失 | ✅ 完整解析 |
| 项目配置 | setup.py | pyproject.toml |
| 包管理器 | pip | uv (更快) |
| Python 版本 | 未明确指定 | >= 3.12 |
| 依赖锁定 | requirements.txt | uv.lock |

**核心改进**:
- **数据完整性**: 修复了 `profile-conversation` 类型推文的解析缺陷
- **开发体验**: 现代化的项目结构和工具链
- **性能优化**: 更快的依赖解析和安装速度

## 贡献和反馈

如果您在使用过程中遇到任何问题或有改进建议，欢迎：

- 在 [GitHub Issues](https://github.com/Liaofushen/twikitFKS/issues) 中报告问题
- 提交 Pull Request 贡献代码
- 给项目点个 ⭐️ 支持

## 许可证

本项目基于 MIT 许可证开源，您可以自由使用、修改和分发。

---

## 总结

twikitFKS 在保持原版 twikit 所有功能的基础上，重点解决了数据完整性问题，并采用了现代化的项目结构。对于需要处理 Twitter 对话数据的开发者来说，这是一个值得考虑的升级选择。

*twikitFKS 致力于提供更稳定、更完整的 Twitter API 访问体验。*
