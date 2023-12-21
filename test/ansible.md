# 检查yml格式是否正确

```shell
ansible-playbook xx.yml --syntax-check
```

# 仅检查 Playbook 是否正确-不执行任务

```shell
ansible-playbook xx.yml --check
```

# 存在即跳过任务

```shell
- name:
  hosts: web1
  tasks:
   - name: nginx started
     service: 
       name: nginx
       state: started
     args:
      creates: /usr/loacl/nginx/log/nginx.pid  # 检查是否存在该文件，存在即跳过任务，避免重复执行
```

