# FGCI/CSC

What we do - homogeneousish configuration. 

# Stack:

 - Ansible
 - NorduGrid ARC CE
 - CentOS7
 - CVMFS/modules
 - Dell Hardware
 - Infiniband
 - KVM
 - NFSv4
 - Slurm

#HSLIDE

# what is ansible?

~~~~
  - name: start and enable slurmctld
    service: name={{ slurmctld_service }} state=started enabled=yes
~~~~

#HSLIDE

# ansible-role-slurm

 - ansible-playbook install.yml -t slurm --list-tags

... too many

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

#VSLIDE

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

# credits

 - @jabl
 - @A1ve5
 - me

