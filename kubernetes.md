# Kubernetesとは
Dockerコンテナを管理するためのツール
Dockerは単体ホスト上に複数コンテナを構築していたが、ホスト故障時の単一故障が発生する可能性がある。
k8sは複数ホスト上に複数コンテナを構築することで単一障害の可能性を潰す。
またコンテナ故障時には自動修復(再起動)する機能を持っていたり、ロードバランシングや自動スケーリング等の機能を持ち合わせている。

Docker単体で使用する場合はdockerコマンドやdocker-composeコマンドを使用していたが、kubernetes(以降、k8s)を使用する際はkubectlコマンド経由でDockerを操作する。

# minikubeのインストール
[minikubeのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-minikube/)

### インストール
```
choco install minikube
```

### 起動
```
minikube start --vm-driver=<driver_name> 
```

### 確認
```
minikube status
```

start済みの場合、以下のような内容が表示されるはず
```
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### 停止
```
minikube stop
```

# kubectlのインストール
[kubectlのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)

### インストール
```
choco install kubectl
```

### バージョン確認
```
kubectl version
```

# Windows(Win10 Pro)でk8sをVirtual Boxで使用する際の注意点

Windows10 Proはインストール時にHyper-Vという仮想環境が起動されるようになっている。
(Docker for Windows使用するときに必要)
しかし、k8sをVirtualBoxで使用するとき、Hyper-VとVirtualBoxで衝突が発生してしまう。(共存はできない)
そのため、Hyper-Vを無効化する必要がある。

無効化
```
bcdedit /set hypervisorlaunchtype off
```

有効化
```
bcdedit /set hypervisorlaunchtype auto
```

いずれもPowerShell(管理者権限)で実行し、実行後は再起動が必要。
minikubeでvirtualboxをドライバーとするには以下のコマンド
```
minikube start --vm-driver=virtualbox
```

# Podとは
1つまたは複数のアプリケーションのコンテナを内包したもの。
管理上の基本単位となり、NICやストレージ、ボリュームを共有する。
Podはノード上に作成される。

## Pod作成
```pod.yml
apiVersion: v1
kind: Pod
metadata:
    name: sample
spec:
    containers:
    - name: nginx 
      image: nginx:1.17.2-alpine
      volumeMounts: # コンテナのマウント先
      - name: storage #volumes.nameと同じ名前にする
        mountPath: /home/nginx
    volumes: # ホストのマウント情報
    - name: storage
      hostPath: 
        path: /data/storage
        type: Directory
```

# Serviceとは
一時的で永続的なIPアドレスを持たないPodに対してクライアントがアクセスするための機能

Podは１つ１つにIPアドレスがふられているが、それぞれのPodと通信する場合IPアドレスを指定する必要がある。
Podはいつ消えるかわからない存在であるため、IPアドレスを覚えておくのはナンセンス。
1つのIPアドレス(Static IP)を用意しておき、Podへの接続を可能とすることで上記の問題を解消する。

## Serviceのタイプ
### ClusterIP
いつ消えるかわからないPodのIPアドレスを抽象化し、代表となるIPアドレスを用意しておくことで
・Podへアクセスする際のPodのIPアドレスを知る必要がなくなる
・ロードバランスを動的に行ってくれる
・クラスタ内部間の通信のみ可能

サービス作成
```
kubectl expose pod <pod name> --type ClusterIP --port <port> --name <service name>

# kubectl get service ←サービス一覧確認
```

ClusterIPはクラスタ内部でしか有効ではない(外部との疎通が取れない)

### NodePort
クラスタ外へPodを公開しNodeIPとNodePort経由で外部との疎通を行うService
手軽に外部公開ができるNodePortは便利なものだが、ノードの再起動などが行われると内部のPodのIP等が切り替わるため
使用できなくなる。
また、NodeIPとNodePortを知っておく必要があるため本番環境での利用には適していない。

サービス作成
```
kubectl expose pod <pod name> --type NodePort --port <port> --name <service name>
```

ymlファイルの場合
```nodeport.yml

# nginxを構築
apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels:
        app: web
        env: study
spec:
    containers:
    - name: nginx
      image: nginx:1.17.2-alpine

# 上記で定義したnginxに外部からアクセスできるようNodePortでServiceを作成する
---
apiVersion: v1
kind: Service
metadata:
    name: web-service
spec:
    type: NodePort
    selector:
        app: web
        env: study
    ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
```

### LoadBalancer
クラスタ外へNodeIPにDNS名が動的に付与されるサービス。
DNSが付与されるので、IPおよびPortを把握しておく必要性がなくなる。
ただし、L4までしかサポートしていないためL7に対応させるにはIngressというリソースを使う必要がある。

サービス作成
```
kubectl expose pod <pod name> --type LoadBalancer --port <port> --name <service name>
```

# Ingress
Podをクラスタ内外へ公開するL7ロードバランサ
URLによるロードバランスが可能(ドメイン、パスによる振り分けが可能になる)

### Addonをリスアップ
```
minikube addons list
```

### Ingress addonを追加
```
minikube addons enable ingress
```
### Ingress controller podをチェック
```
kubectl get pods -n kube-system
```

### ingress resourceを作成
```
kubectl apply -f ingress.yaml
```

**ingress.yaml**
```ingress.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: helloworld
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: helloworld-nodeport
          servicePort: 8080
