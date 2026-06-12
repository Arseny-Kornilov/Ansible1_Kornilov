Role Name

Vector Role

A brief description of the role goes here.

Requirements

Jinja2 Template in templates' directory

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables

Variables:

vector_config_dir: "{{ ansible_env.HOME }}/.vector/config"
vector_config_file: "{{ vector_config_dir }}/vector.yaml"
vector_data_dir: "{{ ansible_env.HOME }}/.vector/data"
vector_user: "{{ ansible_user }}"

Defaults:

local_vector_bin: "{{ ansible_env.HOME }}/.vector/bin/vector"
vector_log_dir: "{{ ansible_env.HOME }}/.vector/logs"
vector_nohup_log: "{{ ansible_env.HOME }}/vector.log"


A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook

- name: Create user vector
  ansible.builtin.user:
    name: "{{ vector_user }}"
    group: "{{ vector_group }}"
    system: yes
    shell: /usr/sbin/nologin
    create_home: no
    state: present

- name: Create system directories
  ansible.builtin.file:
    path: "{{ item }}"
    state: directory
    owner: "{{ vector_user }}"
    group: "{{ vector_group }}"
    mode: '0755'
      loop:
        - "{{ vector_config_dir }}"
        - "{{ vector_data_dir }}"
        - "{{ vector_log_dir }}"

- name: Copy binary to /usr/local/bin
  ansible.builtin.copy:
    src: "{{ local_vector_bin }}"
    dest: /usr/local/bin/vector
    mode: '0755'
    owner: root
    group: root
    remote_src: yes

- name: Copy config
  copy:
    src: /home/vboxuser/.vector/config/vector.yaml
    dest: /etc/vector/vector.yaml
    mode: '0755'
    owner: root
    group: root
    remote_src: yes

- name: Start Vector
  shell: |
    vector --config /home/vboxuser/.vector/config/vector.yaml
  args:
    executable: /bin/bash


Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information

Role by Arseny Kornilov

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
