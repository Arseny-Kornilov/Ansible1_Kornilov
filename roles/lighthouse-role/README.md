Role Name
lighthouse-role

A brief description of the role goes here.

Requirements
gpg packet for signed binaries

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables

Vars:
lighthouse_version: "v8.1.3"
lighthouse_arch_map:
  x86_64: "x86_64-unknown-linux-gnu"
  aarch64: "aarch64-unknown-linux-gnu"

Defaults:
lighthouse_install_dir: "/opt/lighthouse"
lighthouse_user: "lighthouse"
lighthouse_group: "lighthouse"
lighthouse_data_dir: "/var/lib/lighthouse"
lighthouse_pgp_key: "15E66D941F697E28F49381F426416DC3F30674B0"
lighthouse_download_base: "https://github.com/sigp/lighthouse/releases/download/{{ lighthouse_version }}"


A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
- name: Create lighthouse user and group
  user:
    name: "{{ lighthouse_user }}"
    group: "{{ lighthouse_group }}"
    shell: /bin/false
    home: "{{ lighthouse_data_dir }}"
    system: yes

- name: Import Sigma Prime PGP key
  apt_key:
    id: "{{ lighthouse_pgp_key }}"
    keyserver: keyserver.ubuntu.com
  when: ansible_os_family == "Debian"

- name: Determine architecture-specific filename
  set_fact:
    lighthouse_arch_suffix: "{{ lighthouse_arch_map[ansible_architecture] | default(ansible_architecture) }}"
    lighthouse_binary_file: "lighthouse-{{ lighthouse_version }}-{{ lighthouse_arch_suffix }}.tar.gz"
    lighthouse_signature_file: "{{ lighthouse_binary_file }}.asc"

- name: Download Lighthouse binary
  get_url:
    url: "{{ lighthouse_download_base }}/{{ lighthouse_binary_file }}"
    dest: "/tmp/{{ lighthouse_binary_file }}"

- name: Download PGP signature
  get_url:
    url: "{{ lighthouse_download_base }}/{{ lighthouse_signature_file }}"
    dest: "/tmp/{{ lighthouse_signature_file }}"

- name: Verify PGP signature
  command: |
    gpg --verify "/tmp/{{ lighthouse_signature_file }}" "/tmp/{{ lighthouse_binary_file }}"
  args:
    chdir: /tmp

- name: Extract Lighthouse
  unarchive:
    src: "/tmp/{{ lighthouse_binary_file }}"
    dest: "{{ lighthouse_install_dir }}"
    remote_src: yes
    extra_opts: [--strip-components=1]

- name: Make lighthouse binary executable
  file:
    path: "{{ lighthouse_install_dir }}/lighthouse"
    mode: '0755'

- name: Create systemd service
  template:
    src: lighthouse.service.j2
    dest: "/etc/systemd/system/lighthouse.service"

- name: Reload systemd
  systemd:
    daemon_reload: yes

- name: Start and enable Lighthouse service
  systemd:
    name: lighthouse
    state: started
    enabled: yes


Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information

Role By Arseny Kornilov

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