```

上記のyamlファイルを適用することでどのパスにアクセスしても helloworld-nodeportのpodが返ってくる。

# Replica
Podの集合体、Podのスケールが可能

```replicaset.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: sample
spec:
    replicas: 3 # 複製するPodの数
    selector: # ReplicaSetとPodを紐づける template.metadata.labelsと一致する必要がある
        matchLabels:
            app: web
            env: study
    template: # 作成するPodのマニュフェストファイル
        metadata:
            name: nginx
            labels:
                app: web
                env: study
        spec:
            containers:
            - name: nginx
              image: nginx:1.17.2-alpine
```

# Deployment
ReplicaSetの世代管理を行う。

```deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx
    annotations: # Deploymentを更新するときにコメントを残すときに使う
        kubernetes.io/change-cause: "Update nginx 1.17.3"
spec:
    replicas: 2
    selector:
        matchLabels:
            app: web
            env: study
    revisionHistoryLimit: 14 # 何個の世代を残すのか指定する。後々世代を戻すときに使える。
    strategy: 
        type: RollingUpdate
        rollingUpdate:
            maxSurge: 1 # レプリカ数を超えてよいPod数
            maxUnavailable: 1 # 1度に消失してよいPod数
    template:
        metadata:
            name: nginx
            labels:
                app: web
                env: study
        spec:
            containers:
                - name: nginx
                  image: nginx:1.17.3-alpine
```

### strategy.rollingUpdate.maxSurge, maxUnavailableについて
[maxSurge, maxUnavailableのわかりやすい解説](https://tech-lab.sios.jp/archives/18553#RollingUpdate)

### ロールアップ
ロールアップするときはymlファイルを更新し、再度`kubectl apply -f <file>`することで更新してくれる。

### ロールバック
```
kubectl rollout undo <type/name> [--to-revision=<revision number>]

# 例の場合だと14個のrevisionHisotryを残すので、to-revisionでも最大14を指定することが可能
```

# ConfigMap
k8sの設定情報を集約するリソース

# Secret
Pod上で扱い機微情報を取り扱うリソース。
base64で暗号化した情報をPodに展開するときに複合したりできる。

```
apiVersion: v1
kind: Secret
metadata:
    name: sample-secret
data:
    message: SGVsbG8gV29ybGQ= # echo -n 'Hello World !' | base64
    keyfile: WU9VUi1TRUNSRVQtS0VZ   # cat ./keyfile | base64

---
apiVersion: v1
kind: Pod
metadata:
    name: sample
    namespace: default
spec:
    containers:
    - name: sample
      image: nginx:1.17.2-alpine
      env:
      - name: MESSAGE
        valueFrom:
          secretKeyRef:
            name: sample-secret
            key: message
  
      volumeMounts: # Secretで暗号化したいファイルを指定する場合のパターン
      - name: secret-storage # spec.volumes.nameと名前を合わせる
        mountPath: /home/nginx # Pod内の保存先
    volumes:
    - name: secret-storage
      secret:
        secretName: sample-secret
        items:
        - key: keyfile
          path: keyfile # 
```

# Podとホスト間のファイル転送
ホスト→Pod
```
kubectl cp <src> <pod name>:<dest>
```
src: カレントディレクトリからのパス
dest: 絶対パス

Pod→ホスト
```
kubectl cp <pod name>:<src> <dest>
```
src: 絶対パス
dest: カレントディレクトリからのパス

# k8sコマンド
[コマンドチートシート](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/)

### マスターのIPアドレスを表示する
```
kubectl cluster-info
```

### ノードの表示
```
kubectl get nodes
```

### 接続情報の確認
```
kubectl config view
```
どのk8sクラスタにどのユーザーとして接続するのかの接続設定をkubeconfigファイルが定義し、クライアントにあるkubeconfigファイルをkubectlが読み取って接続先を決めている。


### Pod一覧確認
```
kubectl get pod
```

### Service一覧確認
```
kubectl get service
```

### Replicaset一覧確認
```
kubectl get rs
```

### Deploy一覧確認
```
kubectl get deploy
```

### 一覧確認(ロングバージョン)
```
kubectl get <type> -o wide
```

### シェルへアクセスする
```
kubectl exec -it <pod name> sh
```

### 状態の確認
```
kubectl describe <type/name>
```

### ログの確認
```
kubectl logs <type/name>
```

### rollingUpdateのhistoryを確認する
```
kubectl rollout history <type/name>
```

### rollout


