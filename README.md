# Enterprise Infrastructure Project: Design, Implementation, and Automation

## Project Description

This project documents the design, implementation, and automation of an infrastructure oriented towards enterprise services, high availability, and infrastructure modernization. The project covers everything from the construction of a centralized storage platform with TrueNAS Scale, to the implementation of ITSM services, high availability clusters, containers with Podman, and complete automation using Ansible.

The infrastructure was developed following principles of network segmentation, fault tolerance, security, and reproducibility, integrating technologies such as GLPI, MariaDB Galera, HAProxy, Keepalived, Redis, Wiki.js, and container security analysis tools.

The repository is organized by evolutionary parts that show the progressive growth of the architecture:

* **Intelligent storage** with ZFS, iSCSI, and NFS.
* **Core IT services** with hardening and network segmentation.
* **High availability and load balancing** eliminating single points of failure.
* **Modernization with containers** using Podman and rootless pods.
* **Declarative automation** using Ansible playbooks and roles.
* **Identity management** with FreeIPA for centralized authentication and access control.
* **Monitoring and observability** with Prometheus, Loki, and Grafana for metrics, logs, and alerting.

---

## Projects

### [Part I: Intelligent Storage System](docs/part1-storage/README.md)

Implementation of the data persistence layer for the virtual data center, using **TrueNAS Scale** as a unified storage platform.

* **Protocols:** Configuration of **SAN** services using **iSCSI** for block storage (VMFS datastore), and **NAS** using **NFS** for shared file repositories.
* **Architecture:** Creation of a ZFS pool with **RAID-Z (RAID 5)** configuration, combining six virtual disks to offer redundancy against single disk failure and balanced performance.
* **Volume Management:** Allocation of specific capacities using *ZVOLs* (for iSCSI) and *Datasets* (for NFS), implementing quotas and reservations to guarantee extensibility and resource isolation.
* **Objective:** Provide a fault-tolerant, high-performance, centralized storage infrastructure capable of hosting complete operating systems (VMs) and support files (ISOs, templates).

---

### [Part II: IT Infrastructure Services](docs/part2-core-services/README.md)

Deployment and hardening of an IT Service Management (ITSM) application on a segmented network architecture.

* **Application:** Base implementation of **GLPI** for unified management of inventory, incidents, contracts, and data center documentation.
* **Technology Stack:** Deployment on **Rocky Linux 9**, using **Nginx** as a web server and reverse proxy, **PHP-FPM** for application logic processing, and **MariaDB** as the relational database engine.
* **Network Architecture:** Functional segmentation into four isolated networks (Public, Internal, App, and DB), applying strict zone-based firewall policies to separate user, administration, and internal service traffic.
* **Security and Management:** Application of OS hardening (SELinux in enforcing mode, port control, Fail2ban), SSH key authentication, and isolation of management channels (SSH) from service publication channels (HTTPS).

---

### [Part III: High Availability and Load Balancing (HA)](docs/part3-high-availability/README.md)

Evolution of the architecture towards a high availability and fault tolerance model, eliminating single points of failure in all service layers.

* **Load Balancing:** Implementation of an active/passive **HAProxy** cluster managed by **Keepalived**, using a **Virtual IP (VIP)** to distribute HTTPS traffic and ensure automatic failover in the web layer.
* **Database Cluster:** Configuration of an active/active **MariaDB Galera** cluster with two nodes, guaranteeing synchronous replication and data consistency. **MaxScale** was deployed as a database proxy to balance reads and manage write failover.
* **Application High Availability:** Cluster of two web servers (Nginx + PHP-FPM) in active/passive mode, where **Keepalived** transparently moves the VIP between nodes to ensure GLPI service continuity.
* **Resilience Tests:** Documentation and execution of systematic *failover* tests for all layers (application, DB proxy, and DB cluster), demonstrating the system's ability to maintain continuous operation without manual intervention in the event of individual node failures.

---

### [Part IV: Modernization and Containers](docs/part4-containers/README.md)

Migration of native services to a container-based architecture, improving portability, isolation, and maintainability of the application ecosystem.

* **Container Engine:** Implementation of **Podman** for daemonless container execution (*rootless*), using **Quadlet** for declarative pod management and systemd for service lifecycle management.
* **Pod Orchestration:** Design of dedicated pods (GLPI + MariaDB, Wiki.js + PostgreSQL) that share a network namespace, simplifying loopback communication and reducing the port exposure surface.
* **Image Security:** Pre-deployment analysis of all base images (GLPI, MariaDB, Wiki.js, PostgreSQL) using **Trivy** (CVE scanning) and **Dockle** (CIS best practices verification), selecting versions with acceptable vulnerability profiles for the environment.
* **Persistence and Secrets:** Management of persistent storage using volumes mounted on host directories with SELinux contexts (`:Z`), and handling of sensitive credentials through Podman secrets injected as files or environment variables.
* **Cache Integration:** Deployment of an independent **Redis** server on an isolated network, integrated with GLPI as a session and data cache backend to optimize performance.
* **Documentation Platform:** Deployment of **Wiki.js** in a containerized pod, accessible via HAProxy on a dedicated port, for centralized project documentation and future implementations.

