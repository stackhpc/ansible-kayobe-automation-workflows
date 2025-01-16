Kayobe Automation Pipeline (GitLab)
===================================

This Ansible role is capable of generating Gitlab pipeline files for performing CI/CD related activities with OpenStack via Kayobe.
See the table below for a full list of all the currently supported Kayobe automation tasks.

| **Name** | **Description** | **Stage** |
|:---:|:---:|:---:|
| **build Kayobe docker image** | Build a new Kayobe docker image whenever a new tag is pushed to the repository. The resulting image is then pushed to a docker registry such as [registry.gitlab.com/](registry.gitlab.com/). | build |
| **overcloud container image build** | Build overcloud container images. | overcloud |
| **overcloud container image pull** | Pull overcloud container images from a container registry. | overcloud |
| **overcloud database backup** | Perform a backup of the database used by the overcloud. | overcloud |
| **overcloud database recover** | Recover the database used by the overcloud. | overcloud |
| **overcloud deployment image build** | Build the Ironic Python Agent (IPA) image. | overcloud |
| **overcloud host command run** | Run a command against the overcloud hosts. | overcloud |
| **overcloud host configure** | Perform an overcloud host configure. | overcloud |
| **overcloud host image build** | Build the the image that would deployed to overcloud hosts during provisioning. | overcloud |
| **overcloud host package update** | Perform an overcloud host package update. | overcloud |
| **overcloud host upgrade** | Perform targeted upgrade of key services before an upgrade. | overcloud |
| **overcloud inventory discover** | Get an inventory of nodes. | overcloud |
| **overcloud provision** | Provision overcloud nodes. | overcloud |
| **overcloud prune docker images** | Prune docker images from overcloud hosts. | overcloud |
| **overcloud service configuration generate** | Generate the overcloud service configuration. | overcloud |
| **overcloud service deploy** | Deploy overcloud services. | overcloud |
| **overcloud service reconfigure** | Reconfigure overcloud services. | overcloud |
| **overcloud service upgrade** | Perform an upgrade of overcloud services. | overcloud |
| **seed container image build** | Build container images for seed. | overcloud |
| **seed host configure** | Configure the seed host. | seed |
| **seed host package update** | Update the system packages of the seed host. | seed |
| **seed hypervisor host configure** | Configure the seed hypervisor host. | seed |
| **seed hypervisor host package update** | Perform a package update of the seed hypervisor host. | seed |
| **seed service deploy** | Deploy services on the seed. | seed |
| **seed vm provision** | Provision the seed VM. | seed |
| **infra vm host configure** | Perform a host configure of the infra VMs on demand. | infra vm |
| **infra vm host package update** | Perform a package update of the infra VMs hosts on demand. | infra vm |
| **infra vm provision** | Provision infra VMs on demand. | infra vm |
| **infra vm service deploy** | Perform a service deploy against infra VMs on demand. | infra vm |
| **network connectivity check** | Execute a network connectivity check to ensure all hosts are reachable and can reach `nc external ip ` & `nc external hostname`. | network |
| **physical network configure** | Configure the physical network. | network |
| **config diff** | When a pull request is opened generate diff showing the changes made to the configuration. | merge request |
| **tempest** | Perform tests against the deployed openstack environment with tempest. | tempest |

Also available in Kayobe Automation for GitLab are runbooks which are stages designed to complete complex task such as upgrading all hypervisors in safe and reliable manner.

> :warning: The runbooks are **experimental** and may require careful consideration before being deployed.

| **Name** | **Description** |
|:---:|:---:|
| **runbook overcloud service upgrade** | Perform an upgrade of overcloud services; pull containers, upgrade host, run tempest, backup database, upgrade services, run tempest and prune unused docker images. |
| **runbook in place hypervisor host upgrade** | Perform an in place host upgrade of hypervisors; disable compute services, drain hypervisor, package update, reboot and renable compute services. |

Role Variables
--------------

The following variables can be used to make small adjustments to the composition of the workflows.

`gitlab_output_directory`: control the location where the workflows shall be written to.

`gitlab_Kayobe_environments`: list of environments the workflows should target.

`gitlab_registry`: a string that either points to a registry or is `$CI_REGISTRY_URL` in the case of multiple environments that do not share the same registry.

`gitlab_image_name`: name of the Kayobe image defaults to `Kayobe`.

`gitlab_image_tag`: tag used to select Kayobe image defaults to `latest`

`gitlab_Kayobe_base_image`: select the base image used when building the Kayobe docker image. Default is `quay.io/rockylinux/rockylinux:9`.

`gitlab_tempest_test_suites`: a list of options to be made available within the drop-down list for running tempest. Defaults to `default` and `tempest-full`.

If you wish to make more impactful changes such as which stages and built and what jobs are contained when review `gitlab_stages` in `defaults/main.yml`

`gitlab_stages:` is a dictionary of dictionaries that contains string blocks defining the job within a given stage. Any job may be overwritten by editing the `gitlab_JOB_NAME` or a stage can be extended adding to the `gitlab_STAGE_NAME_extra` dictionary. If you require adding additional stages that add the stage to `gitlab_extra_stages`.

```
gitlab_infra_vm_host_configure: |
  rules:
    - !reference [.active_stage_web_rule, rules]
  stage: infra-vm
  resource_group: infra-vm
  script:
    - !reference [.get_secrets, script]
    - .automation/pipeline/infra-vm-host-configure.sh
  allow_failure: true
```

```
gitlab_infra_vm_something_new: |
  rules:
    - !reference [.active_stage_web_rule, rules]
  stage: infra-vm
  resource_group: infra-vm
  script:
    - !reference [.get_secrets, script]
    - .automation/pipeline/does_something_new.sh
  allow_failure: true

gitlab_stage_infra_vm_extra:
  infra_vm_something_new: "{{ gitlab_infra_vm_something_new }}"
```

Finally, `gitlab_runbooks` is collection of runbooks which each compose a single stage that is designed complete a larger more involved task. For example upgrade overcloud services complete with backup of database and tempest testing both before and after. Runbooks are created similiar to normal jobs however they may need altering or even overriding to meet the needs of a given environment. 

Example Playbook
----------------

The following example playbook will generate a `reference` pipeline which can be found under `.gitlab` and `.gitlab/..`

```yaml
- name: Write Kayobe Automation Pipeline for gitlab
  hosts: localhost
  roles:
    - stackhpc.Kayobe_workflows.gitlab
```

License
-------

Apache License 2.0

Author Information
------------------

[StackHPC](https://www.stackhpc.com/)
