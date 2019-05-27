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

## Deployment parameters

When you deploy this template, Azure will ask for following parameters:

![DeployBYOL](https://raw.githubusercontent.com/ondrejholecek/azure-templates/master/FortiGate/Unicast-HA-with-SDN/deploy-BYOL.png)

  * License type - Select "Bring Your Own License" if you have two FortiGate licenses **supporting at least 4 cores**, or "Pay As You Go" which needs no license, but additional fee is payed for the FortiOS.
  * Fortios version - Choose a version to be installed. Be aware that not all GA versions exist in Azure. At this point (May 27 2019) following versions can be used: "5.6.4", "5.6.5", "5.6.6", "6.0.0", "6.0.2", "6.0.3", "6.0.4", "6.2.0". If you don't know what to use, keep "latest" which will install the most recent one.
  * Fortigate name - Choose a VM name for the first and second FortiGates. Names of some other objects will be also based on it.
  * Fortigate license - See the "Licensing" section bellow.
  * Linux Client name - Choose a VM name for the Linux client machine. Names of some other objects will be also based on it.
  * Virtual Network name - Choose a name of the VNET object that this deploying will create.
  * Admin username and password - credentials used on both FortiGates as well as on the Linux client.
  * HA Application ID and secret - See the "High availability" section bellow.
  * My IP address - Route to this address will be configured in the "inside" routing table so you can access the Linux client VM directly, bypassing FortiGates.

## VM size

For FortiGates there are at least four interfaces needed, which means that the smallest and cheapest device type is ``Standard_F4`` and this one is used in this template.

For Linux VM the smallest and cheapest usable device device is ``Standard_B1ls``.

The VM size is not configurable neither for FortiGate nor for the Linux Client VM.

## Licensing

If you select "Pay As You Go" license type during deployment, you leave the "Fortigate A License" and "Fortigate B License" parameters empty, because there is no need for the license for PAYG deployment. However, additional fees will be payed for each FortiGate until they are deleted.

It is better to use your own licenses by selecting "Bring Your Own License" type. In that case you need to provide two liceses each supporting at least 4 CPU cores (because we need 4 interfaces for each FortiGate). The license file needs to be base64-encoded to one line and pasted to the right deployment parameter ("Fortigate A License" or "Fortigate B License"). You can use any tool that can convert to base64 (like https://www.base64encode.org/), but be careful to copy *the whole encoded string* - sometimes some of the last characters are not selected after simple double-click on the string.

Make sure you provide the licences and that you provide them correctly in the deployment stage when using BYOL. **Uploading the license in the later stage** is possible, but **will erase quite a lot of the automatically prepared configuration** and you will need to re-configure things manually. 

## High availability

During the deployment, the HA and Azure SDN setting are auto-configured. This is done mostly automatically but there are two settings that cannot be done without user's cooperation:

  * HA Application ID - This is the ID of the Azure application that can make changes to routing table and (dis)associate public IP addresses. It can be found in "Azure Active Directory" >> "App registrations" >> [select application] >> copy "Application (client) ID".
  * HA Application Secret - This is the password generated by Azure for the application. It can be created on the same application page in "Certificates & secrets" menu, "Client secrets" section. Be aware that the secret is shown only once when it is created, so save it somewhere safe.
  
The Application role in Azure can "Contributor", but such rights are unnecessary wide. Following is the minimal usable role specification (JSON file usable with ``az role definition create --role-definition role.json`` AzureCLI command):

```
{
   "Name": "FortiGate HA management",
   "IsCustom": true,
   "Description": "Allows moving public IP and manipulating routing table when FortiGate HA fails over.",
   "Actions": [
       "Microsoft.Network/publicIPAddresses/read",
       "Microsoft.Network/publicIPAddresses/join/action",
       "Microsoft.Network/networkInterfaces/read",
       "Microsoft.Network/networkInterfaces/write",
       "Microsoft.Network/virtualNetworks/subnets/join/action",
       "Microsoft.Network/routeTables/read",
       "Microsoft.Network/routeTables/write"
   ],
   "NotActions": [],
   "DataActions": [],
   "NotDataActions": [],
   "AssignableScopes": [
      "/subscriptions/{fill_your_subscription_id}/"
   ]
}
```

## Access to devices after deployment

When the deployment is finished, you can switch to "Outputs" section (still on the deployment page) to see the IP addresses assigned to each device. There are also easily usable HTTPs and SSH links for your convenience.

![Outputs](https://raw.githubusercontent.com/ondrejholecek/azure-templates/master/FortiGate/Unicast-HA-with-SDN/outputs.png)


## One click deployment

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fondrejholecek%2Fazure-templates%2Fmaster%2FFortiGate%2FUnicast-HA-with-SDN%2Fmain.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
