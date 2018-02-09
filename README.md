Ansible-role-haproxy
=========

Installs HAProxy on RedHat/CentOS server.

**Note**: This roles use _community_ HAProxy package in version 1.8.

Requirements
------------

This role use custom community package of HAProxy 1.8.4, url is define in defaults variable files.

Role Variables
--------------

All default variables are predefined in [defaults/main.yml](defaults/main.yml).
Example playbook usage is also define in [vars/main.yml](vars/main.yml).

**Note**: An attribute `haproxy_nbproc: 4` should be equal to the number of cpu on haproxy machines.  

**Note**: A variable `haproxy_stats_pass:` should be change before you use this role.

Example Playbook
----------------

Example playbook usage.

```
- name: Run nginx-ingress
  hosts: 'haproxy'
  vars:
   # Below there are some ansible values that could be also define in ansible.cfg
   ansible_become: yes
   ansible_connection: ssh
   ansible_user: ansible
   ansible_ssh_private_key_file: test_key.pem
   #role configuration parameters.
   haproxy_max_conn: 2000000
   haproxy_services:
     - name: "k8s"
       type: frontend
       binds:
         - { host: "*", port: "80" }
       mode: http
       acls:
         - { name: "forward_k8s", rule: "hdr_end(host) -i k8s.example.com" }
       conditions:
         - { state: "use_backend k8s_backend", rule: "forward_k8s" }
       default_backend: company_site
     - name: "company_site"
       type: backend
       balance_type: roundrobin
       options:
         - "option forwardfor"
         - "http-request set-header X-Forwarded-Port %[dst_port]"
       endpoints:
         - { name: "backend_1", address: "192.168.0.1:80", options: "check" }
         - { name: "backend_2", address: "192.168.0.2:80", options: "check" }
     - name: "k8s_backend"
       type: backend
       balance_type: roundrobin
       options:
         - "option forwardfor"
         - "http-request set-header X-Forwarded-Port %[dst_port]"
       endpoints:
         - { name: "backend_1", address: "192.168.0.3:8080", options: "check" }
         - { name: "backend_2", address: "192.168.0.4:8080", options: "check" }
  roles:
       - ansible-role-haproxy


```

License
-------

MIT

TODO
---------------
* Custom error pages
* Log customization
* Add more complex test

Author Information
------------------

This role was created in 2018 for Novomatic Technologies Poland purposes.