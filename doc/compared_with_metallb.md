# Compared with MetalLB

## Advantage
- Supports most of the features of the BGP protocol. Support multiple network architectures.
- K8s friendly. Based on the CRD-Controller mode, use kubectl to control everything about the porter.
- The configuration file is dynamically updated, and the BGP configuration is automatically updated without restarting. Configure BGP flexibly according to the network environment and dynamically enable various BGP features.
- More friendly handling of conflicts with Calico, providing Passive mode and port forwarding mode

## Disadvantage
- Can't cross platform, only support linux
- L2 is not supported (we think L2 can't achieve load balancing, and it is easy to cause single point performance bottleneck)

## Commonality
- We all need more tests
