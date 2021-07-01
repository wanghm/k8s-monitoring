# Prometheus & Grafanaによる監視ダッシュボードの作成

## 1. Promethus

Karbonクラスタ作成時に自動的にPrometheusをインストールしました。
PersistentVolumeはNutanix Volumeを利用しています。

確認：
### Lens > Workloads > Pods, 「All Namespaces」を選択します

![promethews.png](./images/promethews.png)

### prometheus-k8s-0 を選択

### Volumesの配下「persistentVolume Claim」を選択
![promethews_pv.png](./images/promethews_pvc.png)

### kubectlで確認
![promethews_pvc_kubectl.png](./images/promethews_pvc_kubectl.png)

## 2. Grafana

構成：
![promethews_and_grafana.png](./images/promethews_and_grafana.png)

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
![grafana_login.png](./images/grafana_login.png)

初回ログイン：
* Username - admin
* Password - admin


![grafana_change_password.png](./images/grafana_change_password.png)

初期表示画面：

![grafana_initial_dashboard.png](./images/grafana_initial_dashboard.png)

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

![grafana_add_ds_promethus_url.png](./images/grafana_add_ds_promethus_url.png)
#### 4. Save & Test をクリック

![grafana_add_ds_promethus_url_save.png](./images/grafana_add_ds_promethus_url_save.png)

### ダッシュボード作成

#### 1. Grafana画面からDashboard > Manageを選択
![grafana_dashboard_1.png](./images/grafana_dashboard_1.png)

#### 2. New Dashboard > + Add an empty panel をクリック

metricsにcpuで検索し、cluster:node_cpu:sum_rate5m を選択
![grafana_dashboard_2.png](./images/grafana_dashboard_2.png)

保存
![grafana_dashboard_3.png](./images/grafana_dashboard_3.png)

グラフ表示
![grafana_dashboard_4.png](./images/grafana_dashboard_4.png)

![grafana_dashboard_5.png](./images/grafana_dashboard_5.png)

### ダッシュボードのインポート

#### 1.Select Dashboards > Manage from the left-hand toolbar and click Import.

![grafana_dashboard_import_1.png](./images/grafana_dashboard_import_1.png)


#### 2.  Import via grafana.comに 1621 を指定し、Loadをクリック

![grafana_dashboard_import_2.png](./images/grafana_dashboard_import_2.png)

#### 3. Under Prometheus, select your Prometheus data source and click Import.

![grafana_dashboard_import_3.png](./images/grafana_dashboard_import_3.png)

以下のようなダッシュボードが表示されます。
![grafana_dashboard_import_4.png](./images/grafana_dashboard_import_4.png)