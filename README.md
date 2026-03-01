# Домашнее задание к занятию "`Ansible часть 2`" - `Гуреев Юрий`

### Задание 1

Для выполнения некоторых playbook необходимо повышение прав до root, для этого необходимо добавить в файл `/etc/ansible/host` следующие параметры: 
`ansible_user=<username> ansible_become_pass=<user_pass>` 
с указанием имени пользователя и пароля с правими повышения до root для каждого хоста;
в домашнем задание используются группа хостов `servers`

Так же необходимо в playbook добавить следующие строчки:
```
  become: yes
  become_user: root
  become_method: sudo
``` 


#### 1 playbook 
Playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/1.playbook_1_download_and_unpack.yml
Содержимое файла:
```
---
- name: Download and unpack Apache Kafka
  hosts: servers
  become: yes
  become_user: root
  become_method: sudo
  vars:
    download_url: "https://dlcdn.apache.org/kafka/4.1.1/kafka-4.1.1-src.tgz"
    destination_dir: "/opt/kafka"
    archive_name: "kafka.tgz"

  tasks:
    - name: Create directory
      ansible.builtin.file:
        path: "{{ destination_dir }}"
        state: directory
        mode: '0755'

    - name: Download
      ansible.builtin.get_url:
        url: "{{ download_url }}"
        dest: "/tmp/{{ archive_name }}"
        mode: '0644'
        timeout: 30

    - name: Extract archive
      ansible.builtin.unarchive:
        src: "/tmp/{{ archive_name }}"
        dest: "{{ destination_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]
        creates: "{{ destination_dir }}/bin/kafka-server-start.sh"

    - name: Clean up temporary archive
      ansible.builtin.file:
        path: "/tmp/{{ archive_name }}"
        state: absent
```

Вывод работы playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/1.log_playbook_1_download_and_unpack.txt
Содержимое файла:
```
PLAY [Download and unpack Apache Kafka] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [Create directory] ********************************************************
changed: [10.100.0.125]
changed: [10.100.0.124]

TASK [Download] ****************************************************************
changed: [10.100.0.125]
changed: [10.100.0.124]

TASK [Extract archive] *********************************************************
changed: [10.100.0.124]
changed: [10.100.0.125]

TASK [Clean up temporary archive] **********************************************
changed: [10.100.0.125]
changed: [10.100.0.124]

PLAY RECAP *********************************************************************
10.100.0.124               : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.100.0.125               : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```


#### 2 playbook 
Playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/2.playbook_2_tuned_install.yml
Содержимое файла:
```
---
- name: Deploy tuned
  hosts: servers
  become: yes
  become_user: root
  become_method: sudo

  tasks:
    - name: Install tuned
      ansible.builtin.package:
        name: tuned
        state: present

    - name: Enabled and running tuned service
      ansible.builtin.systemd:
        name: tuned
        state: started
        enabled: yes
        daemon_reload: yes
```

Вывод работы playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/2.log_playbook_2_tuned_install.txt
Содержимое файла:
```
PLAY [Deploy tuned] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [Install tuned] ***********************************************************
changed: [10.100.0.125]
changed: [10.100.0.124]

TASK [Enabled and running tuned service] ***************************************
ok: [10.100.0.124]
ok: [10.100.0.125]

PLAY RECAP *********************************************************************
10.100.0.124               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.100.0.125               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```


#### 3 playbook 
Playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/3.playbook_3_motd.yml
Содержимое файла:
```
---
- name: Change MOTD
  hosts: servers
  become: yes
  become_user: root
  become_method: sudo
  vars:
    custom_motd: "Солнце в зените, хорошего дня!"

  tasks:
    - name: Custom message
      ansible.builtin.copy:
        content: "{{ custom_motd }}\n"
        dest: /etc/motd
        mode: '0644'
```

