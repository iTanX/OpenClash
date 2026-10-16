# 自定义规则维护

两份配置模板共用以下 YAML 文件：

| 文件 | 用途 |
| --- | --- |
| `My_Direct_Classical.yaml` | 命中后直连 |
| `My_Proxy_Classical.yaml` | 命中后交给「🧩 自定义代理」策略组选择出口 |

规则文件只负责匹配条件，不填写 `DIRECT` 或策略组名称；出口在 `config/` 中的 INI 模板里指定。自定义规则位于私网规则之后、公共规则之前，按从上到下的顺序优先匹配。

## 添加规则

直接在对应文件的 `payload:` 下新增一行，使用空格缩进：

```yaml
payload:
  # 精确匹配一个域名
  - 'DOMAIN,api.example.com'
  # 匹配域名本身及其全部子域名
  - 'DOMAIN-SUFFIX,example.com'
  # 匹配域名中包含该关键词的所有地址
  - 'DOMAIN-KEYWORD,example'
```

现有三条规则保留原来的关键词匹配方式。例如 `DOMAIN-KEYWORD,333589.xyz` 也会匹配 `333589.xyz.example.com`；如果只希望匹配 `333589.xyz` 及其子域名，应改为 `DOMAIN-SUFFIX,333589.xyz`。

由于包含 `DOMAIN-KEYWORD`，INI 使用 `clash-classic:` 引用，不能直接改成 `clash-domain:`。

## 发布与生效

两个 INI 的自定义规则地址均附带 `?v=20261016-1`，这是手动缓存版本号，不会自动生成。需要再次尝试绕过缓存时，可将其改为 `?v=20261016-2`，并同步修改两个模板。推送后，如果转换后端仍缓存旧 INI，可在 OpenClash 中填写的自定义模板地址末尾也添加或递增 `?v=`；若地址已有查询参数，使用 `&v=`。版本参数通常用于避开按完整 URL 保存的缓存，实际效果取决于后端。

1. 将 YAML 和修改后的 INI 提交并推送到 `iTanX/OpenClash` 仓库的 `main` 分支，确保两份 YAML 的 Raw 地址无需登录即可访问。仅修改本地文件不会更新远程规则。
2. 在 OpenClash 中更新订阅，使用更新后的 INI 模板重新生成配置。
3. 在面板的「🧩 自定义代理」中选择出口；默认跟随「🌐 Default」，也可通过「🧭 手动全部」单独指定节点。完整模板还可直接选择地区组。
4. 查看连接详情，确认命中的规则和出口符合预期。

INI 中的 `28800` 是规则集更新间隔参数，单位为秒，即 8 小时。实际刷新取决于订阅转换后端和生成配置的方式：如果规则被展开写入最终配置，需要再次更新订阅；如果生成了远程 Rule Provider，则由内核按对应 Provider 设置刷新。首次使用前应先确保远程 YAML 已发布。

参考：[Subconverter 规则集与策略组文档](https://github.com/tindy2013/subconverter/blob/master/README-cn.md)。
