# Ansible SAP installation playbooks for HPE Morpheus Enterprise


## Description

This Ansible Collection executes various SAP Software installations and configuration tasks for various SAP solutions and deployment scenarios on supported Linux operating systems.
The roles and playbooks in this repository were tested for the use in HPE Morpheus Enterprise.
The roles and playbooks from https://github.com/sap-linuxlab/community.sap_install were used and slightly modified. Only tested roles and playbooks are part of this repository.

Included roles cover range of tasks:

- Installation of SAP HANA Database
- Installation of SAP Products, like SAP S4HANA, SAP BW4HANA and others. Only SAP S/4HANA and SAP BW/4HANA have been tested for integration into HPE Morpheus Enterprise.


## Requirements
### Control Nodes
Operating system:
Any operating system running HPE Morpheus Enterprise can be used.
SUSE Linux Enterprise Server for SAP applications 15 SP7 has been tested in HPE Morpheus Enterprise. Other versions could be used as well.

| Component | Control Node | Managed Node |
| --- | --- | --- |
| Operating System | Any OS | Red Hat Enterprise Linux for SAP Solutions 8.x, 9.x <br>SUSE Linux Enterprise Server for SAP applications 15 SP5, 15 SP6, 15 SP7 and 16.0 |
| Python | 3.11 or higher | 3.9 or higher |
| Ansible-Core | 2.18 or higher | N/A |
| Ansible | 12 or higher | N/A |

> **Managed Node Registration**<br>

> Operating system do not need to have access to required package repositories or subscription registration at provisioning time, as the VM template being used should contain all required packages.
 
**Additional notes:**

- **Version Compatibility:** For a detailed mapping of supported Python versions and Ansible-Core lifecycles, refer to the official [Ansible-Core Support Matrix](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix).
- **Control Node Permissions:** Ensure the user executing the playbooks has the necessary SSH keys and sudo privileges configured for the target environment.

## Installation Instructions

### Installation
It is recommended to create a fork of this GitHub repository and integrate that fork into HPE Morpheus Enterprise. This allows you to customize the playbooks for your environment if necessary.
The Morpheus_Import_Package.zip archive contains a Python script that imports Workflows and Layouts and their dependencies for automated SAP installations into HPE Morpheus Enterprise. The script also imports Tasks that execute the Ansible roles from this repository.

A detailed description of how to setup HPE Morpheus Enterprise and how to customize the imported Workflows and Layouts is available in [HPE Reference Architecture for SAP automation with HPE Morpheus Enterprise Software on Predefined Configuration and VMware virtualization](https://www.hpe.com/psnow/doc/a50014106enw)

## Use Cases

### Example Scenarios
- Installation of SAP HANA
- Installation of SAP S4HANA or other SAP products, including SAP system copy

More deployment scenarios are available in [community.sap_install](https://github.com/sap-linuxlab/community.sap_install) repository. However, the integration was not tested with HPE Morpheus Enterprise.

### Ansible Roles
All included roles can be executed independently.

| Name | Summary |
| :--- | :--- |
| [sap_hana_install](https://github.com/HewlettPackard/morpheus-sap-install/blob/main/roles/sap_hana_install/) | Install SAP HANA via HDBLCM |
| [sap_swpm](https://github.com/HewlettPackard/morpheus-sap-install/blob/main/roles/sap_swpm) | Install SAP Software via SWPM |

## Testing
This Ansible Collection was tested across different Operating Systems versions, SAP products and scenarios. You can find examples of some of them below.

Operating systems:
- Red Hat Enterprise Linux for SAP Solutions 8.x 9.x (RHEL4SAP)
- SUSE Linux Enterprise for SAP 15.x

SAP Products:
- SAP S/4HANA AnyPremise (1809, 1909, 2020, 2021, 2022, 2023, 2025) with setup as Standard, Distributed or Restore System Copy
- SAP BW/4HANA (2021, 2023) with setup as Standard, Distributed or Restore System Copy
- SAP HANA 2.0 (SPS04+) with setup as Scale-Up

**NOTE: It is not possible to test every Operating System and SAP Product combination with every release. Testing is regularly done for common scenarios: SAP HANA, SAP S4HANA installation and system copy

## Contributing to the sap-linuxlab community playbooks
You can find more information about ways you can contribute at [sap-linuxlab website](https://sap-linuxlab.github.io/initiative_contributions/).

## Support
You can report any issues with this set of playbooks and roles using the [Issues] section.


## Further Information

### Variable Precedence Rules
Please follow [Ansible Precedence guidelines](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable) on how to pass variables when using this collection.

### Getting Started
More information on how to execute Ansible playbooks is in [Getting started guide](https://github.com/HewlettPackard/morpheus-sap-install/blob/main/docs/getting_started/README.md).


## License
[Apache 2.0](https://github.com/HewlettPackard/morpheus-sap-install/blob/main/LICENSE) 
