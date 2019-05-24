# Unicast HA with SDN connector and client VM

## About

Azure template meant for support people to quickly and easily deploy a pair of FortiGates with (unicast) HA and preconfigured Azure SDN connector (with outside public IP and inside default route manipulation). The template also deploys a preconfigured Linux client VM on the private ("inside") network with default route through the master FortiGate.

This is **not official** Fortinet repository and the templates here should not be used to deploy any production systems. If you are looking for something official, go to https://github.com/fortinetsolutions/Azure-Templates .

## Network diagram

After deployment the network topology will look like this:

![Example Diagram](https://raw.githubusercontent.com/ondrejholecek/azure-templates/master/FortiGate/Unicast-HA-with-SDN/azure_github_ha.png)

## One click deployment

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fondrejholecek%2Fazure-templates%2Fmaster%2FFortiGate%2FUnicast-HA-with-SDN%2Fmain.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