---

### [Part V: Ansible](docs/part5-ansible/README.md)

Complete automation of infrastructure deployment and configuration using **Ansible**, enabling reproducibility, scalability, and efficient lifecycle management of services.

* **Modular Playbooks:** Development of playbooks organized by roles (storage, core services, HA, containers) that encapsulate specific tasks for each infrastructure component.
* **Variables and Templates:** Extensive use of parameterized variables and Jinja2 templates to adapt configurations to different environments and facilitate customization without modifying the base code.
* **Idempotency and Validation:** Implementation of idempotent tasks that guarantee consistency of the desired state, along with post-deployment validations to verify correct configuration and operation of services (status verification, connectivity tests, configuration validation).

---

### [Phase VI: Identity Management System](docs/part6-freeipa/README.md)

Implementation of a centralized identity management system for the entire infrastructure using **FreeIPA**, providing unified authentication, DNS, and access control across all hosts and web services.

* **Identity Management:** Deployment of **FreeIPA** on a dedicated VM with **Rocky Linux 9**, providing centralized user and group management, Kerberos-based authentication, and a Certificate Authority for the `infra.local` domain.
* **Local DNS:** Configuration of FreeIPA as the internal DNS server, enabling all VMs to be reached by hostname instead of hard-coded IP addresses across the infrastructure.
* **Access Control:** Design and enforcement of **Host-Based Access Control (HBAC)** policies restricting which users can SSH to which hosts and which services they may use, following a "deny by default" model by disabling the built-in `allow_all` rule.
* **Sudo Rules:** Definition of granular **Sudo Rules** in FreeIPA mapping user groups to specific command groups (`cmd-group-services`, `cmd-group-readonly`), replacing local sudoers files across all nodes.
* **User and Group Design:** Creation of four role-based user groups (`infra-admins`, `infra-core-ops`, `infra-app-ops`, `infra-readonly`) with corresponding host groups (`all-managed`, `entry-servers`, `app-servers`, `infra-servers`, `idm-servers`) to model least-privilege access.
* **LDAP Integration:** Integration of **GLPI** and **Wiki.js** with FreeIPA via LDAP, using dedicated read-only service accounts (`glpi-ldap`, `wikijs-ldap`) so users authenticate to web services with their centralized credentials.
* **Automation:** FreeIPA server installation and client enrollment on all nodes automated through **Ansible** using the `freeipa.ansible_freeipa` collection, with sensitive credentials managed via Ansible Vault.

---

### [Phase VII: Monitoring System](docs/part7-monitoring/README.md)

Implementation of a monitoring system for the infrastructure enabling resource usage tracking and alert generation for events that may compromise service continuity.

* **Stack:** Deployment of **Prometheus + Loki + Grafana** as the unified observability platform, covering metrics, logs, and alerting from a single interface.
* **Metrics Collection:** Installation of **Node Exporter** on all nodes for OS-level metrics, along with service-specific exporters: **Redis Exporter**, **PostgreSQL Exporter**, **MySQLd Exporter**, **cAdvisor** (Podman containers), and native **HAProxy** metrics endpoint.
* **Log Aggregation:** Deployment of **Fluent Bit** on every node as a lightweight log forwarding agent, collecting from systemd journal, `/var/log/`, and Podman container logs, and shipping them to **Loki** for centralized storage and querying.
* **Dashboards:** Import and configuration of community dashboards (Node Exporter Full, HAProxy, Redis, cAdvisor, PostgreSQL, MySQL) to visualize the state of all infrastructure layers.
* **Alerting:** Definition of custom alert rules in **Grafana Alerting** with notification delivery via institutional email (SMTP) and **Telegram** bot, replacing the need for a separate Alertmanager.
* **Identity Integration:** Grafana integrated with **FreeIPA via LDAP** for centralized user authentication and role-based access control aligned with the existing HBAC policy.
* **Automation:** All components (Prometheus, Loki, Fluent Bit, Grafana, exporters) deployed and configured through dedicated **Ansible roles**, ensuring idempotent and reproducible provisioning.

---

## Contact Information

* **Name:** Anderson Vargas Navarro
* **Student ID:** C28183
* [**Institutional Email**](mailto:anderson.vargasnavarro@ucr.ac.cr)
* [**Personal Email**](mailto:vargasnavarroander@gmail.com)
* [**LinkedIn**](https://www.linkedin.com/in/anderson-vargas-navarro/)
