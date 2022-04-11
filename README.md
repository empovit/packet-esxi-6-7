# Deploy VMware ESXi 6.7 on Equinix Metal Servers
This Terraform script lets you deploy VMware ESXi 6.7 on Equinix Metal servers. As of now, only VMware ESXi 6.5 is officially supported on Equinix Metal. This script deploys multiple servers with VMware ESXi 6.5 and then runs a shell script that updates the servers to VMware ESXi 6.7. To run the script, git clone this repo, install Terraform and configure the `variables.tf` file with your Equinix Metal Project ID, amount of servers, ESXi update version etc. Then run the following commands in the script directory:

```
terraform init
terraform apply
```

If you would like to perform the update manually, you need to deploy an ESXi 6.5 server and run the following commands through an SSH session in the server once it has been deployed.

First make sure that SSH and Shell are enabled, they should be enabled by default on Equinix Metal but if not, run the following:

```
vim-cmd hostsvc/enable_ssh
vim-cmd hostsvc/start_ssh
```

Swap must be enabled on the datastore. Otherwise, the update may fail with a "no space left" error.
```
esxcli sched swap system set --datastore-enabled true
esxcli sched swap system set --datastore-name datastore1
```

Prepare the server for the update and run the update. You can get the latest VMware ESXi update versions [here](https://esxi-patches.v-front.de/ESXi-6.7.0.html).

```
vim-cmd /hostsvc/maintenance_mode_enter

esxcli network firewall ruleset set -e true -r httpClient

# The update version can be retrieved from the website above
esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-6.7.0-20191204001-standard

esxcli network firewall ruleset set -e false -r httpClient

vim-cmd /hostsvc/maintenance_mode_exit

reboot
```

Once the update has completed, the server should now be running VMware ESXi 6.7!
