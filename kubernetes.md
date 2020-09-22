# Kubernetesとは
Dockerコンテナを管理するためのツール

Dockerは単体ホスト上に複数コンテナを構築していたが、ホスト故障時の単一故障が発生する可能性がある。
Kubernetesは複数ホスト上に複数コンテナを構築することで単一障害の可能性を潰す。

またコンテナ故障時には自動修復(再起動)する機能を持っていたり、ロードバランシングや自動スケーリング等の機能を持ち合わせている。


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

# Windows(Win10 Pro)でkubernetesをVirtual Boxで使用する際の注意点

Windows10 Proはインストール時にHyper-Vという仮想環境が起動されるようになっている。
(Docker for Windows使用するときに必要)
しかし、kubernetesをVirtualBoxで使用するとき、Hyper-VとVirtualBoxで衝突が発生してしまう。(共存はできない)
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






