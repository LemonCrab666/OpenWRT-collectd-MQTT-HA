
# OpenWRT-collectd-MQTT-HA
使用此 MQTT 模板从您的路由器收集 OpenWRT 统计数据。

## 设置
### Home Assistant MQTT 设置
让我们先准备 Home Assistant。

如果您还没有，请启动并运行 Mosquitto Broker。 阅读 ![官方文档](https://github.com/home-assistant/addons/blob/174f8e66d0eaa26f01f528beacbde0bd111b711c/mosquitto/DOCS.md) 以开始使用。不要忘记配置 MQTT 集成！

### OpenWRT

安装以下软件包：

    luci-app-statistics 
    collectd-mod-mqtt

将安装‘collectd’包和大多数基本依赖项。

Optional `collectd-mod-*` packages can provide more data. These are recommended:
可选`collectd-mod-*`包可以提供更多数据。以下是建议安装：

    collectd-mod-thermal
    collectd-mod-uptime
    collectd-mod-dhcpleases
    collectd-mod-conntrack

导航到‘ 统计 > 设置 ’，如果您安装了可选的 mod 包，请在 通用插件 选项卡中启用它们。

如果您从 master 分支（24.x 及更高版本）运行 OpenWRT 快照，则 luci-app-statistics [能供](https://github.com/openwrt/luci/commit/8bf5646459e229c1d01736f7c45f3b1c9bf3058f) 通过 GUI 配置 MQTT。如果您运行的是 OpenWRT 23.05 之前的版本，则必须按照 CLI 步骤操作。下一个主要版本将内置此功能。
<details>
     <summary>GUI 配置 - OpenWRT snapshots</summary>
    
在 Output Plugins 选项卡中，启用 Mqtt 并单击 Configure。然后单击 Add，并输入以下 Details：
- Name - `OpenWRT` 或您喜欢的
- Host - 这是您的 Home Assistant IP
- Port - `1883` 如果您使用的是默认端口
- User - 您的 MQTT 用户
- Password - 您的 MQTT 密码
- Prefix - `collectd`

保存并应用更改。

</details>


<details>
     <summary>CLI 配置 - OpenWRT 23.05 及以下版本</summary>
通过 SSH 连接到您的 OpenWRT 路由器，在 /etc/collectd/ 中创建一个名为 'conf.d' 的新文件夹

使用您最喜欢的编辑器，在 conf.d 中创建一个名为 `mqtt.conf`

将此配置添加到文件中，并编辑以下行：
* `Host` 将其替换为您的 Home Assistant IP
* `User` 将其替换为您的 MQTT 用户
* `Password` 将此密码替换为您的 MQTT 密码
   
```shell
LoadPlugin mqtt
<Plugin "mqtt">
  <Publish "OpenWRT">
    Host "192.168.1.1"
    Port "1883"
    User "mqtt_openwrt"
    Password "MySuperSafePW2!@"
    ClientId "OpenWRT"
    Prefix "collectd"
    Retain true
  </Publish>
</Plugin>
```

通过执行 `service collectd restart` 在 OpenWRT 上重新启动 collectd.
</details>

OpenWRT 将开始向 Home Assistant 发送数据，但您（还）无法看到它。

### Home Assistant 实体设置

返回 Home Assistant 进行最终设置。
一些文本替换是必需的，在 Windows 上您可以使用 Notepad++。

打开此示例 [configuration.yaml](configuration.yaml).

Replace occurrencies of `<Open-WRT-Hostname>` with the value you see in *OpenWRT > System > System > General Settings > Hostname*
将<Open-WRT-Hostname>替换为您在 OpenWRT > 系统 > 系统 > 常规设置 > 主机名称 中看到的值

每个传感器都有一个如下所示的部分：

```yaml
    device:
            identifiers: ImmortalWrt
            name: CncTion J4125-4L
            model: J4125-4L
            manufacturer: CncTion
```

它允许创建 MQTT 设备，因此所有实体都很好地分组。为**每个传感器替换**这些示例值，输入您喜欢的内容，这些只是信息标签。对每个实体使用相同的值！

完成文本编辑后，将代码粘贴到 Home Assistant 上的 configuration.yaml 中。.

保存文件并重新启动 Home Assistant。

完成后，您将拥有一个以您在上面选择的名称命名的新 MQTT 设备，并且将填充实体。

请注意，某些实体需要额外的 OpenWRT mod 包。

## 仪表板和卡片设置

要使用此卡，您必须从 HACS 安装 `multiple-entity-row` 和 `mini-graph-card`。

编辑仪表板，添加新的 YAML 卡片并粘贴您在[lovelace.yaml](lovelace.yaml)中找到的代码。

## 故障排除

要快速检查您是否正在从 OpenWRT 路由器接收数据，请打开 HA 并导航到 MQTT 集成，然后单击*设置*。在 Listen to a topic 部分中，添加 `collectd/#`，然后单击 开始监听 。您应该会看到许多来自 OpenWRT 的消息。

使用[MQTT Explorer](https://community.home-assistant.io/t/addon-mqtt-explorer-new-version/603739)在 MQTT 服务器上检查收到的数据，或者如果您愿意，请使用以下代码：

    mosquitto_sub -h localhost -p 1883 -u user -P Password -t collectd/# -d

 



