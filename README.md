# memcached-operator
// TODO(user): Add simple overview of use/purpose

## Description
1. init operator
```
$ operator-sdk init --domain=siqi.ibm.com --repo=github.com/siqiliu18/memcached-operator
INFO[0000] Writing kustomize manifests for you to edit... 
INFO[0000] Writing scaffold for you to edit...          
INFO[0000] Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.16.3 
INFO[0002] Update dependencies:
$ go mod tidy           
Next: define a resource with:
$ operator-sdk create api
```
2. Let’s take a look at what was generated:
```
a Dockerfile for the controller image, which will be built out of your project
a Makefile to build, test, deploy, and undeploy the project
a PROJECT file with the domain and project layout configuration
a bin directory that contains the binaries for the controller
a config directory that contains:
base manifests for deploying the CRD and controller of the operator
Kustomization YAML for customizing manifests
RBAC YAML to authorize the various components to interact with each other
a Patch file for enabling Prometheus metrics
some sample YAML for creating a simple instance of our new type
go.mod and go.sum files for managing the Golang depedences of the project
a hack directory containing a boilerplate license file
main.go, which contains the main function for your controller
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

