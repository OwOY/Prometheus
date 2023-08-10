<img src="https://1.bp.blogspot.com/-u5Hk7FwKKDk/XfHzHDZOBWI/AAAAAAAAILY/yR01WuzV5JwOT02qXTSQ5DlUBuiE8qk8QCPcBGAYYCw/s640/57307750-696bb980-70e5-11e9-9b0b-73ad88bde6a3.png">

----
# Intro
Prometheus 受啟發於 Google 的 Brogmon 監控系統（類似的 Kubernetes 是從 Google 的 Brog 演變系統而來），從 2012 年開始由前 Google 工程師在 Soundcloud 以開源軟件的形式進行研發，並於 2015 年早期發布早期2016年5月繼Kubernetes之後成為第二個加入CNCF基金會的正式項目，同年6月發布1.0版本。2017年底發布了基於全新存儲層的2.0版本，能夠更好地與容器平台、雲配合平台。

### 監控目標
- 長期趨勢分析:可以通過對未來的持續時間記錄和指標進行長期趨勢分析，例如，通過對數據監控的空間監測，對觀察狀態上的觀察情況的判斷，我們可以提前對資源進行擴容。
- 對照分析：兩個版本的系統運行資源使用情況的差異如何？在不同容量情況下系統的併發和負載變化如何？通過監控能夠方便的對系統進行跟蹤和比較。
- 警告：當系統出現或者即將出現故障時，監控系統需要迅速反應並通知管理員，從而能夠對問題進行快速的處理或者提前預防問題的發生，避免出現對業務的影響。
- 故障分析與定位：當問題發生後，需要對問題進行調查和處理。通過對不同監控監控以及歷史數據的分析，能夠找到並解決根源問題。
- 數據可視化：通過可視化儀表盤能夠直接獲取系統的運行狀態、資源使用情況、以及服務運行狀態等直觀的信息。
### 優點
- 易於管理
Prometheus核心部分只有一個單獨的二進制文件，不存在任何的第三方依賴(數據庫，緩存等等)。唯一需要的就是本地磁盤，因此不會有潛在級聯故障的風險。
Prometheus基於Pull模型的架構方式，可以在任何地方（本地電腦，開發環境，測試環境）搭建我們的監控系統。對於一些複雜的情況，還可以使用Prometheus服務發現(Service Discovery)的能力動態管理監控目標。
- 監控服務的內部運行狀態
Pometheus鼓勵用戶監控服務的內部狀態，基於Prometheus豐富的Client庫，用戶可以輕鬆的在應用程序中添加對Prometheus的支持，從而讓用戶可以獲取服務和應用內部真正的運行狀態。
- 強大的數據模型
所有採集的監控數據均以指標(metric)的形式保存在內置的時間序列數據庫當中(TSDB)。所有的樣本除了基本的指標名稱以外，還包含一組用於描述該樣本特徵的標籤。
- 強大的查詢語言PromQL
Prometheus內置了一個強大的數據查詢語言PromQL。
通過PromQL可以實現對監控數據的查詢、聚合。同時PromQL也被應用於數據可視化(如Grafana)以及警告當中。
- 高效
對於監控系統而言，大量的監控任務必然導致有大量的數據產生。而Prometheus可以高效地處理這些數據。
- 易於集成
使用Prometheus可以快速搭建監控服務，並且可以非常方便地在應用程序中進行集成。目前支持：Java， JMX， Python， Go，Ruby， .Net， Node.js...等語言
- 可視化
Prometheus Server中自帶了一個Prometheus UI，通過這個UI可以方便地直接對數據進行查詢，並且支持直接以圖形化的形式展示數據。
- 開放性
對於Prometheus來說，使用Prometheus的client library的輸出格式不止支持Prometheus的格式化數據，也可以輸出支持其它監控系統的格式化數據，比如Graphite。
<br><br>
# 使用方法
### 安裝
- Ubuntu
```
sudo apt -y install prometheus
```
- Docker
```
docker run \
     -d \
     --name prometheus \
     -p 9090:9090 \
     -v ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml # 同步config檔案
     prometheus
```
### Config設置(於config內加入，以下以cadvisor為例)
```
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['localhost:8080']
```
- prometheus.yml:
```
# Sample config for Prometheus.

global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['35.239.150.35:9090']

  - job_name: 'node'
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['35.239.150.35:9100']

  - job_name: "cAdvisor"
    static_configs:
      - targets: ['cAdvisor:8888']
```
### 完成:
- 完成上述設定後，可進入 localhost:9090 看見此監控畫面。
![image.png](/prometheus/1.png)

# 警示條件設定
1. 建立警示.yaml檔案  
   alert.yaml
   ```
   groups:
       - name: example
         rules:
           # Alert for any instance that is unreachable for >5 minutes.
           - alert: InstanceDown
             expr: up == 0
             for: 5m
             labels:
               severity: page
             annotations:
               summary: "Instance {{ $labels.instance }} down"
               description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
   ```
2. 在 prometheus.yaml 檔內加上 rule_files
   ```
   rule_files:
     - alert.yaml

   scrape_configs: ......
   ```
3. 詳細警示條件範例請參照 [警示條件範本](https://samber.github.io/awesome-prometheus-alerts/rules.html)
---- 
# 參考文件
- [Prometheus-Book](https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/quickstart/why-monitor#yi-yu-guan-li)
- [開源監控docker的三大神器 – grafana+cAdvisor+prometheus](https://sectools.tw/docker-monitor/)
- [警示條件範本](https://samber.github.io/awesome-prometheus-alerts/rules.html)
