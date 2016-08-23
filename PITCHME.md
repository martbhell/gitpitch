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

# ansible-role-slurm

~~~~
  - name: start and enable slurmctld
    service: name={{ slurmctld_service }} state=started enabled=yes
~~~~


# code

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


