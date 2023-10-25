# Ansible Collection - stackhpc.kayobe_workflows

This repository contains `stackhpc.kayobe_workflows` an Ansible Collection. The collection currently includes a role for generating workflows that utilise [stackhpc/kayobe-automation](https://github.com/stackhpc/kayobe-automation) within GitHub Actions.

## Included content

Roles:
* [github](roles/github/README.md) for generating workflows appropriate for GitHub Actions

## Using this collection

Before using the collection, you need to install the collection with the `ansible-galaxy` CLI:

    ansible-galaxy collection install stackhpc.kayobe_workflows

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml` using the format:

```yaml
collections:
- name: stackhpc.kayobe_workflows
```

See [Ansible Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) for more details.
