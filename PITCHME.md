#Configure a slurm cluster with ansible

CSC - IT Center for Science Ltd

Johan Guldmyr, Slurm User Group 2016, Athens
![FGCILogo](images/FGCI-logo.jpg)

#HSLIDE

# FGCI/CSC

What we do - homogeneousish configuration. 

# Stack:

 - Ansible, CentOS7, CVMFS/modules/Easybuild, Dell, Infiniband
 - KVM, NFSv4, NorduGrid ARC, PXE/kickstart, **Slurm 15.08**

#HSLIDE

# credits

 - @jabl
 - @A1ve5
 - me

#HSLIDE

# ansible is a config management tool

 - similar to puppet, quattor, cfengine, salt, chef
 - no daemon, uses ssh for push mode or cronjob for pull

#HSLIDE

~~~~
  - name: start and enable slurmctld
    service: name={{ slurmctld_service }} state=started enabled=yes
~~~~

#HSLIDE

# ansible-role-slurm

 - ansible-playbook install.yml -t slurm --list-tags

... too many

#HSLIDE

![FGCILogo](images/slurm-list-tasks.gif)

#HSLIDE
what it does (shortest version)

#Configure all the things for slurm

picture.png

#HSLIDE

what it does (shorter version)

configure:
 - one service node
 - one dbd node (could be the same as the service node)
 - submit nodes
 - compute nodes

#HSLIDE

what it does (short version)

 - add yum repo, install slurm, munge, mysql
 - add users
 - create directories
 - template in config files (slurm, gres, cgroup.conf)
 - generate or copy in munge key
 - create cluster in accounting
 - setup epilog
 - increase sysctl values
 - setup PAM + security/access.conf to restrict ssh access

#HSLIDE

# another example

~~~~
  - name: install common Slurm packages
    package: name={{ item }} state=present
    with_items: "{{ slurm_packages }}"
    when: slurm_packages.0 != ""

~~~~

#HSLIDE

# slurm.conf and ansible jinja variables

~~~~
ClusterName={{ slurm_clustername }}
ControlMachine={{ slurm_service_node }}
~~~~

#HSLIDE

~~~~
  - name: sacctmgr show cluster siteName and store in slurm_clusterlist variable
    command: "sacctmgr -n show cluster {{ siteName }}"
    register: slurm_clusterlist
    always_run: yes
    changed_when: False

  - name: add cluster to accounting
    command: "sacctmgr -i add cluster {{ siteName }}"
    when: slurm_clusterlist.stdout.find("{{siteName}}") == -1
~~~~

#HSLIDE

#(single centos7 node) part 1
~~~~
#If the node has hostname "slurm-ansible" configure your ssh/config so you can ssh+sudo to that
sudo yum install ansible
mkdir -p slurm_workdir/{files,roles,group_vars/all}; cd slurm_workdir
git clone https://github.com/CSC-IT-Center-for-Science/ansible-role-slurm roles/ansible-role-slurm
git clone https://github.com/jabl/ansible-role-pam roles/ansible-role-pam
git clone https://github.com/CSC-IT-Center-for-Science/ansible-role-nhc roles/ansible-role-nhc
cp roles/ansible-role-slurm/tests/inventory .
cp roles/ansible-role-slurm/tests/test.yml .
$EDITOR inventory # put "slurm-ansible" under [install] and [compute]
$EDITOR test.yml # for sudo set these in test.yml:
# remote_user: $USER
# become: True 
ansible-playbook -i inventory test.yml --diff # Then run ansible!
~~~~

#HSLIDE

![ansible-role-slurm](images/ansible_role_slurm_1.gif)

#HSLIDE

#(single node) part 2

If you want 16.05 set this variable (for example in test.yml):

~~~~
     - fgci_slurmrepo_version: "fgcislurm1605"
~~~~

#HSLIDE

# todolist:
 - When to restart or just HIP after a config change?
 - Topology Generation
 - HA?
 - Complete Testing (setup a slurm cluster and even submit a job on every change to the ansible role)

#HSLIDE

~8 clusters in Finland (the ones part of FGCI) are using https://github.com/CSC-IT-Center-for-Science/fgci-ansible which uses this ansible-role-slurm role. 
Triton is the largest cluster with ~613 nodes

#HSLIDE

![travis](images/TravisCI-Full-Color-7f5db09495c8b09c21cb678c4de18d21.png) 
![waffle](images/waffle_github.png) ![grafana](images/grafana_logo_new_transparent.png)
![influxdb](images/illustration_influxdb.svg) 

#HSLIDE

 - ansible-push
 - ansible-pull
 - ansible-galaxy and requirements.yml to restrict which version/commit of an ansible role is used
 - git-mirror
