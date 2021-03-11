# CP4D35TSHINTS

## INSTALL CP4D

### The CP4D AWS Installation and Architecture Diagram
https://github.ibm.com/IIG/cp4d-deployment/tree/upi-branch/selfmanaged-openshift/aws

### The CP4D GCP Installation and Architecture Diagram
We will need to install the OCP on GCP followed by manual install of CP4D services. 

Pls find some documentation regarding the OCP installation on GCP 
https://github.com/ibm-cloud-architecture/terraform-openshift4-gcp

## DEBUG

### Increase the timeout at the AWS load balancer level 
Change idle timeout to 30 minutes (1800 seconds instead of default 30 seconds) for the load balancer.

https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/config-idle-timeout.html 

### Check Docker logs
docker logs containername/id 

### Check the DV warnings
db2 "call dvsys.listremotewarnings()" > /tmp/listremotewarning.del

### Extract the warnings file outside
oc cp dv-engine-0:/tmp/listremotewarning.del listremotewarning.del
tar: Removing leading `/' from member names

### To collect DV Engine logs:
ssh to the cluster

run: oc exec -it dv-engine-0 /opt/dv/current/dv-engine-collect-logs.sh

### To collect DV Utils logs:
ssh to the cluster

run: oc exec -it dv-utils-0 /opt/dv/current/dv-utils-collect-logs.sh

### To collect DV Worker logs (from a single worker):
ssh to the cluster
run: oc exec -it dv-worker-0 /opt/dv/current/dv-worker-collect-logs.sh

### In all cases, log archives are produced at /opt/dv/current/logs/.

Log archive names:

dv-engine: dv-engine-bundle-YYYYmmddHHMMSS.zip  eg. dv-engine-bundle-20210211204434.zip

dv-utils: dv-utils-logs-YYYYmmddHHMMSS.zip

dv-worker:  dv-worker-bundle-YYYYmmddHHMMSS.zip

### Check the CP4D DV version from the cluster

oc get pods | grep -i zen-core-api 

You will see two zen core api pod, pick one 

Run oc get pod ${zen_core_api_you_picked} -o yaml > zen-core-api.yaml

cat zen-core-api.yaml | grep "image:"

### For the CP4D cluster we can create two separate labels, one for dv-engine, one for dv-worker, so they each run on nodes with the corresponding labels - Anti-Affinity

The example It uses "dv-dedicated" for both the engine and worker, you can change it to be

dv-dedicated-engine for the dv-engine pod

dv-dedicated-worker for the dv-worker pod

Then we can change the node affinity rule, make sure you have backups of dv-engine and dv-worker statefulset ( mentioned in the instruction below )

Total number of dv engine and dv-worker pod must be less or equal to the dedicated nodes you have.  

In our case we have 3 dedicated nodes, so 1 head 2 worker is the maximum.

#### For dv-engine

oc get statefulset dv-engine -o yaml > dv-engine-statefulset.yaml

This is to make a copy of the dv-engine statefulset just in case you make mistakes, so you can use it to add back correct fields and values.

oc edit statefulset dv-engine

Find section "affinity:" And you should see one "preferredXXXX" section and one "requiredXXXX" section

          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: dv-dedicated
                operator: Exists

Remove the entire preferred section and edit the required section so it looks just like the following i.e two rules in the required section

   spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
              - key: dv-dedicated-engine
                operator: Exists

#### For dv-worker

oc get statefulset dv-worker -o yaml > dv-worker-statefulset.yaml 

This is to have a backup in case things get messed up, so you can use it add back correct fields and values.

oc edit statefulset dv-worker

Find the preferred section, remove and edit required section, two rules

   spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
              - key: dv-dedicated-worker
                operator: Exists

