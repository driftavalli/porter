# Physical machine deployment
## Installation prerequisite
1. The router connected to the physical machine must support the BGP protocol.
2. If you need to implement load balancing on the router side, you need the router to support ECMP and include the following features:

      * Support for receiving multiple equal-cost routes
      * Support for receiving multiple equal-cost routes from the same neighbor
3. If there is a router in the network architecture that does not support BGP (or is prohibited from BGP), then you need to manually write the EIP nexthop route on this router (or through other route discovery protocols)
## Porter Installation
1. Install kubernetes on the machine
2. Get yaml
  
    `wget https://github.com/kubesphere/porter/releases/download/v0.1.0/porter.yaml`


3. Modify a configmap named bgp-cfg in yaml, and simply modify some fields according to the BGP configuration tutorial. Pay attention to the address of the router and the AS domain, and make sure that EBGP is established.
4. Label the master node to ensure that the porter is installed on the master node (if you do not want to install it on the master node, you need to have all possible Node nodes on the router side)
    
    `kubectl label nodes name_of_your_master dedicated=master # Please modify first mastername`

5. Install porter into the cluster

    `kubectl apply -f porter.yaml`

## Router configuration
`Configuration for routers will differ based on the Vendor. The configuration below is for a Cisco Layer 3 switch (Nexus 9000 Series)[https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/index.html]. For more details, please refer to your Vendor documentation for the specific router model.`
1. Enter the N9K configuration interface as admin. Modify the following configuration according to the actual situation. (Note: the actual input cannot be commented)
    ```
    Feature bgp ##Enable BGP function

      Router bgp 65001 #Set the AS domain of this router
      Router-id 10.10.12.1 #Set this router IP
      Address-family ipv4 unicast
          Maximum-paths 8 #Enable ECMP and accept up to 8 equal-cost routes
          Additional-paths send # can send multiple equal-cost routes
          Additional-paths receive # can accept multiple equal-cost routes
      Neighbor 10.10.12.5 # neighbor IP
          Remote-as 65000 # neighbor AS, must be different from the local AS
          Timers 10 30
          Address-family ipv4 unicast
          Route-map allow in #Allow the import of the system routing table
          Route-map allow out #Allow export routing table to system
          Soft-reconfiguration inbound always # Automatically update neighbor status
    ```
2. After the configuration is complete, check the neighbor status as Established. Show bgp ipv4 unicast neighbors
    ```
    myswitvh(config)# show bgp ipv4 unicast neighbors

        BGP neighbor is 10.10.12.5, remote AS 65000, ebgp link, Peer index 3
        BGP version 4, remote router ID 10.10.12.5
        BGP state = Established, up for 00:00:02
        Peer is directly attached, interface Ethernet1/1
        Last read 00:00:01, hold time = 30, keepalive interval is 10 seconds
        Last written 0.996717, keepalive timer expiry due 00:00:09
        Received 5 messages, 0 notifications, 0 bytes in queue
        Sent 13 messages, 0 notifications, 0(0) bytes in queue
        Connections established 1, dropped 0
        Last reset by us 00:01:29, due to session closed
        Last reset by peer never, due to No error

        Neighbor capabilities:
        Dynamic capability: advertised (mp, refresh, gr)
        Dynamic capability (old): advertised
        Route refresh capability (new): advertised received
        Route refresh capability (old): advertised
        4-Byte AS capability: advertised received
        Address family IPv4 Unicast: advertised received
        Graceful Restart capability: advertised
    ```

## Deployment example
1. Add an EIP
    ```
    Kubectl apply -f - <<EOF
    apiVersion: network.kubesphere.io/v1alpha1
    Kind: EIP
    Metadata:
    Labels:
         Controller-tools.k8s.io: "1.0"
    Name: eip-sample
    Spec:
    # Add fields here
         Address: 10.11.11.11 #This is replaced by the EIP you applied for.
         Disable: false
    EOF
    ```
2. To deploy the test Service. Service, you must add the following annotations, and type should also be specified as LoadBalancer, as follows:
    ```
    Kind: Service
    apiVersion: v1
    Metadata:
    Name: mylbapp
    Annotations:
         lb.kubesphere.io/v1alpha1: porter
    Spec:
         Selector:
             App: mylbapp
         Type: LoadBalancer
         Ports:
         - name: http
             Port: 8088
             targetPort: 80
    ```

You can use the sample service we provide.

### Sample Service (`service.yaml`)

`kubectl apply -f service.yaml`

3. Check whether there is a corresponding route on the router. If so, any host connected to this router should be able to access it via EIP+ServicePort.

    ```
    show routing
    ……
    10.11.11.11/32, ubest/mbest: 3/0
    *via 10.10.12.2, [20/0], 00:03:38, bgp-65001, external, tag 65000
    *via 10.10.12.3, [20/0], 00:03:38, bgp-65001, external, tag 65000
    *via 10.10.12.4, [20/0], 00:03:38, bgp-65001, external, tag 65000
    ```
