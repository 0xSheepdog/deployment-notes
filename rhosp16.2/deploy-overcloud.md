# Overcloud Deploy

### Register nodes for Overcloud
https://docs.redhat.com/en/documentation/red_hat_openstack_platform/16.2/html/director_installation_and_usage/assembly_configuring-a-basic-overcloud#proc_registering-nodes-for-the-overcloud_basic

```
openstack overcloud node import --validate-only ~/nodes.yaml

openstack overcloud node import ~/nodes.yaml

openstack baremetal node list --fit
```

---

### Introspection and registration
#### Introspect and classify a specific node(s) from 'manageable' to "available"
```
openstack overcloud node introspect --provide <node1> [node2] [noden]
```

#### Introspect and classify ALL 'manageable' nodes to "available"
```
openstack overcloud node introspect --all-manageable --provide
```

#### Classify ALL 'manageable' nodes to "available" WITHOUT INTROSPECTION
#### DON'T DO THIS UNLESS YOU KNOW WHAT YOU ARE DOING
```
openstack overcloud node provide --all-manageable
```

#### Monitor ironic-introspection of baremetal nodes
```
sudo tail -f /var/log/containers/ironic-inspector/ironic-inspector.log
```

###### (or with my preferred filters)
```
sudo tail -f /var/log/containers/ironic-inspector/ironic-inspector.log | grep -v -e DEBUG -e "OPTIONS / HTTP/1.0"
```

### Troubleshooting registration/introspection
#### Stop/interrupt the introspection process
```
openstack baremetal introspection abort [NODE UUID]
```

#### Take out of maintenance mode
```
openstack baremetal node maintenance unset [NODE UUID]
```

#### Recycle the ironic introspection services
```
sudo systemctl restart tripleo_ironic_inspector_dnsmasq.service tripleo_ironic_inspector.service
```

#### Dump introspection data about a node
```
openstack baremetal introspection data save <node> --file filename.txt
```

#### Clean a node 
(wipe metadata "erase_devices_metadata" or alldisks "erase_devices")
```
# set to "manageable"
openstack baremetal node manage <node>

# clean node
openstack baremetal node clean <node> \
  --clean-steps '[{"interface": "deploy", "step": "erase_devices"}]'
```

---

### Set configurations for Overcloud nodes

#### UEFI boot mode
https://docs.redhat.com/en/documentation/red_hat_openstack_platform/16.2/html/director_installation_and_usage/assembly_configuring-a-basic-overcloud#proc_setting-the-boot-mode-to-uefi-mode_basic

#### Specifying node IDs
https://docs.redhat.com/en/documentation/red_hat_openstack_platform/16.2/html-single/advanced_overcloud_customization/index#proc_assigning-specific-node-ids_controlling-node-placement

##### Configuring the Node ID and UEFI boot mode
1. Get the current capabilties of each node:
```
openstack baremetal node show <node> -f json -c properties | jq -r .properties.capabilities
```
2. Add the node ID to the capabilities list (remove any `profile:xxx`
capability. Also add the `boot_mode:uefi` if needed.
```
openstack baremetal node set --property capabilities="node:controller-<#>,boot_mode:uefi,boot_option:local,...,capability...,capability..." <node>
```
3. Create a heat environment file `scheduler_hints_env.yaml` to match the  
capabilities for each node and sets the baremetal flavor:
```
parameter_defaults:
  OvercloudControllerFlavor: baremetal
  OvercloudComputeFlavor: baremetal

parameter_defaults:
  ControllerSchedulerHints:
    'capabilities:node': 'controller-%index%'
  ComputeSchedulerHints:
    'capabilities:node': 'compute-%index%'
```

#### 



### Configure overcloud networking

#### Network Isolation
https://docs.redhat.com/en/documentation/red_hat_openstack_platform/16.2/html-single/advanced_overcloud_customization/index#assembly_basic-network-isolation

cp /usr/share/openstack-tripleo-heat-templates/network_data.yaml /home/stack/templates/.
(Modify to suit your usecase)


#### Custom network interface templates
https://docs.redhat.com/en/documentation/red_hat_openstack_platform/16.2/html-single/advanced_overcloud_customization/index#assembly_custom-network-interface-templates

Read the sections carefully to understand how to modify the template and ensure they are properly invoked.

```
cp -r ~/openstack-tripleo-heat-templates-rendered/network ~/templates/
```

Edit the appropriate custom-nics files with the right interface names, bond options, etc.
