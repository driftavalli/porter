![GitHub version](https://img.shields.io/badge/version-v0.0.1-brightgreen.svg?logo=appveyor&longCache=true&style=flat)
![go report](https://goreportcard.com/badge/github.com/kubesphere/porter)

# Porter
`Porter` is a load balancer for physical machine deployment Kubernetes. The load balancer is implemented using physical switches, leveraging BGP and ECMP for optimal performance and high availability. We know that in the Kubernetes environment deployed on the cloud, the cloud service provider usually provides the cloud LB plugin to expose the Kubernetes service to the external network. However, in the physical machine deployment environment, because the cloud environment is not available, the service is exposed to the external network. Porter is A plug-in that provides users with exposed services in the physical environment and exposed service consistency experiences on the cloud. The plugin provides two major functional modules:

1. The LB controller and the agent: controller are responsible for synchronizing the BGP routes to the physical switch; the agent is deployed to the node in the DaemonSet mode to maintain the drainage rules.
2. The EIP service, including the EIP pool management and EIP controller, is responsible for updating the EIP information of the service.

Porter is a subproject of [KubeSphere](https://kubesphere.io/). 

## Physical deployment architecture
The following figure is a physical deployment architecture diagram. Suppose a service is deployed on node1 (192.168.0.2) and node2 (192.168.0.6). The service needs to be accessed through public IP 1.1.1.1. After the service deployer deploys the service according to the example. Porter will automatically synchronize routing information to the leaf switch, and then synchronize to the spine, border switch, Internet users can directly access the service through EIP 1.1.1.1.


![node architecture](doc/img/node-arch.png)

Plug-in deployment architecture

The plug-in monitors changes in the service in the cluster through a Manager and broadcasts related routes. At the same time, all the nodes in the cluster are deployed with an agent. Whenever an EIP is used, a host routing rule is added to the host to divert the IP packets sent to the EIP to the local device.


![porter deployment](doc/img/porter-deployment.png)

## Plugin Logic

When the plug-in is deployed as a service in a Kubernetes cluster, it establishes a BGP connection with the cluster's border router (Layer 3 switch). Whenever a service with a specific annotation (an annotation is lb.kubesphere.io/v1apha1: porter, see [example] (config/sample/service.yaml)) is created in the cluster, it is dynamically allocated for the service. EIP (users can also specify EIPs themselves). The LB controller creates routes and routes them to the public network (private network) through BGP, so that external services can be accessed.

The Porter LB controller is a custom controller based on [Kubernetes controller runtime] (https://github.com/kubernetes-sigs/controller-runtime) that automatically changes routing information through changes to the watch service.

![porter architecture](doc/img/porter-arch.png)


## Deploying plugins

1. [Deployed on a physically deployed k8s cluster] (doc/deploy_baremetal.md)
2. [Testing with an analog router on Qingyun] (doc/simulate_with_bird.md)

## Building a new plugin from code

### Software Requirements

1. go 1.11, the plugin uses [gobgp](https://github.com/osrg/gobgp) to create the BGP server, gobgp needs go 1.11
2. docker, no version limit
3. kustomize, the plugin uses [kustomize] (https://github.com/kubernetes-sigs/kustomize/blob/master/docs/INSTALL.md) to dynamically generate the k8s yaml files needed for the cluster
4. If the plugin is pushed to the remote private repository, you need to execute `docker login` in advance.

### Step

1. `git clone https://github.com/kubesphere/porter.git`, enter the code directory
2. Modify config.toml (located under `config/bgp/`) as required in the tutorial above.
3. (optional) modify the code according to your needs
4. (optional) modify the parameters of the image according to your needs (located under `config/manager`)
5. (optional) Follow the [Simulation Tutorial] (doc/simulate_with_bird.md) to deploy a Bird host, modify the BirdIP in `hack/e2e.sh`, and then run `make e2e-test` for e2e testing.
6. Modify the IMG name in the Makefile, then `make release`, and the final yaml file in the `deploy` directory
7. `kubectl apply -f deploy/release.yaml` deployment plugin

## Open source license

**Porter** is licensed under the Apache License, Version 2.0. See [LICENSE](./LICENSE) for the full license text.
