# Ansible for NSX-T

## Overview
This repository contains NSX-T Ansible Modules, which one can use with
Ansible to work with [VMware NSX-T Data Center][vmware-nsxt].

[vmware-nsxt]: https://www.vmware.com/products/nsx.html

For general information about Ansible, visit the [GitHub project page][an-github].

[an-github]: https://github.com/ansible/ansible

These modules are maintained by [VMware](https://www.vmware.com/).

Documentation on the NSX platform can be found at the [NSX-T Documentation page](https://docs.vmware.com/en/VMware-NSX-T/index.html)

## NSX Compatibility

The following versions of NSX are supported:

 * NSX-t 3.0
 * NSX-T 2.5.1
 * NSX-T 2.5
 * NSX-T 2.4

## Prerequisites
We assume that ansible is already installed.
These modules support ansible version 2.8.3 and onwards.

* Python3 >= 3.6.8
* PyVmOmi - Python library for vCenter api.
* OVF Tools - Ovftool is used for ovf deployment.

### Supported NSX Objects/Workflows
The modules in this repository are focused on enabling automation of installation workflows of NSX-T. We have modules that support the legacy MP and new Policy API.

#### MP API
MP API modules can be used to configure an NSX resource with one-to-one mapping.

##### Deployment and installation modules

* nsxt_deploy_ova
* nsxt_licenses
* nsxt_manager_status
* nsxt_licenses_facts
* nsxt_edge_clusters
* nsxt_edge_clusters_facts
* nsxt_fabric_compute_managers
* nsxt_fabric_compute_managers_facts
* nsxt_fabric_nodes
* nsxt_fabric_nodes_facts
* nsxt_compute_collection_fabric_templates
* nsxt_compute_collection_fabric_templates_facts
* nsxt_ip_pools
* nsxt_ip_pools_facts
* nsxt_uplink_profiles
* nsxt_uplink_profiles_facts
* nsxt_transport_zones
* nsxt_transport_zones_facts
* nsxt_transport_nodes
* nsxt_transport_nodes_facts
* nsxt_transport_node_collections
* nsxt_transport_node_collections_facts
* nsxt_transport_node_profiles
* nsxt_transport_node_profiles_facts
* nsxt_compute_collection_transport_templates
* nsxt_compute_collection_transport_templates_facts
* nsxt_controller_manager_auto_deployment

###### Logical networking modules
* nsxt_logical_ports
* nsxt_logical_ports_facts
* nsxt_logical_routers
* nsxt_logical_routers_facts
* nsxt_logical_router_ports
* nsxt_logical_router_ports_facts
* nsxt_logical_router_static_routes
* nsxt_logical_switches
* nsxt_logical_switches_facts
* nsxt_ip_blocks
* nsxt_ip_blocks_facts

#### Policy API
Policy API modules are aggregated such that logical constructs related to an NSX resource can be configured using a single playbook. They can be identified with prefix *nsxt_policy_*. The below list outlines the supported modules and the resources that can be configured through a module.

Note that the Policy modules are supported only for NSX-T 3.0 and above.

1. Tier-0 Gateway (nsxt_policy_tier0)
   1. Tier-0 Locale Services
   2. Tier-0 Static Routes
   3. Tier-0 Interface
   4. Tier-0 BGP
   5. Tier-0 BGP Neighbors
2. Tier-1 Gateway (nsxt_policy_tier1)
   1. Tier-1 Locale Services
   2. Tier-1 Static Routes
   3. Tier-1 Interface
3. Segment (nsxt_policy_segment)
   1. Segment Port
4. Policy Group (nsxt_policy_group)
5. Security Policy and Firewall rules (nsxt_policy_security_policy)
6. IP Pools (nsxt_policy_ip_pool)
   1. IP Address Pool Block Subnet
   2. IP Address Pool Static Subnet
7. IP Blocks (nsxt_policy_ip_block)

Note that to add a new modules in Policy API, it's base class name should be added in the BASE_RESOURCES in module_utils/nsxt_base_resource.py

## Build & Run

### Install PyVmOmi
```
pip install --upgrade pyvmomi pyvim requests ssl
```
### Download and Install Ovf tool 4.3 - [Ovftool](https://my.vmware.com/web/vmware/details?downloadGroup=OVFTOOL430&productId=742)
(Note: Using ovftool version 4.0/4.1 causes OVA/OVF deployment failure with Error: cURL error: SSL connect error\nCompleted with errors\n)

### Download [ansible-for-nsxt](https://github.com/vmware/ansible-for-nsxt/archive/v3.0.0.zip).
```
unzip ansible-for-nsxt-3.0.0.zip
cd ansible-for-nsxt-3.0.0
```
To run a sample Ansible playbook - To create a sample test topology using deployments and install module.

Edit test_basic_topology.yml and answerfile.yml to match values to your environment.
```
ansible-playbook test_basic_topology.yml -vvv
```

### Authentication

#### Using MP API
Ansible-for-nsxt supports two types of authentication using MP API.
1. Basic server authentication
2. Certificate based authentication

##### Basic server authentication
In basic server authentication, client has to explicitly provide NSX username and password for the NSX manager. The credentials have to be listed in ansible-playbook.

##### Certificate based authentication
In certificate based authentication, client has to register their certificates to NSX manager using nsxt_certificates task. After registering the certificates, client has to create its own principal identity on NSX manager using nsxt_principal_identities taks.
The process of certificate registration and creation of principal identity has to be done using basic server authentication. Use test_certificates.yml and test_principal_identities.yml to match the values according to the client's environment.
```
ansible-playbook test_certificates.yml -vvv
ansible-playbook test_principal_identities -vvv
```
The path of the .p12 file i.e the file containing public and private key has to be set to an environment variable named NSX_MANAGER_CERT_PATH. 
**Note:** Make sure NSX_MANAGER_CERT_PATH is set in the same remote host, where modules would be executed. 

###### Generating certificates?
Following commands can be used in order to generate certificates.
```
openssl req -newkey rsa:2048 -extensions usr_cert -nodes -keyout nsx_certificate.key -x509 -days 365 -out nsx_certificate.crt -subj "/C=US/ST=California/L=PaloAlto/O=VMware/CN=certauth-test" -sha256

openssl pkcs12 -export -out nsx_certificate.pfx -inkey nsx_certificate.key -in nsx_certificate.crt

openssl pkcs12 -in nsx_certificate.pfx -out nsx_certificate.p12 -nodes
```

The nsx_certificate.crt file generated as output from the above command contains the public key certificate.
the file nsx_certificate.p12 file contains the public and private key generated. The path of nsx_certificate.p12 file has to be set in the environment variable NSX_MANAGER_CERT_PATH.

Note: usr_cert tells OpenSSL to generate a client certificate. This must be defined in openssl.cnf.

#### Validate CA in MP API

To validate ceritificate authority (CA), set NSX_MANAGER_CA_PATH environment variable on Ansible control node pointing to CA certificate of NSX manager and pass validate_certs as ``True`` in ansible playbook.

#### Using Policy API
All the Policy API based Ansible Modules provide the following authentication mechanisms:

##### Basic Authentication
This is the same as in MP API. It can be used by specifying the following fields in the playbook:
1. **username**: The username to authenticate with the NSX manager
2. **password**: The password to authenticate with the NSX manager

For example:
```yaml
- hosts: localhost
  tasks:
    - name: Update Tier0
      nsxt_policy_tier0:
        hostname: "default"
        username: admin
        password: my-password
        validate_certs: False
        display_name: test-tier0-1
        state: present
```

# Unit Testing
To test the Ansible modules or see examples of playbooks, please put the respective playbook from unit_tests folder to the base folder and run the ansible-playbook.

Please note that you must specify the correct vmware args in order to successfully update the resources.

# Contributing

The ansible-for-nsxt project team welcomes contributions from the community. Before you start working with ansible-for-nsxt, please read our [Developer Certificate of Origin](https://cla.vmware.com/dco). All contributions to this repository must be signed as described on that page. Your signature certifies that you wrote the patch or have the right to pass it on as an open-source patch. For more detailed information, refer to [CONTRIBUTING.md](CONTRIBUTING.md).

# Support

The NSX-T Ansible modules in this repository are community supported. For bugs and feature requests please open a Github Issue and label it appropriately. As this is a community supported solution there is no SLA for resolutions.

# License
Copyright (c) 2018 VMware, Inc.  All rights reserved

The NSX-T Ansible modules in this repository are available under [BSD-2 license or GPLv3](LICENSE.txt) applies to all parts of the ansible-for-nsxt.
You may not use them except in compliance with the License.
