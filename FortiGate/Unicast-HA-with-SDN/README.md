# Unicast HA with SDN connector and client VM

## About

Azure template meant for support people to quickly and easily deploy a pair of FortiGates with (unicast) HA and preconfigured Azure SDN connector (with outside public IP and inside default route manipulation). The template also deploys a preconfigured Linux client VM on the private ("inside") network with default route through the master FortiGate.

This is **not official** Fortinet repository and the templates here should not be used to deploy any production systems. If you are looking for something official, go to https://github.com/fortinetsolutions/Azure-Templates .

## Network diagram

After deployment the network topology will look like this:

![Network diagram](https://raw.githubusercontent.com/ondrejholecek/azure-templates/master/FortiGate/Unicast-HA-with-SDN/diagram.png)

## Automatic configuration

### HA
  * FortiGate A is master unit of the cluster, with override enabled and priority 200.
  * FortiGate B is slave unit, with override enable but priority only 100.

### Interfaces
  * FortiGate interface "port1" is on the "outside" subnet (ie. facing the Internet), with "floating IP" ("FortiGate-shared-IP", always on master device).
  * FortiGate interface "port2" is on the "inside" subnet (ie. connecting internal devices). The Linux client is also connected to this network.
  * FortiGate interface "port3" is dedicated HA management interface with its own access to Internet ("management" subnet) and a public IP address attached to each device ("FortiGateA-mgmt-IP" and "FortiGateB-mgmt-IP").
  * FortiGate interface "port4" is HA heartbeat interface (with unicast HA configuration) on "ha" subnet.

### Routing
  * On the "inside" network, the routing table called "RoutingTable-inside" is created and default route is always going through the master FortiGate. The is also a couple of special routes configured over the assigned public IP ("LinuxClient-public-IP"), going directly to the Internet and bypassing the FortiGates (including "My IP address" that is specified as a deployment parameter). This is to allow independent management connectivity to the Client device.
  * Azure SDN connector is configured on both FortiGates to allow re-assigning the shared public IP address ("FortiGate-shared-IP") and changing the default route ("toDefault") in the inside routing table ("RoutingTable-inside").

## One click deployment

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fondrejholecek%2Fazure-templates%2Fmaster%2FFortiGate%2FUnicast-HA-with-SDN%2Fmain.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
