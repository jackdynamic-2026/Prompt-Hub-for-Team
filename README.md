一个给团队用的 Prompt 与 Skills 资料库。把常用提示词、Skill 链接和使用心得放在一起，登录后共享，按标签查找。

成员可以补充共同备注、推荐好用的条目，也可以把常用内容加入自己的收藏。资料支持 TXT 导入导出，方便整理和分享。

## 能做什么

| 功能 | 说明 |
|---|---|
| 两类资料 | Prompt 保存提示词正文；Skills 保存名称、介绍、GitHub 链接和心得 |
| 查找内容 | 按名称、标签、分享人、更新时间和个人收藏筛选 |
| 协作整理 | 登录成员可以编辑共同备注、追加标签；分享人和管理员可以删除标签 |
| 收藏与推荐 | 收藏只影响自己；团队推荐显示成员推荐的总数 |
| GitHub 仓库资料 | 手动刷新仓库 Star；仓库介绍可追加到草稿，核对保存后生效 |
| 版本与回收站 | 查看历史、恢复允许操作的版本，管理员管理回收站 |
| TXT 交换 | 支持完整格式、Skill 简洁格式和旧 Prompt 格式，先预览再确认导入 |
| 账号设置 | 管理员创建成员；首次改密可选；成员可设置无操作超时时间 |

Skills 只保存资料，不上传或执行代码，也不接入 OpenClaw。页面显示的 GitHub Star 是整个仓库的数量，不是某个 Skill 子目录的星数。平台不会自动运行提示词或调用模型。

## 两个入口

| 地址 | 用途 | 数据位置 |
|---|---|---|
| `/team/` | 团队共享库，需要登录 | 服务端 SQLite 数据库 |
| `/` | 保留的个人提示词工具 | 当前浏览器的 localStorage |

两个入口的数据不会自动同步。从个人版转到团队版时，需要先导出，再导入并核对。

## 本地跑起来

下面以 Windows PowerShell 为例。推荐 Node.js 24 和 Python 3.13；本地验证使用过 Python 3.13.5。进入包含 `package.json` 和 `server` 的项目根目录后操作。

源码检出需要先构建前端。如果使用交付包，也应确认包内是否已经包含 `dist/index.html`。

```powershell
npm ci
npm run build
```

首次运行，创建独立的 Python 环境，安装依赖并初始化管理员：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r server/requirements.lock.txt
$env:PH_PRODUCTION='0'
$env:PH_LOCAL_HTTP='1'
$env:PH_DATA_DIR=Join-Path $PWD 'server\data'
.\.venv\Scripts\python.exe server/manage.py migrate
.\.venv\Scripts\python.exe server/manage.py createsuperuser
.\.venv\Scripts\python.exe server/manage.py runserver 127.0.0.1:8891 --noreload
```

逐条执行，任何一步报错都先停下来处理。浏览器打开 `http://127.0.0.1:8891/team/`，用刚创建的管理员登录。项目不提供默认账号或默认密码。

以后启动时，不需要重建环境或重复创建管理员：

```powershell
$env:PH_PRODUCTION='0'
$env:PH_LOCAL_HTTP='1'
$env:PH_DATA_DIR=Join-Path $PWD 'server\data'
.\.venv\Scripts\python.exe server/manage.py runserver 127.0.0.1:8891 --noreload
```

在运行服务的终端按 `Ctrl+C` 停止。端口被占用时，先确认原来的服务，不要同时启动多份。已有数据的项目升级前先备份，不能把“首次运行”当作升级步骤直接照做。

## 开始使用

管理员在“成员管理”中创建账号，成员登录后选择“提示词”或“Skills”，点击分享按钮录入内容。点击条目卡片查看详情，复制资料、编辑备注、添加标签或推荐给团队。

导入 TXT 时，先选择对应格式，解析并核对预览，再确认导入。导入只创建新副本，不覆盖已有条目。出错时会保留输入；如果提示写入结果未确认，先查询结果，不要连续提交。

更多操作见 [使用说明](docs/USAGE.md)。

## 技术与部署

前端使用 React、TypeScript 和 Vite，后端使用 Django，数据保存在 SQLite。页面有手机布局。

`runserver` 只用于本地测试。团队公网访问需要部署后端、配置 HTTPS、生产密钥、允许访问的 Host 和可信反向代理，并安排备份。项目的 `deploy` 目录提供部署模板；模板需要按实际环境核验。

GitHub Pages 只托管静态内容，不能运行 Django 团队后端。Windows、Linux 及容器部署的验证范围，以对应版本的验收记录为准。


TXT 用于交换资料，不包含账号密码、个人收藏或完整历史，不能代替整库备份。项目提供加密备份和恢复命令；备份文件与密钥应分开保管，并实际验证恢复。

数据库、备份密钥、生产配置和运行日志不要提交到 GitHub。仓库 Star 或资料刷新状态也不代表 Skill 代码已经过安全审查。
