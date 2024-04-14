# memcached-operator
**Class**: https://apps.course-dev.skills.network/learning/course/course-v1:IBMSkillsNetwork+CO0201EN+2021T1/home

## Description
1. init operator >> operator-sdk init --domain=siqi.ibm.com --repo=github.com/siqiliu18/memcached-operator
2. Let’s take a look at what was generated:
```
- a Dockerfile for the controller image, which will be built out of your project
- a Makefile to build, test, deploy, and undeploy the project
- a PROJECT file with the domain and project layout configuration
- a bin directory that contains the binaries for the controller
- a config directory that contains:
    - base manifests for deploying the CRD and controller of the operator
    - Kustomization YAML for customizing manifests
    - RBAC YAML to authorize the various components to interact with each other
    - a Patch file for enabling Prometheus metrics
    - some sample YAML for creating a simple instance of our new type
- go.mod and go.sum files for managing the Golang depedences of the project
- a hack directory containing a boilerplate license file
- main.go, which contains the main function for your controller
```
3. Create an API >> operator-sdk create api --group cache --version v1alpha1 --kind Memcached --resource=true --controller=true
```
--group is the group where your custom resource is going to live, so it will end up in the API group ‘cache.example.com’.
--version determines the version of your API group. You can use different versions to successively upgrade your custom resource.
--resource and --controller flags are set to true to generate the scaffolding for both those things.

Let’s take a look at what changed:
- PROJECT was updated to include the new scaffolding
- go.mod was updated with new depdencies
- main.go was updated to add the new controller to the controller-manager loop

Some of the new generated artifacts are:
- an API directory that contains the files defining your CRD
- YAML manifests in config/ for deploying your CRD
- YAML manifests in config/ for creating RBAC to allow your controller to watch/edit your Memcached type
- YAML manifests in config/samples/ for creating a sample instance of your Memcached type
- a controllers directory, that contains the code for your Memcached controller
```
4. Update api/v1alpha1/memcached_types.go file to specify the *spec* and *status* fields
5. run >> make generate
```
This make target invokes controller-gen to update api/v1alpha1/zz_generated.deepcopy.go to contain
the necessary implementations for the fields you just added. Once that is done,
you should run the following command to generate a YAML manifests for your CRD:
```
7. run >> make manifests
```
The generated config/crd/bases/cache.example.com_memcacheds.yaml is the manifests for your memcached CRD.
config/rbac/role.yaml is an RBAC manifest that contains the permissions to manage your memcached type that is needed by the controller.
```
8. Create controller:
    - The Reconcile method is responsible for reconciling the desired state contained in the custom resource’s status with the actual state running on the system and is
