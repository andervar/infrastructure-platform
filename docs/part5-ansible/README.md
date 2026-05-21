# Project, Part V - Ansible

## Cover Page

### University of Costa Rica

* **Infrastructure Design and Operation CI-0144**
* **Project, Part V - Ansible**
* **Professor:** Ariel Mora Jiménez
* **Student:** Anderson Vargas Navarro - C28183
* **1st semester, 2026**
* **Group:** 002

---

## **Project, Part V - Ansible**

Using the Infrastructure as Code tool Ansible, describe and manage your infrastructure. This means that through Ansible you should be able to provision your entire infrastructure in case an event compromises the continuity of your services and it becomes necessary to deploy everything from scratch. Also, you must consider that Ansible has the ability to expand your infrastructure.

You can consider implementing a DNS service or some other mechanism to identify the VMs of your infrastructure by name.

For this, you must use at least the following components that Ansible provides:

* Hosts Vars and/or Group Vars
* Modules
* Playbooks
* Tasks
* Roles
* Collections

It is valid to use Roles and Collections that are available from Ansible Galaxy.

For this project, the following additional aspects will be graded:

* Planning and design for Ansible
* Implementation decisions made
* Security measures applied

---

## Implementation

### Node Preparation

#### Create `ansible` user in the vms `nat`, `proxy`, `pods` and `redis`

1. Execute this inside each vm (all), create `ansible` user and group.

    ```sh
    sudo groupadd ansible
    sudo useradd -m -s /bin/bash -g ansible ansible
    id ansible
    ```

    * Output.

    ```sh
    uid=1001(ansible) gid=1002(ansible) groups=1002(ansible)
    ```

    * Configure sudo without password ONLY for the `ansible` group.

    ```sh
    echo '%ansible ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/ansible
    sudo chmod 440 /etc/sudoers.d/ansible
    sudo visudo -c
    ```

    * Output.

    ```sh
    /etc/sudoers: parsed OK
    /etc/sudoers.d/ansible: parsed OK
    ```

---

#### SSH Configuration and Keys

1. Prepare SSH for the `ansible` user (on each node).

    ```sh
    sudo install -d -m 700 -o ansible -g ansible /home/ansible/.ssh
    ```

    * **`sudo install -d -m 700 -o ansible -g ansible /home/ansible/.ssh`**: Uses the `install` utility to perform an atomic preparation of the SSH environment. The command creates the configuration directory (`-d`) while simultaneously ensuring that permissions are restrictive (`-m 700`), which is a strict SSH requirement for ignoring unauthorized access. Additionally, it assigns ownership of the directory to the user (`-o ansible`) and the group (`-g ansible`) in a single operation, eliminating the need to run additional commands like `mkdir`, `chmod` and `chown` separately.

2. Inside the `nat` vm generate the key (only once).

    ```sh
    ssh-keygen -t ed25519 -f ~/.ssh/ansible_id -N ""
    ```

    * Copy the public key to the nodes.

    ```sh
    cat ~/.ssh/ansible_id.pub
    ```

    * On each node (`proxy`, `pods`, `redis`).

    ```sh
    echo "<PUBLIC_KEY_VM_NAT>" | sudo tee /home/ansible/.ssh/authorized_keys > /dev/null

    sudo chown ansible:ansible /home/ansible/.ssh/authorized_keys
    sudo chmod 600 /home/ansible/.ssh/authorized_keys
    ```

3. Lock and delete the `ansible` user password login (on each node).

    ```sh
    sudo passwd -d ansible && sudo passwd -l ansible
    ```

4. Perform SSH hardening (on each node).

    ```sh
    sudo nano /etc/ssh/sshd_config
    ```

    * Edit.

    ```ini
    AllowUsers admin ansible
    PasswordAuthentication no
    PermitRootLogin no
    PubkeyAuthentication yes
    ```

    * Apply

    ```sh
    sudo systemctl restart sshd
    ```

    * Verify with the command `sudo sshd -T | grep -E 'allowusers|passwordauthentication|permitrootlogin|pubkeyauthentication'`.

    ```sh
    permitrootlogin no
    pubkeyauthentication yes
    passwordauthentication no
    allowusers admin
    allowusers ansible
    ```

5. Make SSH configuration in the `nat` vm.

    ```sh
    nano ~/.ssh/config
    ```

    * Add

    ```sh
    Host pods-ansible
        HostName 192.168.216.8
        User ansible
        IdentityFile ~/.ssh/ansible_id
    Host proxy-ansible
        HostName 192.168.216.4
        User ansible
        IdentityFile ~/.ssh/ansible_id
    Host redis-ansible
        HostName 192.168.216.9
        User ansible
        IdentityFile ~/.ssh/ansible_id
    ```

6. Verify from the control node `nat`.

    ```sh
    ssh proxy-ansible "hostname && id && sudo whoami"
    ssh pods-ansible  "hostname && id && sudo whoami"
    ssh redis-ansible "hostname && id && sudo whoami"
    ```

    * Output.

    ```sh
    Unauthorized access prohibited. All activity is monitored and logged.
    proxy
    uid=1001(ansible) gid=1002(ansible) groups=1002(ansible) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
    root
    Unauthorized access prohibited. All activity is monitored and logged.
    pods
    uid=1001(ansible) gid=1002(ansible) groups=1002(ansible) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
    root
    Unauthorized access prohibited. All activity is monitored and logged.
    redis
    uid=1001(ansible) gid=1002(ansible) groups=1002(ansible) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
    root
    ```

