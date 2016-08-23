Header
-----

# code

~~~~
  - name: install common Slurm packages
    package: name={{ item }} state=present
    with_items: "{{ slurm_packages }}"
    when: slurm_packages.0 != ""

~~~~