where the lion’s share of the controller logic is implemented. The specifics of implementing the reconcile loop is beyond this tutorial, and will
be covered in the advanced module. For now, replace controllers/memcached_controller.go with this [reference implementation](https://github.com/operator-framework/operator-sdk/blob/efd1235e842032a3402e2c1ae6d9beb9ee86019a/testdata/go/v3/memcached-operator/controllers/memcached_controller.go).
    - Note that you may have to change the import path for github.com/example/memcached-operator/api/v1alpha1 to correctly point to the directory you defined your memcached_types.go
in if you chose to specify a different repo when you initialized your project. After pasting that in, be sure to regenerate the manifests:
9. Build and deploy your operator
    - As a program executing outside the Kubernetes cluster. This might be done for development purposes or for security concerns of the data contained in the cluster. The Makefile contains the target make install run to run the operator locally, but this method is not the focus here.
    - Run as a Deployment inside the Kubernetes cluster. This is what I will show you now.
    - Deployed and managed by Operator Lifecycle Manager. This is recommended for production use because OLM has additional features for the management and upgrade of a running opeartor.
```
$ export USERNAME=<docker-username>
$ make docker-build docker-push IMG=docker.io/$USERNAME/memcached-operator:v1.0.0

$ make deploy IMG=docker.io/$USERNAME/memcached-operator:v1.0.0

$ kubectl get crds
NAME                                          CREATED AT
catalogsources.operators.coreos.com           2021-01-22T00:13:22Z
clusterserviceversions.operators.coreos.com   2021-01-22T00:13:22Z
installplans.operators.coreos.com             2021-01-22T00:13:22Z
memcacheds.cache.example.com                   2021-02-06T00:52:38Z
operatorgroups.operators.coreos.com           2021-01-22T00:13:22Z
rbacsyncs.ibm.com                             2021-01-22T00:08:59Z
subscriptions.operators.coreos.com            2021-01-22T00:13:22Z

$ kubectl --namespace memcached-operator-system get deployments
NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
memcached-operator-controller-manager   1/1     1            1           2m18s
$ kubectl --namespace memcached-operator-system get pods 
NAME                                                     READY   STATUS    RESTARTS   AGE
memcached-operator-controller-manager-76b588bbb5-wvl7b   2/2     Running   0          2m44s

$ kubectl get clusterroles | grep memcached
memcached-operator-manager-role                                        2021-02-06T00:52:38Z
memcached-operator-metrics-reader                                      2021-02-06T00:52:39Z
memcached-operator-proxy-role                                          2021-02-06T00:52:39Z
$ kubectl get clusterrolebindings | grep memcached
memcached-operator-manager-rolebinding                 ClusterRole/memcached-operator-manager-role                        3m
memcached-operator-proxy-rolebinding                   ClusterRole/memcached-operator-proxy-role                          3m
$ kubectl --namespace memcached-operator-system get roles
NAME                                      CREATED AT
memcached-operator-leader-election-role   2021-02-06T00:52:38Z
$ kubectl --namespace memcached-operator-system get rolebindings
NAME                                             ROLE                                           AGE
memcached-operator-leader-election-rolebinding   Role/memcached-operator-leader-election-role   13m

```
10. Edit the sample YAML at config/samples/cache_v1alpha1_memcached.yaml to include a size integer like you defined in your custom resource spec:
```
$ kubectl apply -f config/samples/cache_v1alpha1_memcached.yaml 
memcached.cache.example.com/memcached-sample created

$ kubectl get memcached
NAME               AGE
memcached-sample   18s
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
memcached-sample   1/1     1            1           33s
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
memcached-sample-9b765dfc8-2hvf8   1/1     Running   0          44s

$ kubectl get memcached memcached-sample -o yaml
apiVersion: cache.siqi.ibm.com/v1alpha1
kind: Memcached
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"cache.siqi.ibm.com/v1alpha1","kind":"Memcached","metadata":{"annotations":{},"name":"memcached-sample","namespace":"default"},"spec":{"size":1}}
  creationTimestamp: "2024-04-13T23:52:35Z"
  generation: 1
  name: memcached-sample
  namespace: default
  resourceVersion: "1015284"
  uid: 6b8aaf19-8f7d-4bbd-abb8-2154bab5cbbf
spec:
  size: 1
status:
  nodes:
  - memcached-sample-d5c88d8b5-7jlgg
```
11. **I don't see deployments and pods are created in my local default cluster**, I have to run >> make install run under the project to be able to see deployment and pod
12. To see your controller in action, add another node to your cluster. Change the size in config/samples/cache_v1alpha1_memcached.yaml to 2, and then run:
```
$ kubectl apply -f config/samples/cache_v1alpha1_memcached.yaml
memcached.cache.example.com/memcached-sample configured

$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
memcached-sample-9b765dfc8-2hvf8   1/1     Running   0          50s
memcached-sample-9b765dfc8-jdhlq   1/1     Running   0          3s

$ kubectl get memcached memcached-sample -o yaml
apiVersion: cache.siqi.ibm.com/v1alpha1
kind: Memcached
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"cache.siqi.ibm.com/v1alpha1","kind":"Memcached","metadata":{"annotations":{},"name":"memcached-sample","namespace":"default"},"spec":{"size":2}}
  creationTimestamp: "2024-04-13T23:52:35Z"
  generation: 2
  name: memcached-sample
  namespace: default
  resourceVersion: "1015355"
  uid: 6b8aaf19-8f7d-4bbd-abb8-2154bab5cbbf
spec:
  size: 2
status:
  nodes:
  - memcached-sample-d5c88d8b5-7jlgg
```
13. Cleanup
```
$ kubectl delete memcached memcached-sample
$ make undeploy
```

## Getting Started

### Prerequisites
- go version v1.20.0+
- docker version 17.03+.
- kubectl version v1.11.3+.
- Access to a Kubernetes v1.11.3+ cluster.

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/memcached-operator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified. 
And it is required to have access to pull the image from the working environment. 
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/memcached-operator:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin 
privileges or be logged in as admin.

**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

**NOTE:** Run `make help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

