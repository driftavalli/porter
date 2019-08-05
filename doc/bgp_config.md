# Introduction to BGP Config
Porter uses gobgp to exchange routing information with external routers. Starting BGP requires some configuration information (examples are in config/bgp). Currently, there are not many parameters. The following briefly describes how to configure the BGP server used by the plug-in.

## Sample configuration file
```
[global.config]
    as = 65000
    router-id = "192.168.98.111"
    port = 17900
[porter-config]
    using-port-forward =true
[[neighbors]]
    [neighbors.config]
        neighbor-address = "192.168.98.5"
        peer-as = 65001
    [neighbors.add-paths.config]
        send-max = 8
```
The configuration file is a JSON file. JSON files have multiple expressions. Toml, yaml, and json are three common forms. Gobgp defaults to toml, and can be converted as needed. Modify the porter's process parameter -t to specify the format of the configuration file. Such as:
```
 - args:
        - --metrics-addr=127.0.0.1:8080
        - -f
        - /etc/config/config.yaml
        - -t
        - yaml
   command:
        - /manager
```

## Global configuration

`Modify global.config to modify global parameters`


1. AS is the autonomous domain in which the cluster resides. It must be different from the autonomous domain of the connected router. The same reason is that the routes cannot be transmitted correctly. The specific reasons are different between EBGP and IBGP.
2. Router-id indicates the id of the cluster. Generally, the ip of the primary NIC of the k8s primary node is taken.
3. Port is the port that gobgp listens on. The default is 179. Since calico also uses BGP and occupies port 179, another port must be specified here. If the router in the cluster does not support ports other than 179, you need to enable port forwarding on the node where the port is located and map 179 to a non-standard port.

## Port configuration

`Port-related configuration`

1. Using-port-forward enables port forwarding for switches that do not support ports other than 179, such as Cisco switches.

## Set up neighbors
`The neighbor is the router where the cluster is located. You can add multiple neighbors, and in most cases you only need to configure one.`

1. Neighbor-address is the IP address of the router.
2. The peer-as is the autonomous domain of the neighbor. It must be the same as the one configured in the router. If it is a private network, it generally uses more than 65,000 autonomous domains.
3. Send-max specifies the upper limit of the sending route. If ECMP is to be implemented, this value must be greater than 1.

The porter only uses a small part of the function in gobgp. If there are more requirements, please refer to the configuration file description of gobgp https://github.com/osrg/gobgp/blob/master/docs/sources/configuration.md
