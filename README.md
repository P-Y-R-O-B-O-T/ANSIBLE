# ANSIBLE
* Automation
* Configuration
* Sequential steps
* Deployment
* Management of complex infrastructure

## INVENTORY
* Uses ssh and it is agentless
* Inventory files are used for defining target machines
* they are generally located at `/etc/ansible/hosts`
* Each target has 5 parameters:
    - ansible_host: how to connect to machine ssh or winrm, fuck winrm
    - ansible_port: which port to connect to on target, default is 22 
    - ansible_user: the user that makes the connections to the target
    - ansible_ssh_pass: pass for the machines
* Ansible allows to have parent child relationships between server groups

## TEMPLATES
* Ansible uses jinja 2 templates
* A shaped piece of object which is customizable
* Examples
- Set Ops
```
{{ [1,2,3] | max }}
{{ [1,2,3] | min }}
{{ [1,2,3,3] | unique }}
{{ [1,2,3,4] | union( [7,8] ) }}
{{ [1,2,3,4] | intersect( [4,5] ) }}
{{ 100 | random }}
{{ ["1","2","3"] | join(" ") }}
```
- Conditions and Loops
```
{% for number in [0,1,2,3,4] %}
{{ number }}
{% endfor %}

{% for number in [0,1,2,3,4] %}
    {% if number == 2 %}
        {{ number }}
    {% endif %}
{% endfor %}
```

## ANSIBLE CONFIG FILES
* This files is located at `/etc/ansible/ansible.cfg`
* If working on complex project which contains (`web-playbook`, `db-playbooks` and `network-playbooks`), can create copies of ansible playbooks at (`/opt/web-playbook/ansible.cfg`, `/opt/db-playbooks/ansible.cfg` and `/opt/network-playbooks/ansible.cfg`) and then modify them
* To use these configs we can specify the path of congif files using environment variables `export ANSIBLE_CONFIG=path/to/config/file` and then run the playbook
* The priority for ansible config file picking up are in the following order
    - Environment variable
    - /opt/path
    - ~/.ansible.cfg
    - /etc/ansible/ansible.cfg
* We can also change the ansible behaviour by setting the config variables as environment variables `export ANSIBLE_VAR=VAL` before running the command
* `ansible-config list` to see configuration parameters or list all configurations
* `ansible-config view` to see current config file
* `ansible-config dump` to see current settings

## VARIABLES AND FACTS
* We can define variables in seperate variables file and also in inventory file
* Types of variables:
    - String
    - Bool
    - List
    - Dictionaries
```
- name: zzz
  hosts: localhost
  vars:
    dns_ip: 127.0.0.1 
  tasks:
    - name: zzz
      shell: "ping {{ dns_server }}"
```
* We can also load variables from another file called variable file
    - The variable file searched in `/var` relative to role or to the playbook location
```
./playbook.yaml
- name: zzz
  hosts: localhost
  vars_files:
    - var_file_name.yaml
  tasks:
    - name: zzz
      shell: "cp {{ path1 }} {{path2}}"

./vars/var_file_name.yaml
path1: val1
path2: val2
```

* We can also store variables in inventory file
```
/etc/ansible/hosts
w1 ansible_host=1.1.1.1
w2 ansible_host=1.1.1.2 dns_server=1.1.1.1
w3 ansible_host=1.1.1.3

[web_servers]
w1
w2
w3

[web_servers:vars]
dns_server=8.8.8.8
```

* There is a scope for variables in playbook (play scope, global scope, task scope)
```
- name: zzz1
  hosts: zzz1
  vars:
    zzzv: zzzval
  tasks:
    - name: zzz1
      debug:
       var: zzzval
    - name: zzz12
      vars:
        zzzv: zzzval12
      debug:
        var: zzzv

- name: zzz2
  hosts: zzz2
  tasks:
    - name: zzz2
      debug:
       var: zzzval
```

* Variable precedence `extra variables` > `playbook variables` > `host variables` > `group variables`. See more at ansible docs
> Extra variables are passed through command line `ansible-playbook PLAYBOOK_PATH --extra-vars "VAR=VAL"`
* We can register variables by `register: variable_name`
* These registered variables are within the target host scope
* Variable defined for one host isnt accessable to another host
* We can also print the ragistered variables by
```
- debug:
    vars: registered_variable.attribute
```

## MAGIC VARIABLES
* Helps to get info and state of other hosts
```
/path/to/variables_inventory
web1 ansible_host=1.1.1.1
web2 ansible_host=1.1.1.2 dns_server=1.1.1.1
web3 ansible_host=1.1.1.3
v1:
  - v11: val11
  - v12: val12
v2:
  - val21
  - val22

/path/to/playbook
---
- name: zzz
  hosts: all
  tasks:
    - msg: "{{ hostvars['web2'].dns_server }} {{ hostvars['web2'].ansible_host }}{{ hostvars['web2'].ansible_facts.architecture }}{{ hostvars['web2'].ansible_facts.devices }}{{ hostvars['web2'].ansible_facts.mounts }}{{ group_names }}"
```

