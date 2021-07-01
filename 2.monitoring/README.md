# Prometheus & Grafanaによる監視ダッシュボードの作成

## 1. Promethus

Karbonクラスタ作成時に自動的にPrometheusをインストールしました。
PersistentVolumeはNutanix Volumeを利用しています。

確認：
### Lens > Workloads > Pods, 「All Namespaces」を選択します

<img src=./images/promethews.png width=512>

### prometheus-k8s-0 を選択

### Volumesの配下「persistentVolume Claim」を選択
<img src=./images/promethews_pvc.png width=512>


### kubectlで確認
<img src=./images/promethews_pvc_kubectl.png width=768>


## 2. Grafana

構成：
<img src=./images/promethews_and_grafana.png width=512>

### ingress作成
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-grafana
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: "grafana.local" 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
```
### ホスト名：  grafana.localを作業PCのhostsファイルに追記
```
#Mac: 
sudo echo "<<ingress-nginx-controllerのEXTERNAL-IP>> grafana.local >> /etc/hosts"

# Windows:
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "<EXTERNAL-IP>`grafana.local" -Force
cat C:\Windows\System32\drivers\etc\hosts
```
### Deploy
```
kubectl -f grafana-deployment.yaml
kubectl -f ingress-grafana.yaml
```
### 動作確認
ブラウザでhttp://grafana.localにアクセス

以下の画面が表示されます。

<img src=./images/grafana_login.png width=512>


初回ログイン：
* Username - admin
* Password - admin

<img src=./images/grafana_change_password.png width=512>


初期表示画面：

<img src=./images/grafana_initial_dashboard.png width=512>

### Datasourceを設定

#### 1. Prometheus-operatedのEndpointを取得
```
kubectl get ep -n ntnx-system | grep prometheus-operated
```
例：
```
prometheus-operated           172.20.223.203:9090   
```

#### 2. Grafana画面で,左側のギアアイコンを選択し、「Configuration」配下の「Data source」を選択

![grafana_add_ds_1.png](./images/grafana_add_ds_1.png)

#### 3. Prometheusを選択し、URLにPrometheus-operatedのEndpointを設定

例： http://172.20.223.203:9090

<img src=./images/grafana_add_ds_promethus_url.png width=512>

#### 4. Save & Test をクリック

<img src=./images/grafana_add_ds_promethus_url_save.png width=512>


### ダッシュボード作成

#### 1. Grafana画面からDashboard > Manageを選択
![grafana_dashboard_1.png](./images/grafana_dashboard_1.png)

#### 2. New Dashboard > + Add an empty panel をクリック

metricsにcpuで検索し、cluster:node_cpu:sum_rate5m を選択

<img src=./images/grafana_dashboard_2.png width=512>

保存

<img src=./images/grafana_dashboard_3.png width=512>

グラフ表示

<img src=./images/grafana_dashboard_4.png width=512>

<img src=./images/grafana_dashboard_5.png width=512>

### ダッシュボードのインポート

#### 1. Dashboards > Manage を選択してからImportをクリック

![grafana_dashboard_import_1.png](./images/grafana_dashboard_import_1.png)


#### 2.  Import via grafana.comに 1621 を指定し、Loadをクリック

<img src=./images/grafana_dashboard_import_2.png width=512>

#### 3. Prometheusの配下Prometheus data sourceを選択し、Importをクリック


<img src=./images/grafana_dashboard_import_3.png width=512>

以下のようなダッシュボードが表示されます。

<img src=./images/grafana_dashboard_import_4.png width=512>