Вывод работы playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/3.log_playbook_3_motd.txt
Содержимое файла:
```
PLAY [Change MOTD] *************************************************************

TASK [Gathering Facts] *********************************************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [Custom message] **********************************************************
changed: [10.100.0.124]
changed: [10.100.0.125]

PLAY RECAP *********************************************************************
10.100.0.124               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.100.0.125               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```


### Задание 2

Playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/4.playbook_4_motd_update.yml
Содержимое файла:
```
---
- name: Change MOTD
  hosts: servers
  become: yes
  become_user: root
  become_method: sudo
  vars:
    greet_message: "Солнце в зените, хорошего дня!"

  tasks:
    - name: Get facts
      ansible.builtin.setup:
        gather_subset:
          - all

    - name: Set IP address
      ansible.builtin.set_fact:
        management_ip: >-
          {% if ansible_default_ipv4.address is defined %}
          {{ ansible_default_ipv4.address }}
          {% elif ansible_all_ipv4_addresses | length > 0 %}
          {{ ansible_all_ipv4_addresses[0] }}
          {% else %}
          "IP address not available"
          {% endif %}

    - name: Create message
      ansible.builtin.copy:
        content: |
          |==============================================================|
          |                      WELCOME MESSAGE                         |
          |==============================================================|
          |  Hostname: {{ ansible_fqdn }} ({{ ansible_hostname }})       
          |  IP Address: {{ management_ip }}                             
          |  {{ greet_message }}                                         
          |==============================================================|
          |  System Details:                                             
          |  - OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          |  - Kernel: {{ ansible_kernel }}                              
          |  - Architecture: {{ ansible_architecture }}                  
          |  - CPUs: {{ ansible_processor_cores }} cores                 
          |  - Memory: {{ (ansible_memtotal_mb / 1024) | round(2) }} GB  
          |==============================================================|
        dest: /etc/motd
        mode: '0644'
        backup: yes
```

Вывод работы playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/4.log_playbook_4_motd_update.txt
Содержимое файла:
```
PLAY [Change MOTD] *************************************************************

TASK [Gathering Facts] *********************************************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [Set IP address] **********************************************************
ok: [10.100.0.124]
ok: [10.100.0.125]

TASK [Create message] **********************************************************
ok: [10.100.0.124]
ok: [10.100.0.125]

PLAY RECAP *********************************************************************
10.100.0.124               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.100.0.125               : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```


### Задание 3

Роль доступна по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/apache_role/

Playbook для запуска доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/5.playbook_apache.yml
Содержимое файла:
```
---
- name: Install Apache and configure custom index.html
  hosts: servers
  become: yes
  roles:
    - apache_role
```

Вывод работы playbook доступен по ссылке https://github.com/mhfhfmn/Ansible_2/blob/main/5.log_playbook_apache.txt
Содержимое файла:
```
PLAY [Install Apache and configure custom index.html] **************************

TASK [Gathering Facts] *********************************************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [apache_role : Install Apache] ********************************************
changed: [10.100.0.124]
changed: [10.100.0.125]

TASK [apache_role : Create index.html] *****************************************
changed: [10.100.0.125]
changed: [10.100.0.124]

TASK [apache_role : Started and enabled Apache] ********************************
ok: [10.100.0.125]
ok: [10.100.0.124]

TASK [apache_role : Open port 80 (ufw)] ****************************************
skipping: [10.100.0.125]
skipping: [10.100.0.124]

TASK [apache_role : Check web server] ******************************************
ok: [10.100.0.125 -> localhost]
ok: [10.100.0.124 -> localhost]

RUNNING HANDLER [apache_role : restart apache] *********************************
changed: [10.100.0.125]
changed: [10.100.0.124]

PLAY RECAP *********************************************************************
10.100.0.124               : ok=6    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.100.0.125               : ok=6    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```

Web страница с результатом работы playbook:
![Web страница](https://github.com/mhfhfmn/Ansible_2/blob/main/img/5.screen_web_apache.jpg)