## ANSIBLE FACTS
* The data gathered about hosts when we run playbooks
* These are gathered by setup module
* All facts stored in ansible_facts variable
```
---
- name: zzz
  hosts: all
  tasks:
    - debug:
        var: ansible_facts
```
* We can also disble gathering facts by
```
---
- name: zzz
  hosts: all
  gather_facts: no
  tasks:
    - name: zzz
      debug:
        var: ansible_facts
```

## ANSIBLE PLAYBOOKS
* Ansible Playbooks are lists of tasks that automatically execute for your specified inventory or groups of hostvars
* They are defined as a yaml file, we can have multiple plays in a playbook
```
-
  name: zzz1
  hosts: host_grp_1
  tasks:
    - name: zzz11
      command: date
    - name: zzz12
      script: path/to/script

-
  name: zzz2
  hosts: host_grp_2
  tasks:
    - name: zzz21
      command: date
    - name: zzz22
      script: path/to/script
```

* **Verification**
    - `ansible-playbook PLAYBOOK_PATH --check-mode`: does not mak changes in target host but tells what it will actually do, all modules does not support check-mode they skip the checking
    - `ansible-playbook PLAYBOOK_PATH --diff`: gives a comparison bet before and after states of target host that will occur if we run playbook
    - `ansible-playbook PLAYBOOK_PATH --syntax-check`: does syntax checking
* We can check for linting and indentation and formatting issues by `ansible-lint PLAYBOOK_PATH`
* We can do conditional checks in playbook to perform an action only if a condition matches
```
conditional check for ansible checks for package managers
```
```
conditional loops
```
```
mailing conditional
```
```
environment variable set dev, test, prod
```
## MODULES
* Different actions run by tasks are called modules
* There are different types of modules: system, file, database, cloud, commands and many more
* inline file module can be used to append a line in a file

## PLUGINS
* Extent and customize ansible functionality
* They can enhance inventory, modules, Callbacks
* A plugin can be in form of inventory plugin, action plugin, module plugin, callback plugin
* Example is realtime infrastructure monitoring using access key
* Types of plugins:
    - Lookup plugins: fetch data from external sources like databases
    - Filter plugins: data manipulation and transformation
    - Connection plugins: allow ansible to connect to different types of hosts
    - Invertory plugins: get inventory info from cloud providers and many more
    - Callback plugins: manage hooka and capture events and also allow to run custom actions for them
* There are also cisco routers modules and plugins too `cisco.ios`

## HANDLERS
* Handlers are triggered by events and are notified by a task
* They manage action based on system state and configuration changes
* We can define an action to restart a service or machine and associate it with a task that modifies the configuration, this creates a dependency between the task and handler
* This improves infrastructure Management and effeciency to do a task
```
- name:
  hosts: zzz
  tasks:
    - name: zzz1
      copy:
        src: qqq
        dest: rrr
      notify: q_w_e

  handlers:
    - name: q_w_e
      service:
        name: a_a_a
        state: restarted
```

## ANSIBLE ROLES
* It is like assigning a role in life, a role does all things that are supposed to be done by thet role
* Roles are also more distributable, replicable, reusable
* We can create an empty role by `ansible-galaxy init ROLE_NAME`, then we can edit the files to create one specific one
```
./ROLE_NAME
│ 
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

# dafaults/main.yaml contains the default variable values
# handlers/main.yaml contains the handlers
# tasks/main.yaml contains tasks
# var/main.yaml contains varibbles that need to be defined for role to work
```
* TO USE THIS ROLE WE HAVE TO MOVE THE `ROLE_NAME` DIRECTORY TO THE `roles/` DIRECTORY AS THE PLAYBOOK THAT USES THIS ROLE
* `roles` directory is present in the same directory as the playbook using the roles
* Installed roles are present in `/etc/ansible/roles/`
* To install a role use `ansible galaxy-search ROLE_NAME` and `ansible-galaxy install ROLE_NAME`
* List roles `ansible-galaxy list`
* To see where roles are installed and see other config `ansible-config dump | grep ROLE`
* To install role at a particilar location `ansible-galaxy install ROLE_NAME -p PATH_TO_INSTALL_ROLE`
* A playbook using a role looks like:
```
- name: zzz
  hosts: a_a_a
  roles:
    - ROLE_NAME_1
    - ROLE_NAME_2
    - role: ROLE_NAME_3
      become: true
      become_user: root
      vars:
        role_variable: new_value
```

## ANSIBLE COLLECTIONS
* Ansible collections is a way to package and ship ansible content like role, plugin, module and many more
* collection is a self contained entity and encapsulates components
* Ansible provides builtin modules for notwork automation `network.cisco` `notwork.juniper` `network.arista` collections
* Install a collection `ansible-galaxy collection install COLLECTION_NAME` (network.cisco)
* Advantages: expanded functionality, modularity and reuasbility, simplified shipping

* ansible

```
ansible-galaxy collection install amazon.aws

./playbook.yaml
- name: zzz
  collections:
    - amazon.aws
  tasks:
    - name: creat a aws bucket
      aws_s3_bucket:
        name: zzz
        region: us-west-1
```
