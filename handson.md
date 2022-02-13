# Helm Chart Development Handson

## はじめに

このハンズオンは Helm Chart の開発方法を実際に体験して理解することを目的とします。

以下の流れで Helm Chart を作成していきます。

1. Chart の作成
2. テスト、デバッグ
3. テンプレート化
4. パッケージング
5. Chart Repository の準備
6. リリース

## 事前準備

割り当てられたユーザ名を環境変数に設定してください。

```
$ HANDSONUSER=<ユーザ名>
```

## Chart の作成

Chart の作成は以下の流れで行うことをおすすめします。

1. ボイラープレートの生成
2. 必要なマニフェストファイルへの置き換え
3. 

### ボイラープレートを生成する

Chart を作成するためにはまず最初に `helm create` コマンドで Helm Chart のボイラープレートを作成します。これにより Helm Chart の基本的なディレクトリ構成とファイルが作成されます。

```
$ helm create handson-$HANDSONUSER
Creating handson-user1

$ tree handson-user1 
handson-user1
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

### 必要なマニフェストファイルへの置き換え

`charts/` 配下のマニフェストファイルを実際にデプロイする対象のマニフェストファイルに置き換えます。
置き換えられたら実際に OpenShift にデプロイできることを確認します。

この時点ではテンプレート化は行っておらず、単純に `charts/` 配下に格納したマニフェストファイルがそのまま適用されます。

### Chart の依存関係を定義する



### テスト

oc run operator-test --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --serviceaccount operator-test --command -- /bin/bash -c '[ $(oc get csv sriov-fec.v2.1.0 -o jsonpath='{.status.phase}') = 'Succeeded' ]'

oc run operator-test --dry-run=client -oyaml --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --serviceaccount operator-test --command -- /bin/bash -c '[ $(oc get csv sriov-fec.v2.1.0 -o jsonpath='{.status.phase}') = 'Succeeded' ]'

oc create serviceaccount operator-test
oc adm policy add-role-to-user view -z operator-test
oc run operator-test --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --rm -it --serviceaccount operator-test

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: operator-test
  name: operator-test
spec:
  containers:
  - command:
    - /bin/bash
    - -c
    - '[ $(oc get csv sriov-fec.v2.1.0 -o jsonpath={.status.phase}) = Succeeded ]'
    image: registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream
    name: operator-test
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  serviceAccountName: operator-test
```

## テスト、デバッグ

## パッケージング

## Chart Repository の準備

## リリース

## CICD パイプラインの実装

