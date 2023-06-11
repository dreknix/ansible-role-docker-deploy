# Ansible Role - Docker Deploy

Ansible role for deploying Docker compose instances

## Install role via `roles/requirements.yml`

### Read-Only copy

Configure the role as read-only copy:

```yml
---
# Docker/Docker-Compose
- src: https://github.com/dreknix/ansible-role-docker-deploy.git
  scm: git
  version: main
  name: docker_deploy
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
- src: git@github.com:dreknix/ansible-role-docker-deploy.git
  scm: git
  version: main
  name: docker_deploy
...
```

Install the working copy:

```console
$ ansible-galaxy role install --force --keep-scm-meta -r roles/requirements.yml
```

## Using in a Playbook

```yaml
---
- name: Deploy Docker compose instances
  hosts:
    - docker
  roles:
    - role: docker_deploy
      vars:
        docker_deploy_name: base
        docker_deploy_git_repo: https://github.com/dreknix/docker-compose-base
        docker_deploy_base_templates:
          - src: base/env.j2
            dest: .env
        docker_deploy_base_host_templates: []
        docker_deploy_templates: "{{ docker_deploy_base_templates + docker_deploy_base_host_templates }}"
      tags:
        - docker
...
```

## License

[MIT](https://github.com/dreknix/ansible-role-docker-deploy/blob/main/LICENSE)

