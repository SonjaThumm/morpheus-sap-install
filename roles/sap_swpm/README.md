<!-- BEGIN Title -->
# sap_swpm Ansible Role
<!-- END Title -->
![Ansible Lint for sap_swpm](https://github.com/sap-linuxlab/community.sap_install/actions/workflows/ansible-lint-sap_swpm.yml/badge.svg)

## Description
<!-- BEGIN Description -->
The Ansible role `sap_swpm` installs various SAP Systems installable by SAP Software Provisioning Manager (SWPM).
<!-- END Description -->

<!-- BEGIN Dependencies -->
<!-- END Dependencies -->

## Prerequisites
<!-- BEGIN Prerequisites -->
Managed nodes:
- Directory with SAP Installation media is present and `sap_swpm_software_path` updated.
- Ensure that servers are configured for SAP Systems. 
- Ensure that volumes and filesystems are configured correctly.

### Prepare SAP installation media

Place a valid SWPM SAR file in a directory specified by variable `sap_swpm_swpm_path` (e.g. /software/sap_swpm). Example: `SWPM20SP18_3-80003424.SAR`

Place the following files in a directory specified by variable `sap_swpm_software_path` (e.g. /software/sap_swpm_download_basket):

  - For a new installation
      - Download the appropriate software from SAP Software Download Center, Maintenance Planner, etc.
  - For a restore or new installation
      - SAP IGS                   - `igs*.sar`
      - SAP IGS HELPER            - `igshelper*sar`
      - SAP Host Agent            - `SAPHOSTAGENT*SAR`
      - SAP Kernel DB             - `SAPEXEDB_*SAR`
      - SAP Kernel DB Independent - `SAPEXE_*SAR`
      - SAP HANA Client           - `IMDB_CLIENT*SAR`


Set the right values for the directories in the Options List of HPE Morpheus Enterprise.

<!-- END Prerequisites -->

## Execution
<!-- BEGIN Execution -->
<!-- END Execution -->

<!-- BEGIN Execution Recommended -->
### Recommended
Note: For most scenarios, a database like SAP HANA must be available. Use the role [sap_hana_install](https://github.com/HewlettPackard/morpheus-sap-install/tree/main/roles/sap_hana_install) for installing the SAP HANA database.
<!-- END Execution Recommended -->

### Execution Flow
<!-- BEGIN Execution Flow -->

### Pre-Install

- Determine installation type
    - The product id specified determines the installation type
        - standard installation
        - system restore

- Get SAPCAR executable filename from `sap_swpm_sapcar_path`

- Get SWPM executable filename from `sap_swpm_swpm_path`

- Get all .SAR filenames  from `sap_swpm_software_path`

- (Optional) Update `/etc/hosts` if `sap_swpm_update_etchosts` is set to `true` (Default: `false`).

- (Optional) Do not disable password expiry if `sap_swpm_set_sidadm_noexpire` is set to `false` (Default: `true`).

- (Optional) Apply firewall rules for SAP HANA if `sap_swpm_setup_firewall` is set to `true` (Default: `false`).

- At this stage, the role is searching for a sapinst inifile on the managed node, or it will create one:

    - If a file `inifile.params` is located on the managed node in the directory specified in `sap_swpm_inifile_directory`,
      the role will not create a new one but rather download this file to the control node.

    - If such a file does *not* exist, the role will create an SAP SWPM `inifile.params` file by one of the following methods:<br>

        - It is also possible to use method 1 for creating the inifile and then replace or set additional variables using method 2:<br>
          Define both of the related parameters, `sap_swpm_inifile_sections_list` and `sap_swpm_inifile_parameters_dict`.

        - Method 1: Predefined sections of the file `inifile_params.j2` will be used to create the file `inifile.params`.<br>
          The variable `sap_swpm_inifile_sections_list` contains a list of sections which will part of the file `inifile.params`.<br>
          All other sections will be ignored. The inifile parameters themselves will be set according to other role parameters.<br>
          Example: The inifile parameter `archives.downloadBasket` will be set to the content of the role parameter `sap_swpm_software_path`.

        - Method 2: The file `inifile.params` will be configured from the content of the dictionary `sap_swpm_inifile_parameters_dict`.<br>
          This dictionary is defined like in the following example:<br>
```yaml
sap_swpm_inifile_parameters_dict:
  archives.downloadBasket: /software/download_basket
  NW_getFQDN.FQDN: example.com
```

- The file `inifile.params` is then transferred to a temporary directory on the managed node, to be used by the sapinst process.

- Check for a previous installation run and discard the existing logfile directory if a previous installation failed before the end of the parameter input phase. Create the installation log directory (customizable via sap_swpm_sapinst_instdir, assigned to group sapinst with optional group ID set via sap_swpm_sapinst_gid) to enable continuation of a failed installation after the input phase.

### SAP SWPM

- Execute SWPM. This and the remaining steps can be skipped by setting the default of the parameter `sap_swpm_run_sapinst` to `false`.

### Post-Install

- Set expiry of Linux created users to 'never'

- (Optional) Apply firewall rules for SAP Netweaver if `sap_swpm_setup_firewall` is set to `true` (Default: `false`).

- (Optional) Handle the execution of SUM if SWPM started it (See Up-To-Date Installation below).
<!-- END Execution Flow -->

### Example
<!-- BEGIN Execution Example -->

#### Playbook for installing a SAP ABAP PAS in HPE Morpheus Enterprise
Example shows the command line used for the task in HPE Morpheus Enterprise for the PAS installation. Input parameter are defined in HPE Morpheus Enterprise

--user root --extra-vars '{"ansible_python_interpreter":"<%= customOptions.ansible_python_interpreter %>","sap_swpm_parallel_jobs_nr":"<%= customOptions.sap_swpm_parallel_jobs_nr %>","sap_swpm_master_password":"<%= customOptions.sap_swpm_master_password %>","sap_swpm_software_path":"<%= results.getSAPSWPMinput.sap_swpm_software_path %>","sap_swpm_product_catalog_id":"<%= results.getSAPSWPMinput.sap_swpm_product_catalog_id %>","sap_swpm_sid":"<%= customOptions.sap_swpm_sid %>","sap_swpm_ascs_instance_nr":"<%= customOptions.sap_swpm_ascs_instance_nr %>","sap_swpm_pas_instance_nr":"<%= customOptions.sap_swpm_pas_instance_nr %>","sap_swpm_sapinst_path":"<%= results.getSAPSWPMinput.sap_swpm_sapinst_path %>","sap_swpm_db_sid":"<%= results.getSAPHANAparameter.sap_swpm_db_sid %>","sap_swpm_db_instance_nr":"<%= results.getSAPHANAparameter.sap_swpm_db_instance_nr %>","sap_swpm_db_host":"<%= results.getSAPHANAparameter.sap_swpm_db_host %>","sap_swpm_db_system_password":"<%= customOptions.sap_hana_install_master_password %>"}'
<!-- END Execution Example -->

<!-- BEGIN Role Tags -->
### Role Tags
With the following tags, the role can be called to perform certain activities only:

- tag `sap_swpm_generate_inifile`: Only create the sapinst inifile, without running most of the preinstall steps.
  This can be useful for checking if the inifile is created as desired.
- tag `sap_swpm_sapinst_commandline`: Only show the sapinst command line.
- tag `sap_swpm_pre_install`: Perform all preinstallation steps, then exit.
- tag `sap_swpm_setup_firewall`: Only perform the firewall preinstallation settings (but only if variable `sap_swpm_setup_firewall` is set to `true`).
- tag `sap_swpm_update_etchosts`: Only update file `/etc/hosts` (but only if variable `sap_swpm_update_etchosts` is set to `true`).
<!-- END Role Tags -->

## License
<!-- BEGIN License -->
Apache 2.0
<!-- END License -->

## Maintainers
<!-- BEGIN Maintainers -->
- [Bernd Finger](https://github.com/berndfinger)
- [Sean Freeman](https://github.com/seanfreeman)
<!-- END Maintainers -->

## Role Variables
<!-- BEGIN Role Variables -->
**NOTE: Discontinued variables:**

- `sap_swpm_ansible_role_mode`

### Variables for creating sapinst inifile

#### sap_swpm_run_sapinst
- _Type:_ `bool`
- _Default:_ `true`

Set to `false` to disable execution of sapinst after creation of inifile.


###  Variables for controlling contents of inifile

#### sap_swpm_inifile_sections_list
- _Type:_ `list`
- _Default:_
```yaml
sap_swpm_inifile_sections_list:
  - swpm_installation_media
  - swpm_installation_media_swpm2_hana
  - credentials
  - credentials_hana
  - db_config_hana
  - db_connection_nw_hana
  - db_restore_hana
  - nw_config_other
  - nw_config_central_services_abap
  - nw_config_primary_application_server_instance
  - nw_config_ports
  - nw_config_host_agent
  - sap_os_linux_user
```

Define list of sections that will be used to control parameters added into sapinst inifile.
Available values:
```yaml
sap_swpm_inifile_sections_list:
  - swpm_installation_media
  - swpm_installation_media_swpm2_hana
  - swpm_installation_media_swpm1
  - swpm_installation_media_swpm1_exportfiles
  - swpm_installation_media_swpm1_ibmdb2
  - swpm_installation_media_swpm1_oracledb_121
  - swpm_installation_media_swpm1_oracledb_122
  - swpm_installation_media_swpm1_oracledb_19
  - swpm_installation_media_swpm1_sapase
  - swpm_installation_media_swpm1_sapmaxdb
  - maintenance_plan_stack_tms_config
  - maintenance_plan_stack_tms_transports
  - maintenance_plan_stack_spam_config
  - maintenance_plan_stack_sum_config
  - maintenance_plan_stack_sum_10_batch_mode
  - credentials
  - credentials_hana
  - credentials_anydb_ibmdb2
  - credentials_anydb_oracledb
  - credentials_anydb_sapase
  - credentials_anydb_sapmaxdb
  - credentials_nwas_ssfs
  - credentials_hdbuserstore
  - db_config_hana
  - db_config_anydb_all
  - db_config_anydb_ibmdb2
  - db_config_anydb_oracledb
  - db_config_anydb_oracledb_121
  - db_config_anydb_oracledb_122
  - db_config_anydb_oracledb_19
  - db_config_anydb_sapase
  - db_config_anydb_sapmaxdb
  - db_connection_nw_hana
  - db_connection_nw_anydb_ibmdb2
  - db_connection_nw_anydb_oracledb
  - db_connection_nw_anydb_sapase
  - db_restore_hana
  - nw_config_anydb
  - nw_config_other
  - nw_config_central_services_abap
  - nw_config_central_services_java
  - nw_config_primary_application_server_instance
  - nw_config_additional_application_server_instance
  - nw_config_ers
  - nw_config_ports
  - nw_config_java_ume
  - nw_config_java_feature_template_ids
  - nw_config_java_icm_credentials
  - nw_config_webdisp_generic
  - nw_config_webdisp_gateway
  - nw_config_host_agent
  - nw_config_post_load_abap_reports
  - nw_config_livecache
  - nw_config_sld
  - nw_config_abap_language_packages
  - sap_os_linux_user
```


### Variables to define software paths

#### sap_swpm_sapcar_path
- _Type:_ `string`

Define path to directory with SAPCAR file.

#### sap_swpm_sapcar_file_name
- _Type:_ `string`

(Optional) Define name of SAPCAR file, or leave for auto-detection.

#### sap_swpm_swpm_path
- _Type:_ `string`

Define path to directory with SWPM.

#### sap_swpm_swpm_sar_file_name
- _Type:_ `string`

(Optional) Define name of SWPM file, or leave for auto-detection.

#### sap_swpm_software_extract_directory
- _Type:_ `string`

(Optional) Define path to directory with unpacked SWPM file.

#### sap_swpm_software_use_media
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to use SAP Media files instead of SAR files.</br>
- SWPM2 (New SAP products like S4H, BW4H) uses SAR files.</br>
- SWPM1 (Older SAP products) use CD Media files.

#### sap_swpm_inifile_directory
- _Type:_ `string`

Define directory where sapinst inifile will be stored.

#### sap_swpm_install_saphostagent
- _Type:_ `bool`
- _Default:_ `true`

Set to `false` to disable installation of SAP Hostagent. **Not recommended**


### Variables specific to SAP Netweaver

#### sap_swpm_product_catalog_id
- _Type:_ `string`

Define SWPM Product catalog ID for installation. Example for SAP ASCS Netweaver: `NW_ABAP_ASCS:NW750.HDB.ABAPHA`.

#### sap_swpm_sid
- _Type:_ `string`

Define SAP System ID (SID) for installation.

#### sap_swpm_ascs_instance_nr
- _Type:_ `string`

Define SAP Netweaver ABAP Central Services (ASCS) instance number.

#### sap_swpm_ascs_instance_hostname
- _Type:_ `string`

Define SAP Netweaver ABAP Central Services (ASCS) hostname.

#### sap_swpm_ers_instance_nr
- _Type:_ `string`

Define SAP Netweaver Enqueue Replication Server (ERS) instance number.

#### sap_swpm_ers_instance_hostname
- _Type:_ `string`

Define SAP Netweaver Enqueue Replication Server (ERS) hostname.

#### sap_swpm_pas_instance_nr
- _Type:_ `string`

Define SAP Netweaver Primary Application Server (PAS) instance number.

#### sap_swpm_pas_instance_hostname
- _Type:_ `string`

Define SAP Netweaver Primary Application Server (PAS) hostname.

#### sap_swpm_aas_instance_nr
- _Type:_ `string`

Define SAP Netweaver Additional Application Server (AAS) instance number.

#### sap_swpm_aas_instance_hostname
- _Type:_ `string`

Define SAP Netweaver Additional Application Server (AAS) hostname.

#### sap_swpm_java_scs_instance_nr
- _Type:_ `string`

Define SAP Netweaver JAVA Central Services (SCS) instance number.

#### sap_swpm_java_scs_instance_hostname
- _Type:_ `string`

Define SAP Netweaver JAVA Central Services (SCS) hostname.

#### sap_swpm_master_password
- _Type:_ `string`

Define master password used for all users created during SWPM execution.

#### sap_swpm_ddic_000_password
- _Type:_ `string`

Define DDIC user password in client 000 for new install, or existing for restore.

#### sap_swpm_virtual_hostname
- _Type:_ `string`

Define virtual hostname when installing High Available instances (e.g. SAP ASCS/ERS cluster).<br>
The role attempts to resolve `sap_swpm_virtual_hostname` on the managed node, using DNS and /etc/hosts, and will fail<br>
if this hostname resolution fails. The role will also fail if the IPv4 address for `sap_swpm_virtual_hostname` is<br>
not part of the IPv4 addresses of the managed node.

### Variables specific to SAP HANA Database Installation

#### sap_swpm_db_ip
- _Type:_ `string`

Define IP Address of database host for /etc/hosts update.

#### sap_swpm_db_fqdn
- _Type:_ `string`

Define FQDN of database host for /etc/hosts update.

#### sap_swpm_db_host
- _Type:_ `string`

Define hostname of database host for /etc/hosts update.

#### sap_swpm_db_sid
- _Type:_ `string`

Define database ID (SID).

#### sap_swpm_db_instance_nr
- _Type:_ `string`

Define database instance number.

#### sap_swpm_db_system_password
- _Type:_ `string`

Define database SYSTEM user password.

#### sap_swpm_db_systemdb_password
- _Type:_ `string`

Define database SYSTEMDB user password.

#### sap_swpm_db_sidadm_password
- _Type:_ `string`

Define database sidadm user password.

#### sap_swpm_db_schema_abap
- _Type:_ `string`

Define ABAP database schema name based on database type:</br>
- `SAPHANADB` for SAP HANA
- `ABAP` or `SAPABAP1` on IBM Db2
- `SAPSR3` on Oracle DB

#### sap_swpm_db_schema_abap_password
- _Type:_ `string`

Define ABAP database schema password for new installation or restore from backup.

#### sap_swpm_db_schema_java
- _Type:_ `string`

Define JAVA database schema name.

#### sap_swpm_db_schema_java_password
- _Type:_ `string`

Define JAVA database schema password for new installation or restore from backup.

#### sap_swpm_db_schema
- _Type:_ `string`

#### sap_swpm_db_schema_password:
- _Type:_ `string`


### Variables specific to SAP JAVA UME

#### sap_swpm_ume_client_nr
- _Type:_ `string`
- _Default:_ `000`

Define client number of JAVA UME.

#### sap_swpm_ume_type
- _Type:_ `string`

Define type of JAVA UME.

#### sap_swpm_ume_instance_nr
- _Type:_ `string`
- _Default:_ `{{ sap_swpm_pas_instance_nr }}`

Define instance number of JAVA UME.

#### sap_swpm_ume_j2ee_admin_password
- _Type:_ `string`

Define admin password for JAVA UME.

#### sap_swpm_ume_j2ee_guest_password
- _Type:_ `string`

Define guest password for JAVA UME.

#### sap_swpm_ume_sapjsf_password
- _Type:_ `string`

Define sapjsf password for JAVA UME.

#### sap_swpm_ume_instance_hostname
- _Type:_ `string`

Define instance hostname of JAVA UME.


### Variables specific to SAP HANA Database Restore

#### sap_swpm_backup_location
- _Type:_ `string`

Define directory with SAP HANA Backup files.

#### sap_swpm_backup_prefix
- _Type:_ `string`

Define prefix of SAP HANA Backup files.

#### sap_swpm_backup_system_password
- _Type:_ `string`

Define SAP HANA SYSTEM password from source database.

### sap_hana_backup_rootkeys_filename
- _Type:_ `string`

Define the rootkey filename located in the backup location

### sap_hana_backup_rootkeys_password
- _Type:_ `string`

Password of the System user of the source SAP HANA system


### Variables specific to SAP Web Dispatcher

#### sap_swpm_wd_instance_nr
- _Type:_ `string`

Define instance number of SAP Web Dispatcher.

#### sap_swpm_wd_system_connectivity
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to set parameter `configureSystemConnectivity` to true during installation.

#### sap_swpm_wd_activate_icf
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to activate ICF.

#### sap_swpm_wd_backend_sid
- _Type:_ `string`

Define backend SID for SAP Web Dispatcher connection.

#### sap_swpm_wd_backend_ms_http_port
- _Type:_ `string`

Define backend message server HTTP port for SAP Web Dispatcher connection.

#### sap_swpm_wd_backend_ms_host
- _Type:_ `string`

Define backend message server hostname for SAP Web Dispatcher connection.

#### sap_swpm_wd_backend_rfc_host
- _Type:_ `string`

Define backend hostname for SAP Web Dispatcher RFC connection.

#### sap_swpm_wd_backend_rfc_instance_nr
- _Type:_ `string`

Define backend instance number for SAP Web Dispatcher RFC connection.

#### sap_swpm_wd_backend_rfc_client_nr
- _Type:_ `string`
- _Default:_ `000`

Define backend SAP client for SAP Web Dispatcher RFC connection.

#### sap_swpm_wd_backend_rfc_user
- _Type:_ `string`
- _Default:_ `DDIC`

Define backend SAP user for SAP Web Dispatcher RFC connection.

#### sap_swpm_wd_backend_rfc_user_password
- _Type:_ `string`

Define password for backend SAP user for SAP Web Dispatcher RFC connection.

#### sap_swpm_wd_backend_scenario_size
- _Type:_ `string`

Define to set parameter `scenarioSize` during installation.

#### sap_swpm_wd_virtual_host
- _Type:_ `string`

Define virtual hostname of SAP Web Dispatcher.


### Variables for Linux User

#### sap_swpm_sapadm_password
- _Type:_ `string`

Define password for Linux user SAPADM.

#### sap_swpm_sap_sidadm_password
- _Type:_ `string`

Define password for Linux user SIDADM.

#### sap_swpm_sapadm_uid
- _Type:_ `string`

Define UID of Linux user SAPADM.

#### sap_swpm_sapsys_gid
- _Type:_ `string`

Define GID of Linux group SAPSYS.

#### sap_swpm_sidadm_uid
- _Type:_ `string`

Define UID of Linux user SIDADM.


### Miscellaneous Variables

#### sap_swpm_ascs_install_gateway
- _Type:_ `string`
- _Default:_ `true`

Enable to install gateway as part of ASCS installation.

#### sap_swpm_parallel_jobs_nr
- _Type:_ `string`
- _Default:_ `23`

Define to limit number of parallel extraction SAP HANA jobs.

#### sap_swpm_diagnostics_agent_password
- _Type:_ `string`

Define password for Diagnostic Agent.

#### sap_swpm_fqdn
- _Type:_ `string`

Define FQDN for SAP Installation.

#### sap_swpm_set_fqdn
- _Type:_ `bool`
- _Default:_ `true`

Set to `false` to disable enabling of FQDN during SWPM.

#### sap_swpm_use_password_file
- _Type:_ `string`
- _Default:_ `n`

Set to `y` to use encrypted password file for SWPM execution.</br>Location has to be same as parameter file location.

#### sap_swpm_password_file_path
- _Type:_ `string`

Define path to encrypted password file, used when `sap_swpm_use_password_file` is set to `y`.

#### sap_swpm_use_livecache
- _Type:_ `bool`
- _Default:_ `false`

Enable to use Livecache

#### sap_swpm_load_type
- _Type:_ `string`
- _Default:_ `SAP`

Define load type parameter `loadType`.

#### sap_swpm_generic
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to execute `sap_swpm` role in generic auto-detection mode, ignoring steps for individual detection.

#### sap_swpm_swpm_installation_type
- _Type:_ `string`

Define installation type method. Available types: `restore`, `ha`, `maint_plan_stack`, `ha_maint_plan_stack`.</br>
Installation type is auto-detected from `sap_swpm_product_catalog_id`.

#### sap_swpm_swpm_command_virtual_hostname
- _Type:_ `string`

Define to override default virtual hostname `sap_swpm_virtual_hostname` sapinst command used for HA installation.

#### sap_swpm_swpm_command_mp_stack
- _Type:_ `string`

Define to override default sapinst command parameter for Maintenance Plan stack file.

#### sap_swpm_setup_firewall
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to configure firewall after SWPM execution.

#### sap_swpm_update_etchosts
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to  to update `/etc/hosts` file with SAP system details for SWPM execution.

#### sap_swpm_display_unattended_output
- _Type:_ `bool`
- _Default:_ `false`

Set to `true` to display what sapinst command is being executed.


### Variables for setting owner, group, and permissions for the SAP files in `sap_swpm_software_path`

#### sap_swpm_set_file_permissions
- _Type:_ `bool`
- _Default:_ `true`

Set to `false` to not change the owner, group, and permissions of the files in `sap_swpm_software_path`.

#### sap_swpm_software_directory_mode
- _Type:_ `string`
- _Default:_ `0755`

Set permissions for all directories in `sap_swpm_software_path`.

#### sap_swpm_software_directory_owner
- _Type:_ `string`
- _Default:_ `root`

Set owner for all directories in `sap_swpm_software_path`.

#### sap_swpm_software_directory_group
- _Type:_ `string`
- _Default:_ `root`

Set group ownership for all directories in `sap_swpm_software_path`.

#### sap_swpm_files_sapcar_mode
- _Type:_ `string`
- _Default:_ `0755`

Set permissions for SAPCAR file in `sap_swpm_sapcar_path`.

#### sap_swpm_files_sapcar_owner
- _Type:_ `string`
- _Default:_ `root`

Set owner for SAPCAR file in `sap_swpm_sapcar_path`.

#### sap_swpm_files_sapcar_group
- _Type:_ `string`
- _Default:_ `root`

Set group ownership for SAPCAR file in `sap_swpm_sapcar_path`.

#### sap_swpm_files_non_sapcar_mode
- _Type:_ `string`
- _Default:_ `0644`

Set permissions for all non-SAPCAR files in `sap_swpm_software_path` and for SWPM*.SAR files in `sap_swpm_swpm_path`.

#### sap_swpm_files_non_sapcar_owner
- _Type:_ `string`
- _Default:_ `root`

Set owner for all non-SAPCAR files in `sap_swpm_software_path` and for SWPM*.SAR files in `sap_swpm_swpm_path`.

#### sap_swpm_files_non_sapcar_group
- _Type:_ `string`
- _Default:_ `root`

Set group ownership for all non-SAPCAR files in `sap_swpm_software_path` and for SWPM*.SAR files in `sap_swpm_swpm_path`.

### sap_swpm_sapinst_instdir
- _Type:_ `string`
- _Default:_ `/tmp/sapinst_instdir`

Set the installation log directory used by SWPM.

### sap_swpm_sapinst_gid
- _Type:_ `string`
- _Default:_ `(undefined)`

Set the group ID for the sapinst group. If not defined, the group is created with an auto-assigned ID.


<!-- END Role Variables -->
