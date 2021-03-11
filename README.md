# CP4D35TSHINTS

## INSTALL CP4D

### The CP4D AWS INSTALLATION and the Architecture diagram.
https://github.ibm.com/IIG/cp4d-deployment/tree/upi-branch/selfmanaged-openshift/aws

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
