# migrate_prom_10.0.0_to_44.-*

Assumption:

You have EKS cluster with max 1.21 and prometheus running on it.

Problem Statement:
- Your current prometheus is having more memory consumption due to which you want to use S3 as a object store (thanos supports it) and you want to upgrade your cluster to 1.22, which current prometheus version is not supporting as it's older version which we can't directly upgrade.
- We can't directly existing promtheus to new versino bcoz there are multiple breaking changes happend in between and it's supporting current cluster version too.

Steps
=====
1. Install thanos with promehteus 10.0.0
2. Install thanos querier and thanos storage
3. Add the thanos querier as a datasource to Grafna and access the dashboards using that.
4. Delete prometheus 10.0.0
5. Install thanos with prometheus 39.13.3
6. Validate the older data in grafana prior to new prom installations.

For Step 1 to 3 follow below:

https://github.com/tushardashpute/prom-thanos.git

4. Delete prometheus 10.0.0

helm uninstall prom

        # helm ls
        NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
        prom    default         6               2023-02-05 04:26:29.219582683 +0000 UTC deployed        kube-prometheus-stack-10.0.0    0.42.1
        
        
        # helm uninstall prom
        W0205 08:35:28.471174    5243 warnings.go:70] rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
        W0205 08:35:28.494330    5243 warnings.go:70] rbac.authorization.k8s.io/v1beta1 Role is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 Role
        W0205 08:35:28.519590    5243 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
        W0205 08:35:28.575411    5243 warnings.go:70] rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
        W0205 08:35:28.806502    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.811242    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.812588    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.812834    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.815704    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.823928    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.825991    5243 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
        W0205 08:35:28.857899    5243 warnings.go:70] admissionregistration.k8s.io/v1beta1 MutatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 MutatingWebhookConfiguration
        W0205 08:35:29.048448    5243 warnings.go:70] admissionregistration.k8s.io/v1beta1 ValidatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 ValidatingWebhookConfiguration
        release "prom" uninstalled

Now delete the crd's

        kubectl delete crd prometheuses.monitoring.coreos.com
        kubectl delete crd prometheusrules.monitoring.coreos.com
        kubectl delete crd servicemonitors.monitoring.coreos.com
        kubectl delete crd podmonitors.monitoring.coreos.com
        kubectl delete crd alertmanagers.monitoring.coreos.com
        kubectl delete crd thanosrulers.monitoring.coreos.com

5. Install thanos with prometheus 39.13.3

export prom_version=39.13.3

    # helm upgrade --install prom prometheus-community/kube-prometheus-stack   --version "${prom_version}"    -f prometheus-operator-values.yaml
    Release "prom" does not exist. Installing it now.
    NAME: prom
    LAST DEPLOYED: Sun Feb  5 08:39:08 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    NOTES:
    kube-prometheus-stack has been installed. Check its status by running:
      kubectl --namespace default get pods -l "release=prom"

    Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.


      # k get pods
      NAME                                                     READY   STATUS    RESTARTS   AGE
      alertmanager-prom-kube-prometheus-stack-alertmanager-0   2/2     Running   0          2m57s
      postgres-5595cb886b-svrdq                                1/1     Running   0          5h19m
      prom-grafana-68778d979f-k6wlm                            3/3     Running   0          35s
      prom-kube-prometheus-stack-operator-868bb9657b-pb6kw     1/1     Running   0          3m2s
      prom-kube-state-metrics-7bdfccc985-k8m8f                 1/1     Running   0          3m2s
      prom-prometheus-node-exporter-5sh9p                      1/1     Running   0          3m2s
      prom-prometheus-node-exporter-6zs7h                      1/1     Running   0          3m2s
      prometheus-prom-kube-prometheus-stack-prometheus-0       3/3     Running   0          24s
      springboot-db6684d7b-7b9fz                               1/1     Running   0          6h18m
      springboot-db6684d7b-bvtd5                               1/1     Running   0          6h18m
      springboot-db6684d7b-vs46s                               1/1     Running   0          6h18m
      thanos-querier-65cf594d-5nkxn                            1/1     Running   0          178m
      thanos-store-0                                           1/1     Running   0          173m


      # k get svc
      NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                           AGE
      alertmanager-operated                     ClusterIP      None             <none>                                                                    9093/TCP,9094/TCP,9094/UDP        5m51s
      kubernetes                                ClusterIP      10.100.0.1       <none>                                                                    443/TCP                           6h47m
      postgres                                  NodePort       10.100.78.223    <none>                                                                    5432:32002/TCP                    5h19m
      prom-grafana                              LoadBalancer   10.100.249.24    a579deb69e04d4327b9f2e68e01d857f-1069635652.us-east-1.elb.amazonaws.com   80:31620/TCP                      5m56s
      prom-kube-prometheus-stack-alertmanager   ClusterIP      10.100.135.21    <none>                                                                    9093/TCP                          5m55s
      prom-kube-prometheus-stack-operator       ClusterIP      10.100.54.8      <none>                                                                    443/TCP                           5m55s
      prom-kube-prometheus-stack-prometheus     ClusterIP      10.100.79.11     <none>                                                                    9090/TCP                          5m56s
      prom-kube-state-metrics                   ClusterIP      10.100.67.17     <none>                                                                    8080/TCP                          5m55s
      prom-prometheus-node-exporter             ClusterIP      10.100.241.172   <none>                                                                    9100/TCP                          5m56s
      prometheus-operated                       ClusterIP      None             <none>                                                                    9090/TCP,10901/TCP                5m50s
      springboot                                LoadBalancer   10.100.120.10    a63ee3365c06e4e72bdca4ebddb7a283-1880785659.us-east-1.elb.amazonaws.com   33333:31134/TCP                   6h21m
      thanos-querier                            LoadBalancer   10.100.14.125    a5c81e23433014cfd831f2bbc93aecc0-1888239819.us-east-1.elb.amazonaws.com   10901:30885/TCP,9090:32451/TCP    3h1m
      thanos-sidecar                            LoadBalancer   10.100.225.140   a2986d0cd4a934606b4a99eec0ee8f38-736894394.us-east-1.elb.amazonaws.com    10901:32106/TCP                   5h15m
      thanos-store                              LoadBalancer   10.100.142.71    a5e5a24bbbb0240ecb1678f5d65d8d4d-533113182.us-east-1.elb.amazonaws.com    10901:31367/TCP,10902:32155/TCP   3h

![image](https://user-images.githubusercontent.com/74225291/216829638-776e1836-f083-4e7f-a6e1-d79fee8fa33e.png)
