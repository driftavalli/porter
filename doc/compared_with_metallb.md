# Compared with MetalLB

## Advantage
- Supports most of the features of the BGP protocol. Support multiple network architectures.
- K8s friendly. Based on the CRD-Controller mode, use kubectl to control everything about porter.
- The configuration file is dynamically updated, and the BGP configuration is automatically updated without restarting. Configure BGP flexibly with respect to your network environment and dynamically enable various BGP features.
- Ability to coexist and handle conflicts with Calico, throught the use of Passive mode and port forwarding mode

## Disadvantage
- It is not cross platform, only supports Linux
- L2 is not supported (load balancing on L2 is hard to achieve, and it is easy to create a performance bottleneck at a  single point)

## Commonality
- We will need more tests
