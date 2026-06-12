# Role Name
## Vector Role

# Role Variables

## Variables:
```yaml
vector_config_dir: "{{ ansible_env.HOME }}/.vector/config"
vector_config_file: "{{ vector_config_dir }}/vector.yaml"
vector_data_dir: "{{ ansible_env.HOME }}/.vector/data"
vector_user: "{{ ansible_user }}"
```
## Defaults:
```yaml
local_vector_bin: "{{ ansible_env.HOME }}/.vector/bin/vector"
vector_log_dir: "{{ ansible_env.HOME }}/.vector/logs"
vector_nohup_log: "{{ ansible_env.HOME }}/vector.log"
```


# Example Playbook
```yaml
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

```

# License
-------

## BSD

# Author Information

## Role by Arseny Kornilov

