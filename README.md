# Deploy VMware ESXi 6.7 or 7.0 on Equinix Metal Servers
This Terraform script lets you deploy newer versions of VMware ESXi on Equinix Metal servers that only officially support VMware ESXi 6.5. This script deploys multiple servers with VMware ESXi 6.5 and then runs a shell script that updates the servers to VMware ESXi 6.7 or 7.0. To run the script, git clone this repo, install Terraform and set the variables declared in `variables.tf`, such as your Equinix Metal Project ID, amount of servers, ESXi update version etc. This can be done, for instance, by creating a file `terraform.tfvars` with the desired values in the repo's root, or by using any other method described in [Assigning Values to Root Module Variables](https://www.terraform.io/language/values/variables#assigning-values-to-root-module-variables). Then run the following commands in the script directory:

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

Prepare the server for the update and run the update. You can get the latest VMware ESXi update versions [here](https://esxi-patches.v-front.de/).

```
vim-cmd /hostsvc/maintenance_mode_enter

esxcli network firewall ruleset set -e true -r httpClient

# The update version can be retrieved from the website above
esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-7.0U3d-19482537-standard

esxcli network firewall ruleset set -e false -r httpClient

vim-cmd /hostsvc/maintenance_mode_exit

reboot
```

Once the update has completed, the server should now be running VMware ESXi 6.7 or ESXi 7.0!
