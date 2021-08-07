
### Operator Lifecycle Manager

The Operator Lifecycle Manager project is a component of the 
Operator Framework, an open source toolkit to manage Kubernetes 
native applications, called Operators, in an effective, automated, 
and scalable way.

OLM extends Kubernetes to provide a declarative way to install, 
manage, and upgrade operators and their dependencies in a cluster. 
It also enforces some constraints on the components it manages in 
order to ensure a good user experience.

> https://olm.operatorframework.io/

> https://operatorhub.io/

#### Installing OLM in your cluster

OLM runs by default in OpenShift Container Platform 4.5, which aids 
cluster administrators in installing, upgrading, and granting 
access to Operators running on their cluster and can be easily be installed 
in a non-OpenShift Kubernetes environment by running simple command.

**Prerequisites for a raw kubernetes cluster**
- Access to a Kubernetes v1.11.3+ cluster.
- kubectl v1.11.3+.

Installation can be performed following the below page on quicksetup on a 
vanilla kubernetes cluster

> https://olm.operatorframework.io/docs/getting-started/

Or a specific version of olm can be installed with the install script 
from operatorhub.io 
> https://operatorhub.io/how-to-install-an-operator#

Or from olm release page
> https://github.com/operator-framework/operator-lifecycle-manager/releases

Quickinstall here

```
# Skip if you already a cluster with you 
kind create cluster --name operator --image kindest/node:v1.18.4
```

```
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.18.3/install.sh | bash -s v0.18.3
```

<br/>
<br/>

#### Exploring OLM

Let's get started exploring OLM by viewing its Custom Resource Definitions (CRDs).

OLM ships with a below CRDs in Kind Cluster:

