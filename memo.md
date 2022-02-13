oc new-project vran-acceleration-operators

apiVersion: v1
kind: Namespace
metadata:
  name: vran-acceleration-operators

cat <<EOF > operatorgroup.yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: vran-operators
  namespace: vran-acceleration-operators
spec:
  targetNamespaces:
    - vran-acceleration-operators
EOF

cat <<EOF > subscription.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: sriov-fec-subscription
  namespace: vran-acceleration-operators
spec:
  channel: stable
  name: sriov-fec
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF

$ helm lint ./charts/*

$ helm install vran-acceleration-operator ./charts/sriov-fec-operator

Device Plugin は Accelerator をリソースとするのではなく VF をリソースとする
https://www.fujitsu.com/jp/documents/products/computing/servers/unix/sparc/technical/document/sr-iov_guide.pdf
https://docs.openshift.com/container-platform/4.9/nodes/pods/nodes-pods-plugins.html
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/
https://kubernetes.io/docs/tasks/administer-cluster/extended-resource-node/


helm uninstall 時以下の課題がある
- Subscription と OperatorGroup は削除されるが Cluster Service Version が削除されない
  https://docs.openshift.com/container-platform/4.9/operators/admin/olm-deleting-operators-from-cluster.html#olm-deleting-operators-from-a-cluster
- CR が削除されて Operator が処理を完了するまでは Operator は削除されると困る
  - Operator Install を Sub Chart にすれば Order が 後になるので OK? ただ Hook などで待つ必要がある
  https://helm.sh/docs/topics/charts/#operational-aspects-of-using-dependencies
  - Custom Resource はデプロイ順番が一番最後
  https://github.com/helm/helm/blob/484d43913f97292648c867b56768775a55e4bba6/pkg/releaseutil/kind_sorter.go
  https://github.com/helm/helm/issues/4578

helm template --debug vran-acceleration-operator ./charts/sriov-fec-operator
helm install --dry-run --debug vran-acceleration-operator ./charts/sriov-fec-operator

### Hook

以下の順番で削除する必要がある

CSV=$(oc get subscription sriov-fec-subscription -o jsonpath='{.status.currentCSV}')
helm uninstall vran-acceleration-operator
  oc delete subscription sriov-fec-subscription
  oc delete operatorgroup
oc delete csv $CSV

### テスト

oc run operator-test --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --serviceaccount operator-test --command -- /bin/bash -c '[ $(oc get csv sriov-fec.v2.1.0 -o jsonpath='{.status.phase}') = 'Succeeded' ]'

oc run operator-test --dry-run=client -oyaml --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --serviceaccount operator-test --command -- /bin/bash -c '[ $(oc get csv sriov-fec.v2.1.0 -o jsonpath='{.status.phase}') = 'Succeeded' ]'

oc create serviceaccount operator-test
oc adm policy add-role-to-user view -z operator-test
oc run operator-test --image registry.redhat.io/openshift4/ose-cli:v4.9.0-202201261125.p0.g3f16530.assembly.stream --restart Never --rm -it --serviceaccount operator-test


$ cat <<EOF > sriovfecclusterconfig.yaml
apiVersion: sriovfec.intel.com/v2
kind: SriovFecClusterConfig
metadata:
  name: config
  namespace: vran-acceleration-operators
spec:
  priority: 1
  nodeSelector:
    kubernetes.io/hostname: node1
  acceleratorSelector:
    pciAddress: 0000:14:00.1
  physicalFunction:  
    pfDriver: pci-pf-stub
    vfDriver: vfio-pci
    vfAmount: 2
    bbDevConfig:
      n3000:
        # Network Type: either "FPGA_5GNR" or "FPGA_LTE"
        networkType: "FPGA_5GNR"
        # Programming mode: 0 = VF Programming, 1 = PF Programming
        pfMode: false
        flrTimeout: 610
        downlink:
          bandwidth: 3
          loadBalance: 128
          queues:
            vf0: 16
            vf1: 16
            vf2: 0
            vf3: 0
            vf4: 0
            vf5: 0
            vf6: 0
            vf7: 0
        uplink:
          bandwidth: 3
          loadBalance: 128
          queues:
            vf0: 16
            vf1: 16
            vf2: 0
            vf3: 0
            vf4: 0
            vf5: 0
            vf6: 0
            vf7: 0
EOF

$ helm install n3000-fec-config ./n3000-fec-config

参考
https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx