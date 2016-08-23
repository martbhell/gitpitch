# FGCI/CSC

What we do - homogeneousish configuration. 

# Stack:

 - Ansible, CentOS 7, CVMFS/modules/Easybuild, Dell Hardware, Infiniband
 - KVM, NFSv4, NorduGrid ARC CE, **Slurm 15.08**

#HSLIDE

# credits

 - @jabl
 - @A1ve5
 - me

#HSLIDE

# ansible is a config management tool

 - similar to puppet, quattor, cfengine, salt
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
# what it does (shortest version)

Configure all the things for slurm

picture.png

#HSLIDE

# what it does (shorter version)

configure:
 - one service node
 - one dbd node (could be the same as the service node)
 - submit nodes
 - compute nodes

#HSLIDE

# what it does (short version)

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

# todolist:
 - When to restart or just HIP after a config change?
 - Topology Generation
 - HA?
 - Complete Testing (setup a slurm cluster and even submit a job on every change to the ansible role)

#HSLIDE

~8 clusters in Finland (the ones part of FGCI) are using https://github.com/CSC-IT-Center-for-Science/fgci-ansible which uses this ansible-role-slurm role. 
Triton is the largest cluster with ~613 nodes

#HSLIDE

 - github
 - ansible-push
 - ansible-pull
 - travis/waffle
 - ansible-galaxy and requirements.yml to restrict which version/commit of an ansible role is used
 - git-mirror
