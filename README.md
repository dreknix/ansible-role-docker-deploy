# Ansible Role - Docker Deploy

Ansible role for deploying Docker compose instances

## Using in a Playbook

For each Docker compose instance a systemd service unit is created. The template
can be found in the [Ansible role for Docker](https://github.com/dreknix/ansible-role-docker).

The Docker compose file can be provided via a Git repository of by a template.
All files that are not in the Git repository will be transferred via Ansible
templates. The templates must be stored in a directory beneath the directory
`templates`.

If the variable `docker_deploy_git_repo` is not set, a directory with the name
`docker_deploy_name` will be created.

``` yaml
- name: Deploy Docker compose instance GitLab runner
  hosts: docker_gitlab_runner
  tasks:
    - name: Import role 'dreknix.docker_deploy'
      ansible.builtin.import_role:
        name: dreknix.docker_deploy
      vars:
        docker_deploy_name: gitlab-runner
        docker_deploy_template_dirs:
          - "{{ playbook_dir }}/templates/gitlab-runner"
```

A more complex example:

``` yaml
---
- name: Deploy Docker compose instances
  hosts:
    - docker_base
  vars:
    docker_compose_instances:
      - name: resticer
        state: absent
      - name: base
        git_repo: https://github.com/dreknix/docker-compose-base
        template_dirs:
          - "{{ playbook_dir }}/templates/base/all"
          - "{{ playbook_dir }}/templates/base/{{ inventory_hostname }}"
        touched_files:
          - path: traefik-certs/acme.json
            mode: u=rw,go=
        delete_unmanaged_files: true
        volumes:
          - base_portainer_data
        networks:
          - frontend
          - backend
  tasks:
    - name: Include role 'dreknix.docker_deploy'
      ansible.builtin.include_role:
        name: dreknix.docker_deploy
      vars:
        docker_deploy_name: "{{ docker_compose_instance.name }}"
        docker_deploy_state: "{{ docker_compose_instance.state | default('present') }}"
        docker_deploy_git_repo: "{{ docker_compose_instance.git_repo }}"
        docker_deploy_file_dirs: "{{ docker_compose_instance.file_dirs | default([]) }}"
        docker_deploy_template_dirs: "{{ docker_compose_instance.template_dirs | default([]) }}"
        docker_deploy_touched_files: "{{ docker_compose_instance.touched_files | default([]) }}"
        docker_deploy_delete_unmanaged_files: "{{ docker_compose_instance.delete_unmanaged_files | default(false) }}"
        docker_deploy_volumes: "{{ docker_compose_instance.volumes | default([]) }}"
        docker_deploy_networks: "{{ docker_compose_instance.networks | default([]) }}"
      loop: "{{ docker_compose_instances }}"
      loop_control:
        loop_var: docker_compose_instance
...
```

## Install role via `roles/requirements.yml`

### Read-Only copy

Configure the role as read-only copy:

```yml
---
# Docker/Docker-Compose
- name: dreknix.docker_deploy
  source: https://github.com/dreknix/ansible-role-docker-deploy.git
  type: git
  version: main
...
```

Install the role:

```console
$ ansible-galaxy role install --force -r roles/requirements.yml
```

### Working copy

Configure the role as a working copy:

```yml
---
# Docker/Docker-Compose
- name: dreknix.docker_deploy
  source: git@github.com:dreknix/ansible-role-docker-deploy.git
  type: git
  version: main
...
```

Install the working copy:

```console
$ ansible-galaxy role install --force --keep-scm-meta -r roles/requirements.yml
```

## License

[MIT](https://github.com/dreknix/ansible-role-docker-deploy/blob/main/LICENSE)
