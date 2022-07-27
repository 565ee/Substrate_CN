• [介绍](#index1)  
• [学习成果](#index2)  
• [启动基板模板节点](#index3)  
• [配置 Prometheus 以抓取您的 Substrate 节点](#index4)  
• [使用 Grafana 可视化 Prometheus 指标](#index5)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
最新版本的 Substrate 公开了指标，例如您的节点连接到多少对等点，您的节点正在使用多少内存等。要可视化这些指标，您可以使用 Prometheus和Grafana等工具。在本教程中，您将学习如何使用 Grafana 和 Prometheus 来抓取和可视化节点指标。

可能的架构可能如下所示：
```
+-----------+                     +-------------+                                                              +---------+
| Substrate |                     | Prometheus  |                                                              | Grafana |
+-----------+                     +-------------+                                                              +---------+
      |               -----------------\ |                                                                          |
      |               | Every 1 minute |-|                                                                          |
      |               |----------------| |                                                                          |
      |                                  |                                                                          |
      |        GET current metric values |                                                                          |
      |<---------------------------------|                                                                          |
      |                                  |                                                                          |
      | `substrate_peers_count 5`        |                                                                          |
      |--------------------------------->|                                                                          |
      |                                  | --------------------------------------------------------------------\    |
      |                                  |-| Save metric value with corresponding time stamp in local database |    |
      |                                  | |-------------------------------------------------------------------|    |
      |                                  |                                         -------------------------------\ |
      |                                  |                                         | Every time user opens graphs |-|
      |                                  |                                         |------------------------------| |
      |                                  |                                                                          |
      |                                  |       GET values of metric `substrate_peers_count` from time-X to time-Y |
      |                                  |<-------------------------------------------------------------------------|
      |                                  |                                                                          |
      |                                  | `substrate_peers_count (1582023828, 5), (1582023847, 4) [...]`           |
      |                                  |------------------------------------------------------------------------->|
      |                                  |                                                                          |

```

# <span id='index2'>• 学习成果</span>  
了解如何使用 Prometheus 对 Substrate 节点进行时间序列抓取
了解如何使用 Grafana 和 Prometheus 可视化节点指标

# <span id='index3'>• 启动基板模板节点</span>  
在继续之前，您应该完成 创建您的第一个基板链 教程。这里使用相同的基板版本、目录结构约定和 bin 名称。您当然可以使用您自己的自定义 Substrate 节点而不是模板，只需根据需要编辑显示的命令。

Substrate 公开了一个端点，该端点以 port 上可用 的Prometheus 公开格式9615提供指标。您可以使用 更改端口--prometheus-port <PORT>并使其能够通过本地主机以外的接口访问 --prometheus-external。
```
# Optionally add the `--prometheus-port <PORT>`
# or `--prometheus-external` flags
./target/release/node-template --dev
```

# <span id='index4'>• 配置 Prometheus 以抓取您的 Substrate 节点</span>  
在安装 Prometheus 的工作目录中，您将找到一个prometheus.yml配置文件。让我们修改它（或创建一个自定义的 new on）以配置 Prometheus 通过将暴露的端点添加到目标数组中来抓取它。如果您修改默认值，这将是不同的：
```
# --snip--

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'substrate_node'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    # Override the global default and scrape targets from this job every 5 seconds.
    # ** NOTE: you want to have this *LESS THAN* the block time in order to ensure
    # ** that you have a data point for every block!
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9615']
```

现在我们可以使用 prometheus.yml 配置文件启动一个 Prometheus 实例。假设您将二进制文件下载 cd到安装目录并运行：
```
# specify a custom config file instead if you made one here:
./prometheus --config.file prometheus.yml  
```  
  
# <span id='index5'>• 使用 Grafana 可视化 Prometheus 指标</span>  
现在启动 Grafana，您将登录并在浏览器中导航到它（默认为http://localhost:3000/）。登录（使用默认用户admin和密码admin）并导航到位于 http://localhost:3000/datasources的数据源页面。

然后，您需要选择一种Prometheus数据源类型并指定 Grafana 需要在哪里查找它。

随着您的基板节点和 Prometheus 正在运行，配置 Grafana 以在其默认端口上查找 Prometheus：http://localhost:9090（除非您对其进行了自定义）。

点击Save & Test以确保您正确设置了数据源。现在您可以配置新的仪表板了！
  
模板 Grafana 仪表板
如果你想从这里开始一个基本的仪表板，这里有一个模板示例，你可以Import 在 Grafana 中获取有关节点的基本信息：
https://grafana.com/grafana/dashboards/13759/
![image](https://user-images.githubusercontent.com/28084126/175784356-079164f6-3106-4ecd-ae53-335e0d67b2e9.png)
 
# <span id='index98'>• Substrate Tutorials , Substrate 教程</span>  
CN 中文 Github  [Substrate 教程 : github.com/565ee/Substrate_CN](https://github.com/565ee/Substrate_CN)  
CN 中文 CSDN    [Substrate 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11846056.html)  
EN 英文 Github  [Substrate Tutorials : github.com/565ee/Substrate_EN](https://github.com/565ee/Substrate_EN)  
EN 英文 dev.to  [Substrate Tutorials : dev.to/565ee](https://dev.to/565ee/substrate-tutorials-5n4) 

# <span id='index99'>• Contact 联系方式</span>  
Homepage : [565.ee](https://565.ee)  
微信公众号 : wx468116118  
微信 QQ   : 468116118  
GitHub   : [github.com/565ee](https://github.com/565ee)   
CSDN     : [blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118)  
Email    : 468116118@qq.com    
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/175784401-caaef50b-d719-4e7e-b581-40fd9c1ff592.png)
