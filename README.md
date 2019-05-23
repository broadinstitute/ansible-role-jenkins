# ansible-role-jenkins

### Overview:
A role to install jenkins master and slave within the DSP  environment. This role copies vault secrets and needs some sort of token to pull down tls and password information. The master also runs on docker, so docker/docker-compose is also required to be installed.

### Prerequisites:
- Vault Token
  - [ansible-role-token-copy](https://github.com/broadinstitute/ansible-role-token-copy)
- docker/docker-compose
  - [ansible-role-docker](https://github.com/broadinstitute/ansible-role-docker)


### Examples:
##### Playbook
Playbook example for master/slave. This includes basepackages, token copy, lvm, docker, stackdriver, yumcron, sentinelone, and jenkins on both master and slave in order to meet infosec requirements.
```
#duos/orsp hosts below
- hosts: duos-jenkins-master
  tasks:
  - import_role:
      name: ansible-role-clone-repos
  - include_role:
      name: "{{ item }}"
    with_items:
      - ansible-role-basepackages
      - ansible-role-token-copy
      - ansible-role-lvm
      - ansible-role-docker
      - ansible-role-stackdriver
      - ansible-role-yum-cron
      - ansible-role-sentinelone
      - ansible-role-jenkins

- hosts: duos-jenkins-slave
  tasks:
  - import_role:
      name: ansible-role-clone-repos
  - include_role:
      name: "{{ item }}"
    with_items:
      - ansible-role-basepackages
      - ansible-role-lvm
      - ansible-role-docker
      - ansible-role-token-copy
      - ansible-role-stackdriver
      - ansible-role-yum-cron
      - ansible-role-sentinelone
      - ansible-role-jenkins
```
##### Master group Vars
```
---
#additional packages
additional_packages:
  - python36
  - python36-pip
#docker vars
docker_edition: 'ce'
docker_package: "docker-{{ docker_edition }}-18.09.3-3.el7"
docker_package_state: present
docker_install_compose: true
docker_compose_version: "1.24.0"
docker_compose_path: /usr/local/bin/docker-compose
#overlay2 disk
lvm_groups:
  - vgname: vg_docker
    disks:
      - /dev/sdb
    create: true
    lvnames:
      - lvname: docker-lvm
        size: 100g
        create: true
        filesystem: xfs
        mount: true
        mntp: /var/lib/docker
manage_lvm: true
#jenkins vars
jenkins_type: "master"
jenkins_version: "lts"
jenkins_admin: dsptechops@broadinstitute.org
apache_url: "duos-jenkins.dsp-techops.broadinstitute.org"
#token copy vars
bucket_path: gs://ms-jenkins-tokens/
folder: duos-jenkins
dest_path: /tmp/
#role repos to clone
gitrepos:
#docker
  - { address: 'https://github.com/broadinstitute/ansible-role-docker.git', name: 'ansible-role-docker', version: 'master' }
#basepacks
  - { address: 'https://github.com/broadinstitute/ansible-role-basepackages.git', name: 'ansible-role-basepackages', version: 'master' }
#lvm
  - { address: 'https://github.com/broadinstitute/ansible-role-lvm.git', name: 'ansible-role-lvm', version: 'master' }
#stackdriver
  - { address: 'https://github.com/broadinstitute/ansible-role-stackdriver.git', name: 'ansible-role-stackdriver', version: 'master' }
#yumcron
  - { address: 'https://github.com/broadinstitute/ansible-role-yum-cron.git', name: 'ansible-role-yum-cron', version: 'master' }
#sentinelone
  - { address: 'https://github.com/broadinstitute/ansible-role-sentinelone.git', name: 'ansible-role-sentinelone', version: 'master' }
#Jenkins master
  - { address: 'https://github.com/broadinstitute/ansible-role-jenkins.git', name: 'ansible-role-jenkins', version: 'master' }
#token Copy
  - { address: 'https://github.com/broadinstitute/ansible-role-token-copy.git', name: 'ansible-role-token-copy', version: 'master' }
```
##### Slave group Vars
```
---
#docker vars
docker_edition: 'ce'
docker_package: "docker-{{ docker_edition }}-18.09.3-3.el7"
docker_package_state: present
docker_install_compose: true
docker_compose_version: "1.24.0"
docker_compose_path: /usr/local/bin/docker-compose
#overlay2 disk
lvm_groups:
  - vgname: vg_docker
    disks:
      - /dev/sdb
    create: true
    lvnames:
      - lvname: docker-lvm
        size: 100g
        create: true
        filesystem: xfs
        mount: true
        mntp: /var/lib/docker
manage_lvm: true
#jenkins vars
jenkins_type: "slave"
#additional packages
additional_packages:
  - kubectl
  - python36
  - python36-pip
  - java-1.8.0-openjdk
#token copy vars
bucket_path: gs://ms-jenkins-tokens/
folder: duos-jenkins
dest_path: /tmp/
#role repos to clone
gitrepos:
#stackdriver
  - { address: 'https://github.com/broadinstitute/ansible-role-stackdriver.git', name: 'ansible-role-stackdriver', version: 'master' }
#yumcron
  - { address: 'https://github.com/broadinstitute/ansible-role-yum-cron.git', name: 'ansible-role-yum-cron', version: 'master' }
#sentinelone
  - { address: 'https://github.com/broadinstitute/ansible-role-sentinelone.git', name: 'ansible-role-sentinelone', version: 'master' }
#basepacks
  - { address: 'https://github.com/broadinstitute/ansible-role-basepackages.git', name: 'ansible-role-basepackages', version: 'master' }
#Jenkins slave
  - { address: 'https://github.com/broadinstitute/ansible-role-jenkins.git', name: 'ansible-role-jenkins', version: 'master' }
#token Copy
  - { address: 'https://github.com/broadinstitute/ansible-role-token-copy.git', name: 'ansible-role-token-copy', version: 'master' }
#docker
  - { address: 'https://github.com/broadinstitute/ansible-role-docker.git', name: 'ansible-role-docker', version: 'master' }
#lvm
  - { address: 'https://github.com/broadinstitute/ansible-role-lvm.git', name: 'ansible-role-lvm', version: 'master' }
  ```
  ##### Example Results
  - 1 Master node running on docker with 100G HDD
  - 1 Slave node running on VM instance with 100G HDD with kubectl and docker installed for jenkins to use.

  ##### To-Do:
  groovy script/code to auto add slave to master
  auto add dsde secrets
  
