# Sample : Intel OpenNESS SR-IOV Operator for Wireless FEC Accelerators

## デプロイ方法

```
$ oc new-project vran-acceleration-operators
$ helm lint ./charts/*
# Operator インストール
$ helm upgrade --install vran-acceleration-operator ./charts/sriov-fec-operator -n vran-acceleration-operators
# Custom Resource 作成
$ helm upgrade --install n3000-fec-config ./charts/n3000-fec-config -f ./n3000-fec-config-node1-values.yaml
```

## Sample Helm Chart の構成

Sample Helm Chart は以下の構成となっています。

```
.
├── charts
│   ├── n3000-fec-config : Custom Resource 作成用 Helm Chart
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── _helpers.tpl
│   │   │   └── sriovfecclusterconfig.yaml
│   │   └── values.yaml
│   └── sriov-fec-operator : Operator インストール用 Helm Chart
│       ├── Chart.yaml
│       ├── templates
│       │   ├── _helpers.tpl
│       │   ├── operatorgroup.yaml
│       │   ├── pre-delete-job.yaml
│       │   ├── subscription.yaml
│       │   └── tests
│       │       └── test-operator-install.yaml
│       └── values.yaml
├── n3000-fec-config-node1-values.yaml : Custom Resource 作成時の values.yaml サンプル
```

Custom Resource 作成用 Helm Chart と Operator インストール用 Helm Chart は分割しています。

これは Operator により vRAN FEC acceleration devices が搭載された Node を対象に作成される sriovfecnodeconfig Custom Resouce の `status` に反映される対象の Accelerators を選択するための情報を SriovFecClusterConfig CR の .spec.acceleratorSelector フィールドに指定する必要があるためです。


## Chart　Hooks の利用

Operator インストール用 Helm Chart では helm uninstall 時に自身が管理する Subscription および OperatorGroup リソースを削除しますが、 Operator のアンインストールには [ClusterServiceVersion の削除](https://docs.openshift.com/container-platform/4.9/operators/admin/olm-deleting-operators-from-cluster.html#olm-deleting-operators-from-a-cluster)が必要となります。

これに対応するため [Chart Hook](https://helm.sh/docs/topics/charts_hooks/) 機能を利用して CSV を削除しています。
