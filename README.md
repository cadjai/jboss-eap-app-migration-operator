# User Guide

This guide walks through using the JBoss EAP Application Migration operator 
powered by Ansible and the k8s module using tools and libraries provided by the Operator SDK.

The operator is used to perform Java Application migration from an existing containerized app
to a new one when a major update is made to the image, which will not otherwise be caught by the 
existing ImageStream. 

For now the user is expected to provide a Custom Resource with the existing and new namespaces 
as well as the images (current and new).

The first phase of this migration will use the lift and shift approach to migrate the application
by pulling existing k8 objects from the existing namespace where the current application is running,
export them and clean them then deploy them to the new namespace. 
Some of the k8 objects exported and reimported are Secrets, SecviceAccounts, RoleBinding, ConfigMaps,
ImageStreams, BuildConfigs, DeploymentConfigs and will be expanded upon in later phases. 
Note that the configuration of the JBoss EAP alongside the current running binary are also exported.
In the first phase the configuration is not processed but later tools will be put in place to enable
diffing of the configuration before applying the reconciled version to the new migrated image.

In short the jboss-eap-app-migration-operator migrates an existing JBoss EAP application when a new 
major version of the image is released and then maintains the application instance.
This operator also maintains a single instance of the operator within the OpenShift Service Catalog on the cluster.


### Deploying the JBoss EAP App Migration Operator
This operator is deployed the same way any other operator is deployed. 
#### 1. Manually deploy the operator to a cluster
If doing it manually follow these steps once you have clone the operator source
Build the jboss-eap-app-migration-operator image and push it to a registry:
```sh
$ operator-sdk build quay.io/${YOURQUAYID}/jboss-eap-app-migration-operator:v0.0.1
$ sed -i 's|REPLACE_IMAGE|quay.io/${YOURQUAYID}/jboss-eap-app-migration-operator:v0.0.1|g' deploy/operator.yaml
$ docker push quay.io/${YOURQUAYID}/jboss-eap-app-migration-operator:v0.0.1
```sh

The Deployment manifest is generated at `deploy/operator.yaml`. Be sure to update the deployment image as shown above since the default is just a placeholder.

Create the Namespace the operator will run in, setup RBAC and deploy the jboss-eap-app-migration-operator:

```sh
$ kubectl create namespace jboss-eap-app-migration-operator
$ kubectl create -f deploy/crd/rhtconsulting_v1alpha1_jbosseapappmigration_crd.yaml
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
$ kubectl create -f deploy/operator.yaml
```

Verify that the jboss-eap-app-migration-operator is up and running:

```sh
$ kubectl get deployment
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
jboss-eap-app-migration-operator       1         1         1            1           1m

#### 2. Deploy the operator through the OLM





### Using the operator to migrate an application
Update the sample Custom Resource found under the deploy/crds sub folder of the operator and then deploy it.

```sh
$ kubectl create -f deploy/crds/migrate-kitchensink.yaml
```sh

Watch the operator by either watching the various resources being created within the new namespace or tail 
the operator log 

```sh
$ kubectl tail -f jboss-eap-app-migration-operator-334768tr-uiuhhj -c ansible
```sh

