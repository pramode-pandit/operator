#### Openshift 4.6 Documentaion

> https://docs.openshift.com/container-platform/4.6/logging/cluster-logging-deploying.html

> https://docs.openshift.com/container-platform/4.6/logging/config/cluster-logging-configuring-cr.html

> https://docs.openshift.com/container-platform/4.6/logging/cluster-logging-external.html

#### Setting up openshift cluster logging operator

Create the elasic operator 
```
oc create -f eo-namespace.yaml
oc create -f eo-og.yaml
oc create -f eo-sub.yaml
```

Create the cluser logging operator
```
oc create -f clo-namespace.yaml
oc create -f clo-og.yaml
oc create -f clo-sub.yaml
```

Create the operand cluster logging and log forwarder
```
oc create -f instance-cluster-logging.yaml
oc create -f instance-cluster-log-forwarder.yaml
```

