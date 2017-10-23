# telepie

An opinionated Ansible quick start for Python application deployment

This is a skeleton of what I consider the best practices for deploying Python apps with Ansible.

There is no attempt to cover all possible environments out there, this grew out of a particular 
set of circumstances I operated in during the last couple of years. 
These constraints/assumptions are:
- the Python app is developed in a GitHub repo, as a package
- the app hosts run it as a systemd service 
- the app is deployed as a virtual environment, in `/opt/<service user home>/virtualenvs/`
- the deployment process consists of: 
  - preparing a virtualenv with the app's package and all dependencies on a build host
  - copying the virtualenv in place on the application hosts   
- (optional) deployment progress notifications use HipChat

Host groups
===========

Two groups are provided: 
- build - normally this only requires a single host
- app - the application hosts
 
You should change the app group and role names to match your app, and update them in the environment as well 
(a `staging` enviromment file is provided as an example).   

 
Host preparation
================

Setting up the application user, directories, and permissions on both the build and application hosts is done 
by running the tasks tagged with `prep`. For app machines this also covers setting up the app's service unit.

```bash
ansible-playbook -i staging site.yml --tags="prep"
```

Build and deployment
====================
The tags `build` and `deploy` cover the tasks for the respective activities.

Examples:

```bash
ansible-playbook -i staging site.yml --tags="build,deploy"
```

When deploying the same version to several environments, a single `build` is needed:
```bash
ansible-playbook -i staging site.yml --tags="build,deploy" --extra-vars="version=v1.2.3"
ansible-playbook -i production site.yml --tags="deploy"
```


Variables
=========

For simplicity and sanity, the variables are set up to propagate the app's Python package name as: 
- virtualenv name
- OS user/name
- systemd service name

Of course, these defaults can easily be overriden in the group or environment vars.

As a consequence, the minimal required configuration is setting the `pkg` variable in `group_vars/all/vars`.

Security
========

An unencrypted vault for the HipChat token is prepared in `group_vars/all/vault`, just provide your token before using 
`ansible-vault encrypt` on it.
