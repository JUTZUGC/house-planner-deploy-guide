# House Planner 一键部署指南

> 第三方部署指南 · 非 House Planner 官方仓库。官方项目：[egmalt/house-planner](https://github.com/egmalt/house-planner)
> 内容来自官方文档（2026-11-02）。项目迭代快，请以[官方文档](https://github.com/egmalt/house-planner/blob/main/docs/guide/en/install.md)为准。

[![⚡ 一键部署](https://img.shields.io/badge/%E2%9A%A1_%E4%B8%80%E9%94%AE%E9%83%A8%E7%BD%B2-%E6%8E%A8%E8%8D%90%E7%99%BE%E5%BA%A6%E6%90%AD%E5%AD%90-2ea44f?style=for-the-badge)](#一键部署推荐用百度搭子)

## 一键部署（推荐：用百度搭子）

> [!TIP]
> 户型图不该上传到别人的云：平面、三维、管线与报价都放在自己机器上。

⚠️ 用 Docker Compose 构建，首次 build 要装 PHP 与前端依赖，耗时较长。默认端口 8080，可用 `HOUSE_PORT` 改。首次打开要在 `/api/setup.php` 设密码。界面目前只有英文与俄文；卫星底图与部分在线资源需要联网。

1. **装客户端**
   打开[百度搭子官网](https://www.dumate.cn/)，下载电脑端并安装，用你的账号登录。

> [!IMPORTANT]
> **复制这一句发给百度搭子：**
>
> ```text
> 你好，帮我从 github 部署 JUTZUGC/house-planner-deploy-guide 到我的电脑上
> ```

2. **发送部署指令**
   回到刚打开的对话窗口，粘贴并发送上面那句。

3. **跟着对话往下走**
   百度搭子会用 Docker Compose 构建并启动服务，再打开本地面板设置访问密码，按它的提示确认即可，结束时它会告诉你结果。
   （需本机已装 Docker；未安装时按对话提示安装。）

4. **打开管理面板**
   浏览器访问 [http://localhost:8080](http://localhost:8080)，按官方引导完成初始化配置。

## House Planner 是什么？

House Planner 是开源的在自己服务器上画真实尺寸的户型图，顺手算水电暖与材料报价（协议：MIT，<img src="https://img.shields.io/github/stars/egmalt/house-planner?style=flat-square&label=stars&color=blue" alt="GitHub stars">）。

自托管住宅规划工具：毫米级 2D 平面编辑、3D 预览、卫星图叠加，以及下水、电气、给水、地暖四类管线的布置与计算，并生成带价格的物料清单。

- 真实尺寸：墙体、门窗、家具都按毫米画
- 2D + 3D：平面与三维视图随时切换
- 管线计算：下水、电气、给水、地暖各有图层
- 材料清单：按 JSON 价目表算量并导出 CSV
- 版本与快照：每次保存都留版本，可回滚
- 数据自持：方案以 JSON 存在自己的卷里

（来源：[官方仓库](https://github.com/egmalt/house-planner)）

## 优缺点

| 优点 | 说明 |
|---|---|
| 353 stars 10 月新项目 | 名称搜索 234 条但无同名部署指南 |
| 2D 平面、3D 预览与卫星图叠加 | — |
| 水电暖四类管线带计算与校验 | — |
| 物料清单可按价目表算钱并导出 CSV | — |
| 方案存本地卷 | 带版本与快照 |
| MIT | 自用与二开都自由 |

| 需要注意 | 说明 |
|---|---|
| 首次 Docker 构建耗时较长 | — |
| 界面只有英文与俄文 | — |
| 管线规则偏俄标 | 国内需自行调整 |
| 没有多用户与权限体系 | — |
| 价目表需要自己维护 | — |

> 以上基于官方文档与社区反馈整理（2026-11-02），以官方最新文档为准。

## 快速部署（官方方式）

```bash
git clone https://github.com/egmalt/house-planner.git
cd house-planner
docker compose up -d --build
echo "已启动：打开 http://localhost:8080 ，先到 /api/setup.php 设密码"
```

启动后浏览器访问 [http://localhost:8080](http://localhost:8080)（管理面板）。（来源：[https://github.com/egmalt/house-planner/blob/main/docs/guide/en/install.md](https://github.com/egmalt/house-planner/blob/main/docs/guide/en/install.md)）

## 验证状态

| 场景 | 状态 |
|---|---|
| README 与安装文档确认 Docker 方式、8080 端口与密码向导 | 已核对 2026-11-02 |
| 未实测构建与启动 | 说明 2026-11-02 |
| MCP 冒烟：initialize / tools/list 返回 get_guide、search_guide、get_project_info | 已验证 2026-11-02 |
| 非目标宿主调用 get_guide 被拦截（HOST_NOT_ALLOWED） | 已验证 2026-11-02 |

> 本表仅列出已在本机实际执行并成功的场景；未列出的部分按官方文档整理。

## MCP 接入

项目本身不含 MCP；本仓库的 MCP 只提供部署步骤、端口与常见问题。

本仓库自带一个零依赖的 MCP server（`mcp-server.mjs`），读取同目录的 `guide.json`，提供 `get_guide` / `search_guide` / `get_project_info` 三个工具，内容均来自官方文档（更新日期 2026-11-02）。

**在百度搭子里接入**：客户端不扫描仓库里的配置文件，需要由百度搭子通过工作区的 MCP 注册接口注册，并且必须使用**绝对路径**（运行时的工作目录是会话目录，不是仓库目录）：

```json
{
  "type": "local",
  "command": ["node", "<仓库绝对路径>/mcp-server.mjs"]
}
```

完成后，直接在同一个对话框里说：

> 把仓库克隆到固定目录（例如 C:\Users\<你>\house-planner-deploy-guide），再用绝对路径把里面的 mcp-server.mjs 注册到当前工作区的 MCP 配置里

> 注意：注册是按**工作区**生效的，换工作区要重新注册；克隆目录不要放在会话临时目录，否则目录被清理后注册项会失效。

**其它 MCP 客户端**：仓库根目录的 `.mcp.json` 只是通用写法（`command` + `args`），接入时同样要把 `args` 改成绝对路径。

> 使用范围：该 MCP 仅在百度搭子中可用；其它客户端调用会返回 `HOST_NOT_ALLOWED`，并附带下载与部署指引。

## 首次设置

1. 打开 `http://localhost:8080/api/setup.php` 设访问密码
2. 回到首页开始画平面
3. 需要卫星底图时，在地图面板填经纬度与旋转角

想在首次启动就定好密码，可以这样起：

```bash
HOUSE_PASSWORD='换成你的密码' docker compose up -d --build
```

## 端口与数据

| 项目 | 位置 |
|---|---|
| 端口 | `HOUSE_PORT`，默认 8080 |
| 方案、版本与配置 | Docker 卷 `house-data` |
| 价目表 | 仓库内 JSON 目录，可自行替换 |

数据在卷里，`docker compose down` 不会丢；加 `-v` 才会删。

## 开发模式

需要 Node.js 22 与 PHP 8.1+，两个终端分别跑 `npm run dev:api` 与 `npm run dev`，前端在 5173，API 在 8000。

## 注意

- 首次 `--build` 较慢，机器差的话留足时间
- 密码只对空存储生效，改密码要先删卷里的 `config.php`
- 界面只有英文与俄文，中文内容可正常录入
- 卫星图与在线底图需要联网
- MIT，可自用可二开

## 常见坑

- 8080 打不开：先看 `docker compose ps`，首次 build 未完成时还没监听
- 进不去：先访问 `/api/setup.php` 设置密码
- 改了密码不生效：密码只在空存储时写入，需删卷里的 `config.php` 再重启
- 端口冲突：用 `HOUSE_PORT` 换一个再起
- 数据丢了：别用 `docker compose down -v`
- 卫星图不显示：检查网络与经纬度、旋转角是否填对
- 管线算不出量：先补齐墙高、楼层与价目表字段

## 免责声明

本仓库为第三方指南，与 House Planner 官方无隶属关系，不代表官方立场。
教程基于官方文档编写（更新日期：2026-11-02），项目迭代快，请以[官方文档](https://github.com/egmalt/house-planner/blob/main/docs/guide/en/install.md)为准。
因使用本教程产生的部署问题，请先在官方仓库的 issue 中检索。
