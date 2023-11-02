# prometheus-grafana-on-eks
Pre-requisite
-------------
  1. EKS Cluster
  2. Ebs-csi add on for EKS
  3. Service account --> ebs-csi-controller
  4. Install helm

IAM ODIC provider
-----------------
    eksctl utils associate-iam-oidc-provider \
    --region <region-name> \
    --cluster <cluster-name> \
    --approve

Create service account role and attach policy
-----------------------------------------------
    eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --region <region-name> \
    --namespace kube-system \
    --cluster <cluster-name> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
<img width="790" alt="image" src="https://github.com/Abhi-chintu/prometheus-grafana-on-eks/assets/94033251/7b69e287-467b-48c1-87d4-ef05aed20331">

Create ebs-csi-driver addon
---------------------------
     eksctl create addon --name aws-ebs-csi-driver --cluster <cluster-name> --service-account-role-arn arn:aws:iam::473413441227:role/AmazonEKS_EBS_CSI_DriverRole --region <region-name>

Add prometheus and grafana helm repo
------------------------------------
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo add grafana https://grafana.github.io/helm-charts

Create namespace 
----------------
    kubectl create ns prometheus
    kubectl create ns grafana

Deploy grafana using below command
-----------------------------------
    helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2" \
    --set server.service.type=LoadBalancer

Now check for the pod and service running (Make sure that every container and service is in running state)
----------------------------------------------------------------------------------------------------------
    kubectl get all -n prometheus
    
<img width="788" alt="image" src="https://github.com/Abhi-chintu/prometheus-grafana-on-eks/assets/94033251/b2bd61c2-c9a9-4a4a-aa07-90150f4dd339">

 **Copy and paste the load balancer in browser to see the UI of prometheus**

Deploying Grafana using helm repo
---------------------------------

     mkdir -p monitoring/grafana
     vi monitoring/grafana/grafana.yaml

    #Paste this code in the file
    
     datasources:
        datasources.yaml:
          apiVersion: 1
          datasources:
          - name: Prometheus
            type: prometheus
            url: http://prometheus-server.prometheus.svc.cluster.local
            access: proxy
            isDefault: true

      #install grafan with the below command
      helm install grafana grafana/grafana \
      --namespace grafana \
      --set persistence.storageClassName="gp2" \
      --set persistence.enabled=true \
      --set adminPassword='SecretPass' \
      --values ~/monitoring/grafana/grafana.yaml \
      --set service.type=LoadBalancer

Now check for the pod and service running (Make sure that every container and service is in running state)
----------------------------------------------------------------------------------------------------------
    kubectl get all -n grafana

   <img width="789" alt="image" src="https://github.com/Abhi-chintu/prometheus-grafana-on-eks/assets/94033251/ccac13ec-f062-4fea-8f71-e704132ce790">

**Copy and paste the load balancer in browser to see the UI of prometheus**


          





    

     