```
$ kubectl get crd
catalogsources.operators.coreos.com           2021-08-01T06:54:51Z
clusterserviceversions.operators.coreos.com   2021-08-01T06:54:52Z
installplans.operators.coreos.com             2021-08-01T06:54:52Z
operatorconditions.operators.coreos.com       2021-08-01T06:54:52Z
operatorgroups.operators.coreos.com           2021-08-01T06:54:52Z
operators.operators.coreos.com                2021-08-01T06:54:52Z
subscriptions.operators.coreos.com            2021-08-01T06:54:52Z
````


OLM ships with 6 CRDs in Openshift 4.5:

```
$ oc get crd | grep -E 'operators.coreos.com'
catalogsources.operators.coreos.com                         2020-07-29T04:45:11Z
clusterserviceversions.operators.coreos.com                 2020-07-29T04:45:16Z
installplans.operators.coreos.com                           2020-07-29T04:45:21Z
operatorgroups.operators.coreos.com                         2020-07-29T04:45:29Z
operatorsources.operators.coreos.com                        2020-07-29T04:45:13Z
subscriptions.operators.coreos.com                          2020-07-29T04:45:35Z  
```


- **CatalogSource:**

    - Its collection of Operator metadata (ClusterServiceVersions, CRDs, and 
      PackageManifests) that define an application.
    - OLM uses CatalogSources to build the list of available 
      operators that can be installed from OperatorHub in the OpenShift web 
      console. 
    - In OpenShift 4.5, the web console has added support for managing 
      the out-of-the-box CatalogSources as well as adding your own custom 
      CatalogSources. You can create a custom CatalogSource using the [OLM 
      Operator Registry](https://github.com/operator-framework/operator-registry).

- **OperatorGroup:**

    - Configures all Operators deployed in the same namespace as the 
      OperatorGroup object to watch for their Custom Resource (CR) in a list 
      of namespaces or cluster-wide.
    - [configuring-operatorgroups](https://operator-framework.github.io/olm-book/docs/operator-scoping.html#configuring-operatorgroups)


- **Subscription:**

    - Relates an operator to a CatalogSource. 
    - Keeps CSVs up to date by tracking a channel in a package.
    - Subscriptions describe which channel of an operator package to subscribe 
      to and whether to perform updates automatically or manually. 
      If set to automatic, the Subscription ensures OLM will manage and upgrade 
      the operator to ensure the latest version is always running in the cluster.
    - [manually-approving-upgrades-via-subscriptions](https://operator-framework.github.io/olm-book/docs/subscriptions.html#manually-approving-upgrades-via-subscriptions)

- **InstallPlan:**

    - Calculated list of resources to be created in order to automatically 
      install or upgrade a CSV.

- **ClusterServiceVersion (CSV):**

    - The metadata that accompanies your Operator container image. 
    - It can be used to populate user interfaces with info like your 
      logo/description/version and it is also a source of technical information 
      needed to run the Operator. It includes RBAC rules and which Custom 
      Resources it manages or depends on. 
    - OLM will parse this and do all of the hard work to wire up the correct 
      Roles and Role Bindings, ensuring that the Operator is started (or updated) 
      within the desired namespace and check for various other requirements, 
      all without the end users having to do anything. 

- **OperatorCondition :**

    - An OperatorCondition is CustomResourceDefinition that creates a communication between OLM and an operator it manages. 
    - Operators may write to the Status.Conditions array to modify OLM management the operator.

- **PackageManifest:**

    - An entry in the CatalogSource that associates a package identity with 
      sets of CSVs. Within a package, channels point to a particular CSV. 
    - Because CSVs explicitly reference the CSV that they replace, a 
      PackageManifest provides OLM with all of the information that is 
      required to update a CSV to the latest version in a channel (stepping 
      through each intermediate version).

      ```
        # Kind Cluster
        kubectl api-resources | grep operators.coreos.com
        
        # Openshift
        oc api-resources | grep operators.coreos.com
      ```

OLM is powered by controllers that reside within the 
openshift-operator-lifecycle-manager namespace as three Deployments 
(catalog-operator, olm-operator, and packageserver):

```
$  kubectl get deploy -n olm
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
catalog-operator   1/1     1            1           8h
olm-operator       1/1     1            1           8h
packageserver      2/2     2            2           8h
```

```
$ oc get deploy -n openshift-operator-lifecycle-manager 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
catalog-operator   1/1     1            1           368d
olm-operator       1/1     1            1           368d
packageserver      2/2     2            2           368d
```



#### CatalogSources

**Observe the CatalogSources that ship with OLM and Kind cluster:**

- Only community operators are available 

```
kubectl get catalogsources -A
NAMESPACE   NAME                    DISPLAY               TYPE   PUBLISHER        AGE
olm         operatorhubio-catalog   Community Operators   grpc   OperatorHub.io   81m
```

```
$ kubectl get pods -n olm
NAME                                READY   STATUS    RESTARTS   AGE
catalog-operator-7459c7fd97-n6gr9   1/1     Running   0          74m
olm-operator-7765c55559-nxqr5       1/1     Running   0          74m
operatorhubio-catalog-njmr4         1/1     Running   0          73m
packageserver-655d979b75-5lzqn      1/1     Running   0          73m
packageserver-655d979b75-6c8zn      1/1     Running   0          73m
```

```
kubectl get packagemanifest -l catalog=operatorhubio-catalog
```



**Observe the CatalogSources that ship with OLM and OpenShift 4**

- Along with community, operators are also available from various redhat sources

```
$ oc get catalogsources -n openshift-marketplace
NAME                  DISPLAY               TYPE   PUBLISHER   AGE
certified-operators   Certified Operators   grpc   Red Hat     368d
community-operators   Community Operators   grpc   Red Hat     368d
redhat-marketplace    Red Hat Marketplace   grpc   Red Hat     368d
redhat-operators      Red Hat Operators     grpc   Red Hat     368d
```

```
$ oc get pods -n openshift-marketplace
NAME                                   READY   STATUS             RESTARTS   AGE
certified-operators-6bbd96cbd8-92z6g   1/1     Running            0          4h40m
community-operators-544c5b9887-bst7d   1/1     Running            0          368d
community-operators-6945fb7444-7fx9x   1/1     Running            0          4h40m
marketplace-operator-df484cccc-dg5vs   1/1     Running            0          368d
redhat-marketplace-5fb8f6c8dc-5zhl4    1/1     Running            0          4h40m
redhat-operators-544945d95f-w287p      1/1     Running            0          4h40m
```

```
oc get packagemanifests -l catalog=certified-operators
oc get packagemanifests -l catalog=community-operators
oc get packagemanifests -l catalog=redhat-operators
oc get packagemanifests -l catalog=redhat-marketplace
```

Here is a brief summary of each CatalogSource on Openshift:

- Certified Operators:

  - All Certified Operators have passed Red Hat OpenShift Operator Certification, an offering under Red Hat Partner Connect, our technology partner program. In this program, Red Hat partners can certify their Operators for use on Red Hat OpenShift. With OpenShift Certified Operators, customers can benefit from validated, well-integrated, mature and supported Operators from Red Hat or partner ISVs in their hybrid cloud environments.

- Community Operators:

  - With access to community Operators, customers can try out Operators at a variety of maturity levels. Delivering the OperatorHub community, Operators on OpenShift fosters iterative software development and deployment as Developers get self-service access to popular components like databases, message queues or tracing in a managed-service fashion on the platform. These Operators are maintained by relevant representatives in the operator-framework/community-operators GitHub repository.

- Red Hat Operators:

  - These Operators are packaged, shipped, and supported by Red Hat.

- Red Hat Marketplace:

  - Built in partnership by Red Hat and IBM, the Red Hat Marketplace helps organizations deliver enterprise software and improve workload portability. Learn more at marketplace.redhat.com.


#### Creating the OperatorGroup and Subscription

Operator group available in Kind Cluster

```
$ kubectl get og -A
NAMESPACE   NAME               AGE
olm         olm-operators      102m
operators   global-operators   102m
```

Operator group available in Openshift

```
$ oc get og -A
NAMESPACE                              NAME                           AGE
openshift-monitoring                   openshift-cluster-monitoring   368d
openshift-operator-lifecycle-manager   olm-operators                  368d
openshift-operators                    global-operators               368d
```

```
# Kind Cluster
kubectl create ns myrojet

