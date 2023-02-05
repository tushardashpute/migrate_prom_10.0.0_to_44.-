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



![image](https://user-images.githubusercontent.com/74225291/216829638-776e1836-f083-4e7f-a6e1-d79fee8fa33e.png)
