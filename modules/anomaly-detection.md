# Module 13: Anomaly Detection

**Goal:** Configure Anomaly Detection to alert upon abnormal/suspicious traffic
---

Calico offers [Anomaly Detection](https://docs.tigera.io/v3.10/threat/anomaly-detection/overview) (AD) as a part of its [threat defense](https://docs.tigera.io/threat/) capabilities. Calico's Machine Learning software is able to baseline "normal" traffic patterns and subsequently detect abnormal or suspicious behaviour. This may resemble an Indicator of Compromise and will generate an Alert in the UI.
Use official documentation for the most recent [configuration instructions](https://docs.tigera.io/threat/anomaly-detection/customizing).

## Steps

1. Review and apply the Anomaly Detection jobs for the managed cluster.

Instructions below for a Managed cluster only. Follow [configuration documentation](hhttps://docs.tigera.io/v3.10/threat/anomaly-detection/customizing) to configure AD jobs for management and standalone clusters.

```bash
less ./demo/70-anomaly-detection/ad-jobs-deployment-managed.yaml

# The following AD jobs and thresholds have been configured as env vars in the ad-jobs-deployment.yaml. 
# In production these values may trigger more alerts than required
	# - name: AD_max_docs
	#   value: "100000"
	# - name: AD_train_interval_minutes
	#   value: "20"
	# - name: AD_search_interval_minutes
	#   value: "5"
	# - name: AD_DnsLatency_IsolationForest_score_threshold
	#   value: "-0.886"
	# - name: AD_ProcessRestarts_threshold
	#   value: "5"
	# - name: AD_port_scan_threshold
	#   value: "200"
	# - name: AD_ip_sweep_threshold
	#   value: "32"
	# - name: AD_BytesInModel_min_size_for_train
	#   value: "1000"
	# - name: AD_SeasonalAD_c
	#   value: "500"
	# - name: AD_BytesOutModel_min_size_for_train
	#   value: "1000"
	# - name: AD_debug
	#   value: "True"
```

2. We need to substitute the Cluster Name in the YAML file with the variable `CALICOCLUSTERNAME` we configured in [Module 3](../module/joining-eks-to-calico-cloud.md). This enables the Machine Learning jobs to target the correct indices in Elastic Search
	```bash
	sed -i "" "s/\$CALICOCLUSTERNAME/${CALICOCLUSTERNAME}/g" ./demo/70-anomaly-detection/ad-jobs-deployment-managed.yaml
	```
	>some linux variants may require different syntax

	```bash
	sed -i "s/\$CALICOCLUSTERNAME/$CALICOCLUSTERNAME/g" ./demo/70-anomaly-detection/ad-jobs-deployment-managed.yaml
	```
3. Now apply the Anomaly Detection deployment YAML
	```bash
	kubectl apply -f ./demo/70-anomaly-detection/ad-jobs-deployment-managed.yaml
	```

4. Simulate anomaly by using an NMAP port scan above the threshold set in our Deployment env vars listed in Step 1.

	```bash
	# mock port scan
	POD_IP=$(kubectl -n dev get po --selector app=centos -o jsonpath='{.items[0].status.podIP}')
	kubectl exec -it -n dev netshoot -- nmap -Pn -r -p 1-1000 $POD_IP
	```
	>Output should resemble
	```bash
	kubectl exec -n dev netshoot -- nmap -Pn -r -p 1-1000 $POD_IP
	
	Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-23 20:20 UTC
	Nmap scan report for 10.240.0.89
	Host is up.
	All 1000 scanned ports on 10.240.0.89 are filtered

	Nmap done: 1 IP address (1 host up) scanned in 201.37 seconds
	```
5. After a few minutes we can see the Alert generated in the Web UI

<img src="../img/anomaly-detection-alert.png" alt="Anomaly Detection Alert" width="100%"/>

[Next -> Module 14](../modules/honeypod-threat-detection.md)