# Openshift
oc new-project myproject
```

**Create the OperatorGroup:**

We should first create an OperatorGroup to ensure Operators installed 
to this namespace will be capable of watching for Custom Resources within the myproject namespace:

```
cat > argocd-operatorgroup.yaml <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: argocd-operatorgroup
  namespace: myproject
spec:
  targetNamespaces:
    - myproject
EOF
```


```
# Kind Cluster
kubectl create -f argocd-operatorgroup.yaml
kubectl get og -n myproject


# Openshift
oc create -f argocd-operatorgroup.yaml
oc get og -n myproject
```

**Create the subscription:**

Create a Subscription manifest for the ArgoCD Operator. 
Ensure the installPlanApproval is set to Manual. 
This will allow us to review the InstallPlan before choosing to install the 
Operator:

```
# Kind Cluster
cat > argocd-subscription.yaml <<EOF
apiVersion: operators.coreos.com/v1alpha1 
kind: Subscription 
metadata: 
  name: my-argocd-operator 
  namespace: myproject 
spec: 
  channel: alpha 
  installPlanApproval: Manual
  name: argocd-operator 
  source: operatorhubio-catalog 
  sourceNamespace: olm
EOF
```

```
# Openshift
cat > argocd-subscription.yaml <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: argocd-operator
  namespace: myproject 
spec:
  channel: alpha
  installPlanApproval: Manual
  name: argocd-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
EOF
```

```
# Kind Cluster
kubectl create -f argocd-subscription.yaml
kubectl get subscription -n myproject
kubectl get installplan -n myproject

# Openshift
oc create -f argocd-subscription.yaml
oc get subscription -n myproject
oc get installplan -n myproject
```

#### Reviewing the InstallPlan and Installing the Operator

Fetch the InstallPlan and observe the Kubernetes objects that will 
be created once approved:

```
ARGOCD_INSTALLPLAN=`oc get installplan -o jsonpath={$.items[0].metadata.name}`

oc get installplan $ARGOCD_INSTALLPLAN -o yaml
```

You can get a better view of the InstallPlan by navigating to the ArgoCD 
Operator in the Console.

Navigate to the Operators section of the UI and select the ArgoCD 
Operator under Installed Operators. Ensure you are scoped to the 
myproject namespace. 

Approve via oc path from cli

```
oc patch installplan $ARGOCD_INSTALLPLAN --type='json' -p '[{"op": "replace", "path": "/spec/approved", "value":true}]'
```

Verify the operator installtion

```
oc get ip -n myproject

oc get clusterserviceversion -n myproject

oc get crd | grep argoproj.io

oc get sa | grep argocd

oc get roles | grep argocd

oc get rolebindings | grep argocd

oc get deployments
```

Follow the same steps with kubectl command to review and install operantor


#### Creating the Custom Resource

Let's deploy our ArgoCD Server "operand" by creating the ArgoCD 
manifest via the CLI. We can also do this on the OpenShift console.

```
# Kind Cluster
cat > argocd-cr.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  namespace: myproject
spec: {}
EOF
```

```
# Openshift
cat > argocd-cr.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  namespace: myproject
spec:
  dex:
    image: quay.io/ablock/dex
    openShiftOAuth: true
    version: openshift-connector
  rbac:
    policy: |
      g, system:cluster-admins, role:admin
  server:
    route:
      enabled: true
EOF
```

Create the ArgoCD Custom Resource:

```
# Kind Cluster
kubectl create -f argocd-cr.yaml

# Openshift
oc create -f argocd-cr.yaml
```

The ArgoCD Operator should now begin to generate the ArgoCD Operand artifacts. This can take up to one minute:

```
# Kind Cluster
kubectl get deployments -n myproject
kubectl get secrets -n myproject
kubectl get services -n myproject
kubectl get routes -n myproject

# Openshift
oc get deployments -n myproject
oc get secrets -n myproject
oc get services -n myproject
oc get routes -n myproject
```

#### Accessing the ArgoCD Dashboard

Let's access the ArgoCD Dashboard via an OpenShift Route:

```
ARGOCD_ROUTE=`oc get routes example-argocd-server -o jsonpath={$.spec.host}`
echo $ARGOCD_ROUTE
```
On the ArgoCD webpage, select Login via OpenShift to use OpenShift as our identity provider.

For Kind cluster port forward the service and access as http://localhost:8080

```
kubectl -n myproject port-forward svc/      8080:80
```

#### Uninstalling the Operator

We can easily uninstall your operator by first removing the ArgoCD Custom 
Resource. Removing the ArgoCD Custom Resource, should remove all of the 
Operator's Operands

```
oc delete argocd example-argocd -n myproject
oc get deployments -n myproject
```

And then uninstalling the Operator:

```
oc delete -f argocd-subscription.yaml
ARGOCD_CSV=`oc get csv -o jsonpath={$.items[0].metadata.name}`
oc delete csv $ARGOCD_CSV
```

Once the Subscription and ClusterServiceVersion have been removed, 
the Operator and associated artifacts will be removed from the cluster.

```
oc get pods -n myproject
oc get roles -n myproject
```

For Kind Cluster, follow the same set of commands with kubectl cli


