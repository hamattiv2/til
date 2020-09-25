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

### LoadBalancer
クラスタ外へNodeIPにDNS名が動的に付与されるサービス。
DNSが付与されるので、IPおよびPortを把握しておく必要性がなくなる。
ただし、L4までしかサポートしていないためL7に対応させるにはIngressというリソースを使う必要がある。

サービス作成
```
kubectl expose pod <pod name> --type LoadBalancer --port <port> --name <service name>
```



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

### pod一覧確認(ロングバージョン)
```
kubectl get pod,svc -o wide
```

### Service一覧
```
kubectl get service
```

### シェルへアクセスする
```
kubectl exec -it <pod name> sh
```

### Podを削除する
```
kubectl delete pod <pod name>
```

### Serviceを削除する
```
kubectl delete service <service name>
```
