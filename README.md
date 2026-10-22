# xblack 赛车图鉴（xinjiemian）

QQ 飞车道具图鉴的纯静态前端：搜索、分类筛选、分页、排序、卡片详情与右键复制。
页面本身不含数据，所有条目通过同源 `/api` 接口按需拉取，图片直接引用
`https://iips.speed.qq.com/images/{id}.png`。

## 目录结构

```
.
├── index.html        # 单页应用：样式 + 交互逻辑全部内联
├── sw.js             # Service Worker：仅做缓存版本清理，不做离线缓存
├── web.config        # IIS 配置：API 反向代理 + 原始数据文件封禁
├── logo_annual.png   # 横幅 logo（452×65）
└── logo_web.png      # 站点小 logo（102×75）
```

## 后端接口契约

前端固定请求以下两个接口（同源，`cache: no-store`）：

### `GET /api/meta`

```json
{
  "pageSize": 60,
  "total": 122,
  "categories": [{ "value": "S级赛车", "count": 10 }],
  "quickTabs": [{ "key": "S", "label": "S", "count": 10 }]
}
```

- `quickTabs[].key` 取值：`ALL`、`skin`、`S`、`A`、`B`、`C`、`D`、`T3`、`T2`、`T1`、`suit`、`pet`
- 缺省时前端会回退到内置的 12 个页签

### `GET /api/items?page=1&page_size=60&sort=desc&cat=ALL&q=`

```json
{
  "items": [
    {
      "id": 100039,
      "name": "霜刃·星轨",
      "type": "A级赛车",
      "slot": "a",
      "desc": "描述文本",
      "usage": "竞速模式可用"
    }
  ],
  "page": 1,
  "pageSize": 60,
  "total": 122,
  "pages": 3
}
```

- `sort`：`desc`（最新优先）/ `asc`
- `cat`：`ALL` 或 `quickTabs` 里的 key
- `q`：名称 / ID / 类型 / 描述的关键词
- `slot` 控制卡片主题色：`s`、`a`、`b`、`skin`、`suit`、`pet`、`box`

## 部署（IIS）

`web.config` 做了三件事：

1. `urlCompression` 开启静态压缩；
2. 把 `/api/*` 反向代理到 `http://127.0.0.1:8000/api/*`；
3. 对 `xblack_data.bin`、`xblack_data.js` 直接返回 403，避免原始数据被下载。

部署时把本目录设为站点根目录，并保证本机 `8000` 端口有配套后端在跑，
同时安装 IIS 的 **URL Rewrite** 与 **ARR（Application Request Routing）** 模块并启用代理。

## 说明

前端不包含后端与数据文件；单独打开 `index.html` 会因 `/api/meta` 不可用而提示
`Load failed`。预览需要自行提供上述接口。
