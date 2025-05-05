# linode-dbaas-acl
daemonset and associated manifests to autoupdate Linode DBaaS ACLs
Step 1 - Configure your configmaps to work with your DBaaS and Linode
cm dbinfo has 2 kv pair - dbtype which should be either mysql or postgresql and then the instanceid, which is the 6 digit id of your managed db (you can see this in the cloud manager url when you open that db cluster). Leave the "" around the instanceid.
```
  kind: ConfigMap
  metadata:
    name: dbinfo
    namespace: default
  data:
    dbtype: postgresql
    instanceid: "6digits"
```
cm acl holds any static cidrs or hosts outside of your k8s cluster that you want to have access to the db cluster, adjust these as needed. They can be ipv6 or ipv4, I would suggest adding both to ensure things work.
```
  kind: ConfigMap
  metadata:
    name: acl
    namespace: default
  data:
    static-cidrs: |
      8.8.8.8/32
      2600:3c06::f03c:95ff:feb2:464b/128
```      
Now make sure you create a Linode PAT with permissions to write to database. Enter that token here in your secret. You can also use another secrets management method here.
```
  kind: Secret
  metadata:
    name: update-token
    namespace: default
  type: Opaque
  data:
    update-token: placeyourtokenhere
```
Deploy the update_dbaas.yaml via kubectl apply -f and this should keep your dbaas cluster updated with your static ip list and all k8s nodes in your cluster (both ipv4 and ipv6 addresses) to ensure you can access the cluster and no one else can. This will sleep and rerun on all nodes ever 24 hours with a random wait period of 1 - 60 seconds before starting the first time.
