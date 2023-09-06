Kayobe Automation Workflows (GitHub)
=========

This Ansible role is capable of generating GitHub workflow files for performing CI/CD related activities with OpenStack via Kayobe.
See the table below for a full list of all the currently supported kayobe automation tasks.

| **Name** | **Description** |
|:---:|:---:|
| **build-kayobe-docker-image** | Build a new kayobe docker image whenever a new tag is pushed to the repository. The resulting image is then pushed to a docker registry such as [ghcr.io](http://ghcr.io). |
| **run-config-diff** | When a pull request is opened generate diff showing the changes made to the configuration. |
| **run-infra-vm-host-package-update** | Perform a package update of the infra VMs hosts on demand. |
| **run-infra-vm-host-configure** | Perform a host configure of the infra VMs on demand. |
| **run-infra-vm-provision** | Provision infra VMs on demand. |
| **run-infra-vm-service-deploy** | Perform a service deploy against infra VMs on demand. |
| **run-network-connectivity-check** | Execute a network connectivity check to ensure all hosts are reachable and can reach `nc_external_ip ` & `nc_external_hostname`. |
| **run-overcloud-container-image-pull** | Pull container images from a container registry. |
| **run-overcloud-database-backup** | Perform a backup of the database used by the overcloud. |
| **run-overcloud-host-configure** | Perform an overcloud host configure. |
| **run-overcloud-host-package-update** | Perform an overcloud host package update. |
| **run-overcloud-inventory-discover** | Get an inventory of nodes. |
| **run-overcloud-provision** | Provision overcloud nodes. |
| **run-overcloud-service-deploy** | Deploy overcloud services. |
| **run-overcloud-service-upgrade** | Perform an upgrade of overcloud services. |
| **run-seed-host-configure** | Configure the seed host. |
| **run-seed-host-package-update** | Update the system packages of the seed host. |
| **run-seed-hypervisor-host-configure** | Configure the seed hypervisor host. |
| **run-seed-hypervisor-host-package-update** | Perform a package update of the seed hypervisor host. |
| **run-seed-service-deploy** | Deploy services on the seed. |
| **run-seed-vm-provision** | Provision the seed VM. |
| **run-tempest** | Perform tests against the deployed openstack environment with tempest. |

Role Variables
--------------

The following variables can be used to make small adjustments to the composition of the workflows.

`github_output_directory`: control the location where the workflows shall be written to.

`github_runs_on`: control which runner can accept this workflow. See GitHub for more information on [runs-on](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on).

`github_image_name`: name of the kayobe image defaults to `kayobe`.

`github_image_tag`: tag used to select kayobe image defaults to `latest`

`github_registry_username`: username used to authenticate with the docker registry.

`github_registry_password`: password used to authenticate with the docker registry.

`github_kayobe_arguments`: a dictionary of arguments that can be used to override the default arguments found within `vars/main.yml`. For example if you wanted to change the value of `KAYOBE_ENVIRONMENT` from its default of `production` you can simply add `KAYOBE_ENVIRONMENT` to this dictionary and it will take precedence over the defaults.

`github_*_hook:` see section [Template Hooks](#template-hooks)  for information about this variables

`github_buildx_inline_config`: provide configuration parameters to buildx. Useful for connecting to insecure docker registry.

```yaml
github_buildx_inline_config: |
  [registry."10.20.30.40:80"]
    http = true
    insecure = true
```

`github_timeout`: control how a long a job may run before being cancelled. Timeout is defined in minutes and defaults to 360 minutes (6 hours)

If you wish to make more impactful changes such as which workflows are built and what they contain then see the list of dictionaries called `workflows` in `defaults/main.yml`

`github_buildx_enable_provenance`: whether or not to enable build attestations/provenence. This has been [noted](https://github.com/docker/build-push-action/releases/tag/v4.1.1) to cause issues with docker registries such as Pulp. Default to false.

`github_workflows:` is a list of dictionaries that contains each of the workflows described above. A given list element is made up of the following:

- `name`: the name which the workflow shall refer to itself as within GitHub workflows user interface.

- `file_name`: name which the workflow will be saved as.

- `trigger`: a dictionary of event triggers that can be used to start the workflow.

- `arguments`: list of arguments keys used by the automation task the contents will be acquired from `kayobe_arguments` or the defaults.

- `path_override`: Some workflows benefit from a dedicated workflow template as the `generic.yml.j2` cannot meet the requirements of all workflows. Therefore it might be beneficial to create specific templates within the `{{ playbook_dir }}/templates/` directory and set this variable to that specific path.

- `use_bespoke`: As mentioned in `path_override` some workflows require a more specific template to meet their needs. Setting this to true would create a workflow from a bespoke template if one exists within the roles `templates` directory. Note this is intended for **internal use only**.

The following will override `github_workflows` to ensure only `Build Kayobe Image` and `Run Kolla Config Diff` is generated.

```yaml
github_workflows:
  - "{{ build_kayobe_image }}"
  - "{{ run_kolla_config_diff }}"
```

In the following example `github_build_kayobe_image` has been changed to use a user provided template stored within their `{{ playbook_dir }}/templates`

```yaml
github_build_kayobe_image:
  file_name: build-kayobe-docker-image.yml
  path_override: "{{ playbook_dir }}/templates/build-kayobe-docker-image.yml.j2"
```

Template Hooks
--------------

Workflows can be expanded with the use of hooks which are variables that if provided can be inserted into the appropriate location enabling the introduction of additional steps within the workflow job. This could include the use of HashiCorp Vault or installing and configuring a network proxy.

There are currently three hooks available

- `github_checkout_hook`: a hook that occurs before the repository is cloned by the `checkout` action.

- `github_kayobe_hook`: a hook that occurs after the the repository has been cloned and before the kayobe automation task has started.

- `github_final_hook`: a hook that occurs after the kayobe automation task has finished.

A hook must be defined as a variable and should be a scalar block string.

```yaml
github_checkout_hook: |
  - name: Import secrets via Hashicorp Vault
    id: secrets
    uses: hashicorp/vault-action@v2.5.0
    with:
      url: https://vault.stackhpc.com:8200
      method: approle
      roleId:  ${{ secrets.ROLE_ID }}
      secretId: ${{ secrets.SECRET_ID }}
      tlsSkipVerify: true
      secrets: |
        stackhpc/data/github kayobe_vault_password_${{ needs.env.outputs.environment }} | KAYOBE_VAULT_PASSWORD ;
        stackhpc/data/github kayobe_automation_ssh_private_key_${{ needs.env.outputs.environment }} | KAYOBE_AUTOMATION_SSH_PRIVATE_KEY ;
```

Example Playbook
----------------

The following example playbook will generate a series of `reference` workflows which can be found under `.github/workflows`

```yaml
- name: Write Kayobe Automation Workflows for GitHub
  hosts: localhost
  roles:
    - stackhpc.kayobe_automation_workflows.github
```

License
-------

Apache License 2.0

Author Information
------------------

[StackHPC](https://www.stackhpc.com/)
