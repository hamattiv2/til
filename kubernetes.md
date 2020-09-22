# Kubernetesとは
Dockerコンテナを管理するためのツール

Dockerは単体ホスト上に複数コンテナを構築していたが、ホスト故障時の単一故障が発生する可能性がある。
Kubernetesは複数ホスト上に複数コンテナを構築することで単一障害の可能性を潰す。

またコンテナ故障時には自動修復(再起動)する機能を持っていたり、ロードバランシングや自動スケーリング等の機能を持ち合わせている。


# minikubeのインストール
[minikubeのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-minikube/)

### minikubeの起動
```
choco install minikube
```