---

#### SSH Connection to `nat`

1. Add ansible ssh key.

    ```sh
    cat ~/.ssh/ansible_id.pub | sudo tee /home/ansible/.ssh/authorized_keys
    ```

    * Change permissions.

    ```sh
    sudo chown ansible:ansible /home/ansible/.ssh/authorized_keys
    sudo chmod 600 /home/ansible/.ssh/authorized_keys
    ```

    * Verify.

    ```sh
    ssh -i ~/.ssh/ansible_id ansible@localhost "hostname && id && sudo whoami"
    ```

---

### Ansible Base Installation and Configuration

#### Ansible Installation

Following the installation guide: [Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#ensuring-pip-is-available)

1. Install dependencies (pip) on the `nat` vm.

    ```sh
    sudo dnf install python3-pip -y
    ```

    * Check.

    ```sh
    python3 -m pip -V
    ```

    * Output.

    ```sh
    pip 26.0.1 from /home/admin/.local/lib/python3.9/site-packages/pip (python 3.9)
    ```

2. Install Ansible.

    ```sh
    python3 -m pip install --user ansible
    ```

    * Verify installation.

    ```sh
    ansible --version
    ```

    * Output.

    ```sh
    ansible [core 2.15.13]
      config file = None
      configured module search path = ['/home/admin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
      ansible python module location = /home/admin/.local/lib/python3.9/site-packages/ansible
      ansible collection location = /home/admin/.ansible/collections:/usr/share/ansible/collections
      executable location = /home/admin/.local/bin/ansible
      python version = 3.9.21 (main, Jun 27 2025, 00:00:00) [GCC 11.5.0 20240719 (Red Hat 11.5.0-5)] (/usr/bin/python3)
      jinja version = 3.1.6
      libyaml = True
    ```

3. Install the `ansible.posix` collection.

    ```sh
    ansible-galaxy collection install ansible.posix
    ```

    * This collection includes `ansible.posix.firewalld` and `ansible.posix.selinux`, which helps with task management.

4. Create initial project structure.

```sh
mkdir -p ~/ansible-project/{inventory/{group_vars,host_vars},playbooks,roles}
```

---

#### Configure `ansible.cfg` and inventory

1. Create `ansible.cfg` file in the root of the project.

    ```sh
    nano ~/ansible-project/ansible.cfg
    ```

    * Add: [ansible_cfg](../../ansible-project/ansible.cfg)

    * Justification:
      * **`inventory = inventory/hosts.yaml`**: Centralizes the management of nodes in a specific file and uses YAML format, which is the modern Ansible standard for allowing clearer reading and better structure for complex variables than the traditional INI format.
      * **`remote_user = ansible`**: Standardizes the technical user with which Ansible will connect to all remote nodes, avoiding dependence on local users of administrators and guaranteeing environment homogeneity.
      * **`private_key_file = ~/.ssh/ansible_id`**: Enforces the use of a specific SSH key for automation without needing to declare it in the command line at each execution, which increases security and simplifies the workflow.
      * **`interpreter_python = auto_silent`**: Silences annoying warnings about Python interpreter detection on remote nodes and automatically selects the most appropriate version of the system, guaranteeing clean terminal executions.
      * **`stdout_callback = yaml`**: Transforms the standard terminal output (which by default is a dense JSON-like block) to a highly readable and structured YAML format, greatly facilitating the visual analysis of errors in real time.
      * **`gathering = smart`**, **`fact_caching = jsonfile`**, **`fact_caching_connection = ...`** and **`fact_caching_timeout = 3600`**: Implement a persistent caching system for "facts" (data collected from nodes). This drastically reduces the execution times of repetitive Playbooks, as Ansible will not need to query the hardware of the machines at each execution for an hour.
      * **`roles_path = roles/`**: Keeps the project structure clean and modular, explicitly indicating to the Ansible engine where to search for and load the packaged logic of the roles.
      * **`log_path = ansible.log`**: Records the complete and chronological history of all Playbook executions. This guarantees system traceability and auditing; its integration with the native Linux tool *logrotate* ensures that storage does not saturate over time.
      * **`become=true`**: Activated globally for convenience for this project, as the vast majority of initial tasks (package installation, service configuration and system file manipulation) require elevated privileges. *(Note: Should be deactivated punctually with `become: false` in Rootless Podman tasks).*
      * **`become_method=sudo`**, **`become_user=root`** and **`become_ask_pass=false`**: Define the exact mechanics of privilege escalation. They automate the transformation to `root` user through `sudo` without requiring human intervention to enter passwords, which is vital for unattended deployments.
      * **`pipelining = true`**: Is one of Ansible's most aggressive and effective network optimizations. It drastically reduces the number of SSH connections needed to execute Python modules on remote nodes, accelerating the total deployment time by up to 50%.

2. Configure log rotation with `logrotate`.

    ```sh
    sudo nano /etc/logrotate.d/ansible
    ```

    * Add.

    ```sh
    /home/admin/ansible-project/ansible.log {
        weekly
        rotate 4
        compress
        delaycompress
        missingok
        notifempty
        create 0640 admin admin
    }
    ```

    * Justification:
      * **`weekly`**: rotates the log once a week. For an infrastructure of 4 nodes it is sufficient; `daily` would be excessive.
      * **`rotate 4`**: keeps 4 weeks of history before deleting. Sufficient for auditing without accumulating disk.
      * **`compress`**: compresses rotated logs with gzip. A text log compresses to 10-20% of its original size.
      * **`delaycompress`**: does not compress the log of the immediate previous rotation. Useful if you need to read the recent log without decompressing.
      * **`missingok`**: does not fail if `ansible.log` does not exist yet (for example before the first run).
      * **`notifempty`** — does not rotate if the log is empty, avoids useless files.
      * **`create 0640 admin admin`**: creates the new empty `ansible.log` after rotating, with correct permissions so Ansible can write.
    * Verify that the syntax is correct:

    ```sh
    sudo logrotate --debug /etc/logrotate.d/ansible
    ```

    > *Note:* Add `ansible.log` to `.gitignore`.

3. Create `inventory/hosts.yaml` file. Reference: [Inventory Guide](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html).

    ```sh
    nano ~/ansible-project/inventory/hosts.yaml
    ```

    * Add: [inventory_hosts](../../ansible-project/inventory/hosts.yaml)

4. Verify connection with the inventory nodes.

    ```sh
    cd ~/ansible-project
    ansible all -m ping
    ```

    * Output.

    ```sh
    proxy | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
    pods | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
    nat | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }
    redis | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
    ```

---

#### Global Variables (`group_vars`) and Sensitive (`vault.yaml`)

```txt
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
│   ├── group_vars/
│   │   └── all/
│   │     ├── all.yaml
│   │     └── vault.yaml (encrypted)
│   └── hosts.yaml
│   └── host_vars/
├── playbooks/
└── roles/
```

1. Creation of `all.yaml` file in groups.

    ```sh
    nano ~/ansible-project/inventory/group_vars/all.yaml
    ```

    * Add: [group_vars_all](../../ansible-project/inventory/group_vars/all.yaml)

    * Verify.

    ```sh
    ansible all -m debug -a "var=dns_servers"
    ```

    * Output.

    ```sh
    nat | SUCCESS => {
        "dns_servers": [
            "163.178.88.2",
            "163.178.88.4"
        ]
    }
    redis | SUCCESS => {
        "dns_servers": [
            "163.178.88.2",
            "163.178.88.4"
        ]
    }
    proxy | SUCCESS => {
        "dns_servers": [
            "163.178.88.2",
            "163.178.88.4"
        ]
    }
    pods | SUCCESS => {
        "dns_servers": [
            "163.178.88.2",
            "163.178.88.4"
        ]
    }
    ```

2. Using *Ansible Vault* create an encrypted file to store sensitive data. Reference: [Ansible Vault](https://medium.com/@vinoji2005/day-23-mastering-ansible-vault-for-secure-automation-73d44937ab9b)

    ```sh
    ansible-vault create ~/ansible-project/inventory/group_vars/all/vault.yaml
    ```

    * Add:

    ```ini
    # Databases Passwords
    vault_mariadb_root_password: "<PASSWORD_ROOT>"
    vault_mariadb_glpi_password: "<PASSWORD_GLPI>"
    vault_postgres_wikijs_password: "<PASSWORD_WIKIJS>"

    # Redis Cache Password
    vault_redis_password: "<PASSWORD_REDIS>"
    ```

    * View the contents of `vault.yaml`

    ```sh
    ansible-vault view ~/ansible-project/inventory/group_vars/all/vault.yaml
    ```

    > To run *playbooks* that use encrypted files, you must use `ansible-playbook site.yml --ask-vault-pass`

---

#### Variables per Host (`host_vars`)

```sh
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
│   ├── group_vars/
│   └── hosts.yaml
│   └── host_vars/
│       ├── nat.yaml
│       ├── proxy.yaml
│       ├── pods.yaml
│       └── redis.yaml
├── playbooks/
└── roles/
```

1. Create `host_vars/nat.yaml`.

    ```sh
    nano ~/ansible-project/inventory/host_vars/nat.yaml
    ```

    * Add: [host_vars_nat](../../ansible-project/inventory/host_vars/nat.yaml)

2. Create `host_vars/proxy.yaml`.

    ```sh
    nano ~/ansible-project/inventory/host_vars/proxy.yaml
    ```

    * Add: [host_vars_proxy](../../ansible-project/inventory/host_vars/proxy.yaml)

3. Create `host_vars/pods.yaml`.

    ```sh
    nano ~/ansible-project/inventory/host_vars/pods.yaml
    ```

    * Add: [host_vars_pods](../../ansible-project/inventory/host_vars/pods.yaml)

4. Create `host_vars/redis.yaml`.

    ```sh
    nano ~/ansible-project/inventory/host_vars/redis.yaml
    ```

    * Add: [host_vars_redis](../../ansible-project/inventory/host_vars/redis.yaml)

---

### Creation of the `Common` Role

#### Snapshots before executing *playbooks*

1. Perform Snapshot on the `nat` vm.
![[Pasted image 20260428201631.png]]

2. Perform Snapshot on the `proxy` vm.
![[Pasted image 20260428201451.png]]

3. Perform Snapshot on the `pods` vm.
![[Pasted image 20260428201525.png]]

4. Perform Snapshot on the `redis` vm.
![[Pasted image 20260428201608.png]]

---

#### Creation of the structure and files of the `Common` role

```sh
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
├── playbooks/
└── roles/
    ├── common/
        ├── tasks/
        │   ├── firewall.yaml
        │   ├── hosts.yaml
        │   ├── main.yaml
        │   ├── nat.yaml
        │   ├── selinux.yaml
        │   └── timezone.yaml
        └── handlers/
            └── main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/common/{tasks,handlers}
    ```

2. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/handlers/main.yaml
    ```

    * Add: [common_handler_main](../../ansible-project/roles/common/handler/main.yaml)

3. Create `tasks/selinux.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/selinux.yaml
    ```

    * Add: [common_tasks_selinux](../../ansible-project/roles/common/tasks/selinux.yaml)

4. Create `tasks/hosts.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/hosts.yaml
    ```

    * Add: [common_tasks_hosts](../../ansible-project/roles/common/tasks/hosts.yaml)

5. Create `tasks/timezone.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/timezone.yaml
    ```

    * Add: [common_tasks_timezone](../../ansible-project/roles/common/tasks/timezone.yaml)

6. Create `tasks/firewall.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/firewall.yaml
    ```

    * Add: [common_tasks_firewall](../../ansible-project/roles/common/tasks/firewall.yaml)

7. Create `tasks/nat.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/nat.yaml
    ```

    * Add: [common_tasks_nat](../../ansible-project/roles/common/tasks/nat.yaml)

8. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/common/tasks/main.yaml
    ```

    * Add: [common_tasks_main](../../ansible-project/roles/common/tasks/main.yaml)

---

#### Verification of the `Common` Role

##### Syntax and Variables Verification of the `Common` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_common.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test Common Role
      hosts: all
      gather_facts: true
      roles:
        - common
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_common.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_common.yaml
    ```

3. Verify that variables load correctly

    ```bash
    # Global variables (all hosts)
    ansible all -m debug -a "var=dns_servers"

    # Firewall variables for each node
    ansible all -m debug -a "var=firewall_zones"

    # Variables specific to nat
    ansible nat -m debug -a "var=nat_policy" --ask-vault-pass
    ansible nat -m debug -a "var=ip_forward"
    ansible nat -m debug -a "var=firewall_zones" --ask-vault-pass
    ansible nat -m debug -a "var=hosts_entries"

    # Variables specific to proxy
    ansible proxy -m debug -a "var=proxy_interfaces"
    ansible proxy -m debug -a "var=backend_servers"
    ansible proxy -m debug -a "var=haproxy"

    # Variables specific to pods
    ansible pods -m debug -a "var=pods_interfaces"
    ansible pods -m debug -a "var=glpi_image"
    ansible pods -m debug -a "var=podman_user"

    # Variables specific to redis
    ansible redis -m debug -a "var=redis_interfaces"
    ansible redis -m debug -a "var=redis"
    ```

---

##### Dry Run of the `Common` Role

1. Dry run on `nat`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --check --diff \
      --limit nat \
      --ask-vault-pass
    ```

    * Output

    ```yaml
    nat: ok=11 changed=0 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0  
    ```

2. Dry run on `pods`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --check --diff \
      --limit pods \
      --ask-vault-pass
    ```

    * Errors obtained:
      * *INVALID_ZONE*: Occurs because the zone to which parameters are being assigned does not exist, using as reference to improve the firewall task: [Ansible Firewalld and adding new zone](https://stackoverflow.com/questions/42293872/ansible-firewalld-and-adding-new-zone)
      * In addition to this, a `flush_handlers` interruption is added for greater security.
      * Changes are detected in the task *Assign interfaces to zones*, but reviewing the variables, it is believed that when run in the actual execution it will disappear.

3. Dry run on `proxy`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --check --diff \
      --limit proxy \
      --ask-vault-pass
    ```

    * Changes are detected in the task *Assign interfaces to zones*, but reviewing the variables, it is believed that when run in the actual execution it will disappear.

4. Dry run on `redis`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --check --diff \
      --limit redis \
      --ask-vault-pass
    ```

    * Changes are detected in the task *Assign interfaces to zones*, but reviewing the variables, it is believed that when run in the actual execution it will disappear.

5. Dry run on all nodes

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    * Output.

    ```sh
    nat: ok=12 changed=0 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0   
    pods: ok=10 changed=2 unreachable=0 failed=0 skipped=12 rescued=0 ignored=0   
    proxy: ok=11 changed=2 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0   
    redis: ok=10 changed=2 unreachable=0 failed=0 skipped=12 rescued=0 ignored=0   
    ```

---

##### Execution of the `Common` Role

1. Actual execution on `nat`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --limit nat \
      --ask-vault-pass
    ```

    * Output (Same output in two runs).

    ```sh
    nat: ok=13 changed=0 unreachable=0 failed=0 skipped=8 rescued=0 ignored=0
    ```

2. Actual execution on `pods`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --limit pods \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    pods: ok=11 changed=2 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0
    ```

    * Output (Second run).

    ```sh
    pods: ok=10 changed=0 unreachable=0 failed=0 skipped=11   rescued=0 ignored=0   
    ```

3. Actual execution on `proxy`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --limit proxy \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    proxy: ok=12 changed=2 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0
    ```

    * Output (Second run).

    ```sh
    proxy: ok=11 changed=0 unreachable=0 failed=0 skipped=10   rescued=0 ignored=0 
    ```

4. Actual execution on `redis`.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --limit redis \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    redis: ok=11 changed=2 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0 
    ```

    * Output (Second run).

    ```sh
    redis: ok=10 changed=0 unreachable=0 failed=0 skipped=11   rescued=0 ignored=0  
    ```

5. Actual execution on all nodes.

    ```bash
    ansible-playbook playbooks/tests/test_common.yaml \
      --ask-vault-pass
    ```

* Output.

    ```sh
    nat: ok=13 changed=0 unreachable=0 failed=0 skipped=8    rescued=0 ignored=0   
    pods: ok=10 changed=0 unreachable=0 failed=0 skipped=11   rescued=0 ignored=0
    proxy: ok=11 changed=0 unreachable=0 failed=0 skipped=10   rescued=0 ignored=0
    redis: ok=10 changed=0 unreachable=0 failed=0 skipped=11   rescued=0 ignored=0  
    ```

* Summary of which tasks run on each host.

| Task | nat | proxy | pods | redis |
| ------------------------------ | --- | ----- | ---- | ----- |
| SELinux enforcing | X | X | X | X |
| Timezone | X | X | X | X |
| Assign interfaces to zones | X | X | X | X |
| Configure services per zone | X | X | X | X |
| Configure ports per zone | | X | | |
| Configure masquerade per zone | X | | | |
| Configure rich rules per zone | | X | X | X |
| Enable IP forwarding | X | | | |
| Verify NAT policy | X | | | |
| Create NAT policy | X | | | |
| Add /etc/hosts entries | X | | | |

---

### Creation of the `HAProxy` Role

#### Creation of the structure and files of the `HAProxy` role

```ini
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
│   ├── group_vars/
│   │   ├── all.yaml
│   │   └── vault.yaml (encrypted)
│   └── hosts.yaml
│   └── host_vars/
│       ├── nat.yaml
│       ├── proxy.yaml
│       ├── pods.yaml
│       └── redis.yaml
├── playbooks/
└── roles/
    ├── common/
    │   ├── tasks/
    │   │   ├── firewall.yaml
    │   │   ├── hosts.yaml
    │   │   ├── main.yaml
    │   │   ├── nat.yaml
    │   │   ├── selinux.yaml
    │   │   └── timezone.yaml
    │   └── handlers/
    │       └── main.yaml
    └── haproxy/
        ├── tasks/main.yaml
        ├── templates/
        │   ├── openssl.cnf.j2
        │   └── haproxy.cfg.j2
        └── handlers/main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/haproxy/{defaults,handlers,tasks,templates}
    ```

    Reference: [Ansible HA Proxy Configuration](https://oneuptime.com/blog/post/2026-02-21-ansible-configure-haproxy-load-balancer/view)

2. Create `defaults/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/haproxy/defaults/main.yaml
    ```

    * Add: [haproxy_defaults_main](../../ansible-project/roles/haproxy/defaults/main.yaml)

3. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/haproxy/handlers/main.yaml
    ```

    * Add: [haproxy_handlers_main](../../ansible-project/roles/haproxy/handlers/main.yaml)

4. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/haproxy/tasks/main.yaml
    ```

    * Add: [haproxy_tasks_main](../../ansible-project/roles/haproxy/tasks/main.yaml)

5. Create `templates/haproxy.cfg.j2`.

    ```sh
    nano ~/ansible-project/roles/haproxy/templates/haproxy.cfg.j2
    ```

    * Add: [haproxy_templates_haproxy_cfg](../../ansible-project/roles/haproxy/templates/haproxy.cfg.j2)

6. Create `templates/openssl.cfg.j2`.

    ```sh
    nano ~/ansible-project/roles/haproxy/templates/openssl.cnf.j2
    ```

    * Add: [haproxy_templates_openssl_cnf](../../ansible-project/roles/haproxy/templates/openssl.cnf.j2)

---

#### Verification of the `HAProxy` Role

##### Syntax Verification of the `HAProxy` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_haproxy.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test HAProxy Role
      hosts: proxy
      gather_facts: true
      roles:
        - haproxy
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_haproxy.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_haproxy.yaml
    ```

---

##### Dry Run of the `HAProxy` Role

1. Dry run of the `HAProxy` role.

    ```bash
    ansible-playbook playbooks/tests/test_haproxy.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    * Output.

    ```sh
    proxy: ok=12 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0  
    ```

    > Most changes are in the `haproxy.cfg` configuration file that change comments and formatting things, plus other changes in permissions in other files.

---

##### Execution of the `HAProxy` Role

1. Actual execution of the `HAProxy` role.

    ```bash
    ansible-playbook playbooks/tests/test_haproxy.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    proxy: ok=12 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0   
    ```

    * Output (Second run).

    ```sh
    proxy: ok=11 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
    ```

---

### Creation of the `Podman_Base` Role

#### Creation of the structure and files of the `Podman_Base` role

```ini
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
├── playbooks/
└── roles/
    ├── common/
    ├── haproxy/
    └── podman_base/
      ├── defaults/main.yaml
        ├── tasks/main.yaml
        └── handlers/
            └── main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/podman_base/{defaults,handlers,tasks}
    ```

2. Create `defaults/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/podman_base/defaults/main.yaml
    ```

    * Add: [podman_base_defaults_main](../../ansible-project/roles/podman_base/defaults/main.yaml)

3. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/podman_base/handlers/main.yaml
    ```

    * Add:[podman_base_handlers_main](../../ansible-project/roles/podman_base/handlers/main.yaml)

4. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/podman_base/tasks/main.yaml
    ```

    * Add: [podman_base_tasks_main](../../ansible-project/roles/podman_base/tasks/main.yaml)

---

#### Verification of the `Podman_Base` Role

##### Syntax Verification of the `Podman_Base` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_podman_base.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test Podman Base Role
      hosts: pods
      gather_facts: true
      roles:
        - podman_base
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_podman_base.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_podman_base.yaml
    ```

---

##### Dry Run of the `Podman_Base` Role

1. Dry run of the `Podman_Base` role.

    ```bash
    ansible-playbook playbooks/tests/test_podman_base.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    * Output.

    ```sh
    pods: ok=5 changed=2 unreachable=0 failed=0 skipped=8 rescued=0 ignored=0   
    ```

    > Changes are due to some packages being installed.

---

##### Execution of the `Podman_Base` Role

1. Actual execution of the `Podman_Base` role.

    ```bash
    ansible-playbook playbooks/tests/test_podman_base.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    pods: ok=12 changed=5 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0 
    ```

    * Output (Second run).

    ```sh
    pods: ok=10 changed=0 unreachable=0 failed=0 skipped=2 rescued=0 ignored=0   
    ```

---

### Creation of the `GLPI_pod` Role

#### Creation of the structure and files of the `GLPI_pod` role

```ini
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
├── playbooks/
└── roles/
    ├── common/
    ├── haproxy/
    ├── podman_base/
    └── glpi_pod/
        ├── defaults/
        │   └── main.yaml
        ├── tasks/
        │   └── main.yaml
        ├── templates/
        │   ├── glpi-pod.pod.j2
        │   ├── mariadb.container.j2
        │   ├── glpi.container.j2
        │   ├── glpi.env.j2
        │   └── mariadb.cnf.j2
        └── handlers/main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/glpi_pod/{defaults,handlers,tasks,templates}
    ```

2. Create `defaults/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/defaults/main.yaml
    ```

    * Add: [glpi_pod_defaults_main](../../ansible-project/roles/glpi_pod/defaults/main.yaml)

3. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/handlers/main.yaml
    ```

    * Add: [glpi_pod_handlers_main](../../ansible-project/roles/glpi_pod/handlers/main.yaml)

4. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/tasks/main.yaml
    ```

    * Add: [glpi_pod_tasks_main](../../ansible-project/roles/glpi_pod/tasks/main.yaml)

5. Create `templates/glpi-pod.pod.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/glpi-pod.pod.j2
    ```

    * Add: [glpi_pod_templates_glpi_pod](../../ansible-project/roles/glpi_pod/templates/glpi-pod.pod.j2)

6. Create `templates/mariadb.container.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/mariadb.container.j2
    ```

    * Add: [glpi_pod_templates_mariadb_container](../../ansible-project/roles/glpi_pod/templates/mariadb.container.j2)

7. Create `templates/glpi.container.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/glpi.container.j2
    ```

    * Add: [glpi_pod_templates_glpi_container](../../ansible-project/roles/glpi_pod/templates/glpi.container.j2)

8. Create `templates/mariadb.env.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/mariadb.env.j2
    ```

    * Add: [glpi_pod_templates_mariadb_env](../../ansible-project/roles/glpi_pod/templates/mariadb.env.j2)

9. Create `templates/glpi.env.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/glpi.env.j2
    ```

    * Add: [glpi_pod_templates_glpi_env](../../ansible-project/roles/glpi_pod/templates/glpi.env.j2)

10. Create `templates/mariadb.cnf.j2`.

    ```sh
    nano ~/ansible-project/roles/glpi_pod/templates/mariadb.cnf.j2
    ```

    * Add: [glpi_pod_templates_mariadb_cnf](../../ansible-project/roles/glpi_pod/templates/mariadb.cnf.j2)

---

#### Verification of the `GLPI_pod` Role

##### Syntax Verification of the `GLPI_pod` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_glpi_pod.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test GLPI pod Role
      hosts: pods
      gather_facts: true
      roles:
        - glpi_pod
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_glpi_pod.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_glpi_pod.yaml
    ```

---

##### Dry Run of the `GLPI_pod` Role

1. Dry run of the `GLPI_pod` role.

    ```bash
    ansible-playbook playbooks/tests/test_glpi_pod.yaml \
      --check --diff \
    ```

    * Output.

    ```sh
    pods: ok=10 changed=7 unreachable=0 failed=0 skipped=10   rescued=0 ignored=0
    ```

---

##### Execution of the `GLPI_pod` Role

1. Actual execution of the `GLPI_pod` role.

    ```bash
    ansible-playbook playbooks/tests/test_glpi_pod.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    pods: ok=17 changed=7 unreachable=0 failed=0 skipped=3    rescued=0 ignored=0   
    ```

    * Output (Second run).

    ```sh
    pods: ok=16 changed=0 unreachable=0 failed=0 skipped=3    rescued=0 ignored=0   
    ```

---

### Creation of the `WikiJS_pod` Role

#### Creation of the structure and files of the `WikiJS_pod` role

```ini
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
├── playbooks/
└── roles/
    ├── common/
    ├── haproxy/
    ├── podman_base/
    ├── glpi_pod/
    └── wikijs_pod/
        ├── defaults/
        │   └── main.yaml
        ├── tasks/main.yaml
        ├── templates/
        │   ├── wikijs-pod.pod.j2
        │   ├── postgres-wikijs.container.j2
        │   ├── wikijs.container.j2
        │   ├── wikijs.container.j2
        │   ├── postgres.env.j2
        │   └── wikijs.env.j2
        └── handlers/
            └── main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/wikijs_pod/{tasks,handlers,defaults,templates}
    ```

2. Create `defaults/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/defaults/main.yaml
    ```

    * Add: [wikijs_pod_defaults_main](../../ansible-project/roles/wikijs_pod/defaults/main.yaml)

3. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/handlers/main.yaml
    ```

    * Add: [wikijs_pod_handlers_main](../../ansible-project/roles/wikijs_pod/handlers/main.yaml)

4. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/tasks/main.yaml
    ```

    * Add: [wikijs_pod_tasks_main](../../ansible-project/roles/wikijs_pod/tasks/main.yaml)

5. Create `templates/postgres-wikijs.container.j2`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/templates/postgres-wikijs.container.j2
    ```

    * Add: [wikijs_pod_templates_postgres-wikijs_container](../../ansible-project/roles/wikijs_pod/templates/postgres-wikijs.container.j2)

6. Create `templates/postgres.env.j2`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/templates/postgres.env.j2
    ```

    * Add: [wikijs_pod_templates_postgres_env](../../ansible-project/roles/wikijs_pod/templates/postgres.env.j2)

7. Create `templates/wikijs-pod.pod.j2`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/templates/wikijs-pod.pod.j2
    ```

    * Add: [wikijs_pod_templates_wikijs-pod_pod](../../ansible-project/roles/wikijs_pod/templates/wikijs-pod.pod.j2)

8. Create `templates/wikijs.container.j2`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/templates/wikijs.container.j2
    ```

    * Add: [wikijs_pod_templates_wikijs_container](../../ansible-project/roles/wikijs_pod/templates/wikijs.container.j2)

9. Create `templates/wikijs.env.j2`.

    ```sh
    nano ~/ansible-project/roles/wikijs_pod/templates/wikijs.env.j2
    ```

    * Add: [wikijs_pod_templates_wikijs_env](../../ansible-project/roles/wikijs_pod/templates/wikijs.env.j2)

---

#### Verification of the `WikiJS_pod` Role

##### Syntax Verification of the `WikiJS_pod` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_wikijs_pod.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test WikiJS pod Role
      hosts: pods
      gather_facts: true
      roles:
        - wikijs_pod
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_wikijs_pod.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_wikijs_pod.yaml
    ```

---

##### Dry Run of the `WikiJS_pod` Role

1. Dry run of the `WikiJS_pod` role.

    ```bash
    ansible-playbook playbooks/tests/test_wikijs_pod.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    * Output.

    ```sh
    pods: ok=9 changed=6 unreachable=0 failed=0 skipped=4    rescued=0 ignored=0
    ```

---

##### Execution of the `WikiJS_pod` Role

1. Actual execution of the `WikiJS_pod` role.

    ```bash
    ansible-playbook playbooks/tests/test_wikijs_pod.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    pods: ok=12 changed=6 unreachable=0 failed=0 skipped=1    rescued=0 ignored=0  
    ```

    * Output (Second run).

    ```sh
    pods: ok=11 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0   
    ```

---

### Creation of the `Redis` Role

#### Creation of the structure and files of the `Redis` role

```ini
ansible-project/
├── ansible.cfg
├── ansible.log
├── inventory/
├── playbooks/
└── roles/
    ├── common/
    ├── haproxy/
    ├── podman_base/
    ├── glpi_pod/
    ├── wikijs_pod/
    └── redis/
        ├── defaults/main.yaml
        ├── tasks/main.yaml
        ├── templates/
        │   └── redis.conf.j2
        └── handlers/main.yaml
```

1. Create the directory structure.

    ```sh
    mkdir -p ~/ansible-project/roles/redis/{tasks,handlers,defaults,templates}
    ```

2. Create `defaults/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/redis/defaults/main.yaml
    ```

    * Add: [redis_defaults_main](../../ansible-project/roles/redis/defaults/main.yaml)

3. Create `handlers/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/redis/handlers/main.yaml
    ```

    * Add: [redis_handlers_main](../../ansible-project/roles/redis/handlers/main.yaml)

4. Create `tasks/main.yaml`.

    ```sh
    nano ~/ansible-project/roles/redis/tasks/main.yaml
    ```

    * Add: [redis_tasks_main](../../ansible-project/roles/redis/tasks/main.yaml)

5. Create `templates/redis.conf.j2`.

    ```sh
    nano ~/ansible-project/roles/redis/templates/redis.conf.j2
    ```

    * Add: [redis_templates_redis_conf](../../ansible-project/roles/redis/templates/redis.conf.j2)

---

#### Verification of the `Redis` Role

##### Syntax Verification of the `Redis` Role

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/tests/test_redis.yaml
    ```

    * Add

    ```ini
    ---
    - name: Test Redis Role
      hosts: redis
      gather_facts: true
      roles:
        - redis
    ```

2. Verify syntax

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/tests/test_redis.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/tests/test_redis.yaml
    ```

---

##### Dry Run of the `Redis` Role

1. Dry run of the `Redis` role.

    ```bash
    ansible-playbook playbooks/tests/test_redis.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    > Several errors occurred, but they were conditioned to only actual execution, in *dry run* mode they are avoided.

---

##### Execution of the `Redis` Role

1. Actual execution of the `Redis` role.

    ```bash
    ansible-playbook playbooks/tests/test_redis.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    redis: ok=13 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0   
    ```

    * Output (Second run).

    ```sh
    redis: ok=12   changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0   
    ```

---

### Creation of the `site.yaml` Playbook

1. Create test file.

    ```sh
    nano ~/ansible-project/playbooks/site.yaml
    ```

    * Add: [playbooks_site](../../ansible-project/playbooks/site.yaml)

---

#### Verification of the `site.yaml` Playbook

##### Syntax Verification of the `site.yaml` Playbook

1. Verify syntax.

    ```bash
    cd ~/ansible-project
    ansible-playbook playbooks/site.yaml --syntax-check
    ```

    * Output.

    ```sh
    playbook: playbooks/site.yaml
    ```

---

##### Dry Run of the `site.yaml` Playbook

1. Dry run of the `site.yaml` playbook.

    ```bash
    ansible-playbook playbooks/site.yaml \
      --check --diff \
      --ask-vault-pass
    ```

    * Output.

    ```sh
    nat: ok=11 changed=0 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0   
    pods: ok=30 changed=0 unreachable=0 failed=0 skipped=32 rescued=0 ignored=0   
    proxy: ok=21 changed=0 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0   
    redis: ok=14 changed=0 unreachable=0 failed=0 skipped=19 rescued=0 ignored=0   
    ```

---

##### Execution of the `site.yaml` Playbook

1. Actual execution of the `site.yaml` playbook.

    ```bash
    ansible-playbook playbooks/site.yaml \
      --ask-vault-pass
    ```

    * Output (First run).

    ```sh
    nat: ok=14 changed=0 unreachable=0 failed=0 skipped=8 rescued=0 ignored=0
    pods: ok=46 changed=0 unreachable=0 failed=0 skipped=17 rescued=0 ignored=0   
    proxy: ok=23 changed=0 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0   
    redis: ok=23 changed=0 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0   
    ```

    * Output (Second run).

    ```sh
    nat: ok=13 changed=0 unreachable=0 failed=0 skipped=8 rescued=0 ignored=0   
    pods: ok=45 changed=0 unreachable=0 failed=0 skipped=17 rescued=0 ignored=0   
    proxy: ok=22 changed=0 unreachable=0 failed=0 skipped=10 rescued=0 ignored=0   
    redis: ok=22 changed=0 unreachable=0 failed=0 skipped=11 rescued=0 ignored=0   
    ```

---

#### Result

```txt
└── ansible-project
    ├── ansible.cfg
    ├── ansible.log
    ├── inventory
    │   ├── group_vars
    │   │   └── all
    │   │       ├── all.yaml
    │   │       └── vault.yaml
    │   ├── hosts.yaml
    │   └── host_vars
    │       ├── nat.yaml
    │       ├── pods.yaml
    │       ├── proxy.yaml
    │       └── redis.yaml
    ├── playbooks
    │   ├── site.yaml
    │   └── tests
    │       ├── test_common.yaml
    │       ├── test_glpi_pod.yaml
    │       ├── test_haproxy.yaml
    │       ├── test_podman_base.yaml
    │       ├── test_redis.yaml
    │       └── test_wikijs_pod.yaml
    └── roles
        ├── common
        │   ├── handlers
        │   │   └── main.yaml
        │   └── tasks
        │       ├── firewall.yaml
        │       ├── hosts.yaml
        │       ├── main.yaml
        │       ├── nat.yaml
        │       ├── selinux.yaml
        │       └── timezone.yaml
        ├── glpi_pod
        │   ├── defaults
        │   │   └── main.yaml
        │   ├── handlers
        │   │   └── main.yaml
        │   ├── tasks
        │   │   └── main.yaml
        │   └── templates
        │       ├── glpi.container.j2
        │       ├── glpi.env.j2
        │       ├── glpi-pod.pod.j2
        │       ├── mariadb.cnf.j2
        │       ├── mariadb.container.j2
        │       └── mariadb.env.j2
        ├── haproxy
        │   ├── defaults
        │   │   └── main.yaml
        │   ├── handlers
        │   │   └── main.yaml
        │   ├── tasks
        │   │   └── main.yaml
        │   └── templates
        │       ├── haproxy.cfg.j2
        │       └── openssl.cnf.j2
        ├── podman_base
        │   ├── defaults
        │   │   └── main.yaml
        │   ├── handlers
        │   │   └── main.yaml
        │   └── tasks
        │       └── main.yaml
        ├── redis
        │   ├── defaults
        │   │   └── main.yaml
        │   ├── handlers
        │   │   └── main.yaml
        │   ├── tasks
        │   │   └── main.yaml
        │   └── templates
        │       └── redis.conf.j2
        └── wikijs_pod
            ├── defaults
            │   └── main.yaml
            ├── handlers
            │   └── main.yaml
            ├── tasks
            │   └── main.yaml
            └── templates
                ├── postgres.env.j2
                ├── postgres-wikijs.container.j2
                ├── wikijs.container.j2
                ├── wikijs.env.j2
                └── wikijs-pod.pod.j2
```
