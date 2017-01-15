# Using Letsencrypt with Haproxy


## Example

### requirements.yml
```
# Alternative src https://github.com/haos616/ansible-haproxy
- src: haos616.haproxy
  version: "0.1.3"
  name: "haproxy"
```

## Install roles
```
ansible-galaxy install -p roles/ -r requirements.yml
```

### group_vars/all.yml or host_vars/production.yml
```
# Old certs
haproxy_certs: []
haproxy_config_template: "{{ playbook_dir }}/roles/letsencrypt/templates/haproxy.cfg.j2"

letsencrypt_admin_email: "admin@example.com"

letsencrypt_certs:
  - domains:
      - 'example.com'
  - domains:
      - 'two-example.com'
      - 'blog.two-example.com'
      - 'ci.two-example.com'

haproxy_domains:
  - { name: 'example.com', host: '127.0.0.1', port: 8001 }
  - { name: 'two-example.com', host: '127.0.0.1', port: 8002 }
  - { name: 'cinematograph.live', host: '127.0.0.1', port: 8003 }
  - { name: 'blog.two-example.com', host: '127.0.0.1', port: 8004 }
  - { name: 'ci.two-example.com', host: '127.0.0.1', port: 8005 }

letsencrypt_haproxy_certs:
  - 'example.com.pem'
  - 'two-example.com.pem'
```

### hosts/production.yml
```
production ansible_ssh_host='example.com' ansible_ssh_user='root'
```

### letsencrypt-setup.yml
```
- hosts: all
  become_user: root
  become: yes
  gather_facts: yes
  tags: ['setup']
  roles:
    - { role: letsencrypt, task: init }
    - { role: haproxy, task: setup }
    - { role: letsencrypt, task: setup }
    - { role: letsencrypt, task: renew }
```

### letsencrypt-deploy.yml
```
- hosts: all
  become_user: root
  become: yes
  gather_facts: yes
  tags: ['deploy', 'setup']
  roles:
    - { role: letsencrypt, task: init }
    - { role: haproxy, task: deploy }
    - { role: letsencrypt, task: create }
    - { role: haproxy, task: deploy, fake: 1 }
```

### letsencrypt.yml
```
- include: letsencrypt-setup.yml
- include: letsencrypt-deploy.yml
```

### Setup
```
ansible-playbook letsencrypt.yml -i hosts/production.yml --tags=setup
```

### Update
```
ansible-playbook letsencrypt.yml -i hosts/production.com.yml --tags=deploy
```