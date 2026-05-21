# Project, Part I - Intelligent Storage System

## Cover Page

### University of Costa Rica

* **Technological Infrastructure Integration CI-0138**
* **Project, Part I - Intelligent Storage System**
* **Professor:** Ariel Mora Jiménez
* **Student:** Anderson Vargas Navarro - C28183
* **2nd semester, 2025**
* **Group:** 001

---

## **Project, Part I - Intelligent Storage System**

### General Objective

Build the technological infrastructure for the storage of a small virtual
data center.

### Specific Objectives

1. Define the configuration strategy and use of storage according to
   predefined requirements for the data center.
2. Define low-level storage structures from virtual hard disks, volumes,
   and logical disk arrays.
3. Implement and enable the IP SAN service through the iSCSI protocol, and
   the NAS service through NFS protocols.
4. Describe the architecture of the implemented storage system.
5. Install test client applications for the configured services.

### General Description

This project consists of preparing the intelligent storage system for a
virtual data center, which will support the services to be implemented in
later stages of the course. For the project, the open-source software
called TrueNAS Scale will be used, which has been made available through
virtual machines on the Computational Academic Cloud (NAC) platform of the
UCR, along with the type 1 hypervisor VMWare ESXi for testing. The
activities to be developed during the project are the following:

1. Describe the architecture of the data center's storage system, based on
   requirements defined in this statement and the virtual resources
   available for its construction.
2. Define and implement physical volumes, logical volumes, disk arrays, and
   other required logical disk structures, through the console interface or
   the web interface of the TrueNAS Scale system.
3. Configure the iSCSI server service in the TrueNAS Scale system, to
   offer block-level storage service, and configure an ESXi client computer
   to perform basic tests with said service.
4. Configure network file services in the TrueNAS Scale system, and
   configure a client computer to perform tests with said services.

### Data Center Storage Requirements

This section describes the storage requirements for the operation of the
virtual data center. Considering that it is a simple data center, only 3
data requirements have been defined:

1. *Local storage for the operating system of a hypervisor*: each
   hypervisor in a data center, i.e. a computer that has computer
   virtualization capabilities, requires space for its operating system.
   This requirement will be satisfied using the local hard drive of the
   hypervisor itself, so no reservation or space consideration is required
   for this item.
2. *Storage for virtual machines*: it is the space used by virtual
   machines placed on a hypervisor. A storage space must be defined for
   each hypervisor, considering the following aspects:
    * It must be a block storage system, not a file system.
    * It must have reasonably good performance, as it will impact the
      execution times of the virtual machines.
    * It must consider some level of fault tolerance in the hard drives, to
      avoid immediate errors in the operation of the machines in case of a
      failure.
    * An initial amount of space can be defined for the hypervisor, with a
      space reserve to be expanded if necessary.
3. *Storage for support and installation files, among others*: it is the
   storage (repository) for placing ISO images, disks with virtual machine
   (VM) base images, application installers, etc., from which the running
   virtual machines are built. It is a common space for the data center's
   hypervisors, with the following considerations:
    * It must be a network storage system, to be shared.
    * It does not need to have the best level of performance, but does need
      some level of fault tolerance.
    * An initial amount of space must be allocated, with a space reserve to
      be expanded if necessary.
    * The required access control is that of a private network, in a
      network designated as secure, with access only to authorized
      equipment.

The total space available to satisfy the above requirements will depend on
your storage organization decisions. The documentation with the description
of the architecture must include the criteria used to establish the final
available capacities of each requirement.

### Construction of Arrays and Volumes

To meet the fault tolerance and storage performance requirements, you must
consider the types of RAID arrays studied, but also those available on the
Openfiler platform. To meet the extensibility requirements, which allow
expanding the available space in a service without affecting user
operation, you must use the functionality available through Linux Volume
Management (LVM), which must be researched and implemented.

### SAN Service Implementation

For the implementation of a block storage service, through a SAN, use the
iSCSI service provided by TrueNAS Scale.

### NAS Service Implementation

For the implementation of a network storage service, whether NFS, use the
services provided by TrueNAS Scale.

### Test Clients

For the implementation of the test clients, use a virtual machine with
ESXi in the Academic Cloud. For the ESXi client, after connecting the SAN
and NAS storages, build and install a small virtual machine by obtaining
the ISO image from the NAS and installing the operating system in the SAN
space. You can also directly import the virtual machine created in the
first lab of the course.

### Architecture Description

Prepare a document with the description of the architecture that considers
the following elements:

* General system description: it is an introductory section that contains
  the general characteristics of the storage system to be implemented, and
  that describes the architecture concerns obtained from the task statement.
* Description of architecture views: in this case, an architecture that
  includes views to describe the performance of the storage system, the
  security of the information, and the implementation of the storage system
  will be considered.
* Storage performance: describing the criteria and decisions made for the
  organization of storage resources.
* Information security: describing the security proposal for the
  information and system resources. Use the security model defined for the
  course, with the level of detail indicated in class.
* Implementation of the storage system: describing the components,
  procedures, and implementation details of the storage system.

### Available Storage Infrastructure

The available virtual storage infrastructure consists of 1 virtual disk of
16 Gbytes for the operating system of the storage service and 6 hard drives
of 25 Gbytes each, for a total of up to 150 Gbytes of (raw) space for each
mini data center. You can partition each of the hard drives into at least 2
partitions each, but no more than three partitions, depending on your array
and volume design. The disk array definition must consider the performance
and information security considerations, in the same way as if the working
platform were physical.

---

## System Architecture Description

### General System Description

This section will introductorily address the general characteristics of
the storage system to be implemented, identify the possible stakeholders
and the concerns obtained from them.

#### General System Characteristics

This intelligent storage system consists of a storage infrastructure using
*TrueNas Scale*, designed for a virtual data center that provides services
for block storage (*SAN* using *iSCSI*), as well as for files (*NAS* using
*NFS*). The system aims to guarantee performance, fault tolerance,
extensibility, and security; for this, *RAID* arrays, logical volumes are
used, as well as storage support for virtual machines managed with *VMWare
ESXi*, *ISO* images, and shared files within a secure network.

#### Stakeholders

For this part, only the primary stakeholders (*Primary Stakeholders*) were
identified.

* **Developers & Builders:** Their role is to design, implement, and
  configure the intelligent storage system; that is, the creation of
  physical and logical volumes, RAID arrays, and the configuration of the
  iSCSI (SAN) and NFS (NAS) services in the TrueNAS Scale system.
* **Operators & Maintainers:** Their role is to manage space allocation,
  volume expansion, backups, and fault recovery.
* **Users:** Their role is to use the data center's storage resources to
  host virtual machines, store ISO images, or share files.

#### Concerns

* **Developers & Builders:**
  * How is it ensured that the use and administration of local storage is
    adequate for the hypervisor's operating system?
  * How is it ensured that the partitioning strategy and defined disk
    arrays optimize the use of resources, meeting the requirements for
    initial capacity and reserve for expansion?
  * What RAID configurations offer the best balance between capacity,
    redundancy, and performance for block storage of VMs and network file
    storage?
  * How is it ensured that the iSCSI and NFS services are correctly
    configured, authenticated, and accessible from ESXi clients?
* **Operators & Maintainers:**
  * How will storage quota management be carried out to prevent excessive
    consumption by some parties?
  * How will continuous system monitoring be carried out, including disk
    status, performance, and volume utilization?
  * How will volume expansion and space management be handled without
    affecting service availability?
* **Users:**
  * How is the continuous availability of storage guaranteed, avoiding
    interruptions during file access or virtual machine execution?
  * How is the integrity and protection of stored data ensured against
    possible failures or network errors?
  * What performance levels and response times can be expected during read
    and write operations?

---

### Description of Architecture Views

This section presents the architecture views that describe the storage
system, from the perspectives of performance, security, and implementation.
The **performance view** defines how available resources will be organized
to meet efficiency and availability objectives. The **security view**
describes the methods that primarily protect data integrity; and the
**implementation view** details how the specific components and
configurations are materialized after the proposed design.

---

### Storage Performance

This section describes the design criteria taken for the logical
organization of storage, which seek to balance performance, fault
tolerance, and capacity efficiency requirements.

#### Specific System Characteristics

For the implementation of this **Intelligent Storage System**, there are
various resources which, as already mentioned, must meet several criteria
and objectives. The resource specifications to be worked with are:

* **ESXi:**
  * **Memory:** 16GB
  * **Storage:** 4GB
* **TrueNas Scale:**
  * **Memory:** 8GB
  * **Storage:** 7 disks
    * 150GB (6 x 25GB)
    * 20GB (*boot-pool*)

#### Organization Strategy and Fault Tolerance

This subsection focuses on *TrueNas Scale* storage, and how the available
storage is distributed, the type of fault tolerance (*RAID*), the
justification for the selection, as well as a visualization of this
distribution and storage flow.

##### Distribution and Fault Tolerance Strategy**

* Since one of the 7 available disks cannot be used, the other 6 will be
  used, for a total of 150GB.
* With these 6 disks, a single *pool* with ZFS will be created to manage
  them.
* On this *pool*, a RAID type 5 configuration will be used (*striping with
  parity*).
* Due to this fault tolerance configuration, the space that can be
  assigned is the raw capacity minus the capacity of a single disk (for
  parity). Therefore, approximately 125 GB remain to be distributed.
* This *pool* storage has three main parts, which are:
  * **ZVOL** (*ZFS Volume*): This will be dedicated to the SAN *block
    storage* part with an approximate capacity of 100GB, using the *iSCSI*
    protocol.
  * **Dataset**: This part is used for NAS *file storage* with an
    approximate capacity of 15GB, using the *NFS* protocol.
  * **Other**: This storage will be dedicated to expansion, metadata, or
    snapshots, with a capacity of 10GB.

| Service | Protocol | Space | Characteristic | Client |
| ---------------- | --------- | ------- | -------------- | ------------ |
| SAN for VMs | iSCSI | ~100 GB | Tolerance | ESXi (VMs) |
| NAS files | NFS | ~15 GB | Shared | ESXi |
| Expansion reserve | --- | ~10 GB | Snapshots | --- |

##### Justification of the Choice

* **Justification for the type of fault tolerance (RAID 5):**
  * **Capacity optimization:** Allows using the space of all disks minus
    one, used to store parity. This maximizes the available useful storage.
  * **Fault tolerance:** Protects information against the failure of one
    disk, allowing the system to continue operating while the failed disk
    is replaced; this ensures the continuous availability of services.
  * **Adequate performance:** Offers good performance in read and write
    operations, especially in mixed workloads (read/write). Although
    writing may be slightly slower than in RAID 1+0, in this context the
    balance between capacity and fault tolerance is a priority.
  * **Efficiency and scalability:** Parity space is distributed among all
    disks, avoiding overload on a single disk. This facilitates future
    pool expansion, if more disks are added.

* **Justification for storage distribution:**
  * **Block storage (SAN):** The approximate allocation of 100GB for the
    SAN is related to the space and performance demand occupied by VMs in
    ESXi. It also allows hosting many virtual machines, and this is
    necessary since it is expected that this system will contain several
    VMs in the future.
  * **File storage (NAS):** Approximately 15GB are allocated for the NAS,
    sufficient to store support files and ISO images; additionally, in
    relation, this allocation avoids "over-provisioning" a section that
    does not have a critical IOPS service.
  * **Other:** 10GB is allocated for future expansion, since this part is
    one of the relevant concerns required: a part for future expansion, as
    well as metadata, snapshots, and internal operation.

![infra_proy_pt1_1.png](../assets/infra_proy_pt1_1.png)

---

### Information Security

This section describes the security measures and properties implemented in
the storage system, with the objective of primarily protecting the
integrity, availability, and confidentiality properties of the data, as
well as controlling access to and use of storage resources.

#### Data Integrity

* Thanks to the RAID 5 configuration, which implements distributed parity,
  providing a layer of protection against *pool* site failures. This
  ensures that data in the event of a failure can be safely reconstructed
  without loss of information.
* A key point of using the ZFS file system is that it incorporates data
  verification and auto-correction (*checksum*), which allows detecting and
  repairing read and write errors when they occur.

#### System Availability

* Fault tolerance due to the RAID 5 strategy, applied to the pool's disks,
  which ensures the system can survive the failure of a complete virtual
  disk without stopping the service. The service remains active during
  reconstruction, which avoids interruptions in SAN (iSCSI) and NAS (NFS)
  accesses.
* The distribution of storage across different datasets and volumes ensures
  that a failure in one part does not affect the functioning of another
  area of the system.

#### Confidentiality, Authentication, and Authorization

Access control is established through users and roles defined in both
*TrueNAS Scale* and *ESXi*.
In *TrueNAS* there are two main users:

* **root**: user with full system administration privileges.
* **admin**: operational user with configuration and general management
  permissions, without access to critical base system tasks.
In *ESXi* three user levels are defined:
* **root**: full access for hypervisor management and advanced maintenance.
* **admin**: responsible for the creation, configuration, and control of
  virtual machines.
* **viewer**: user with read-only permissions, limited to observing the
  system status.
These user levels ensure authentication (identity verification before
access) and authorization (privilege limitation by role), ensuring that
each interaction with the system is duly controlled.

#### Information Separation

* The division of storage into SAN and NAS allows separating critical
  information from support files and VM bases, minimizing the risk of
  unauthorized access or cross-corruption.
* Access policies and permissions are defined by resource type and user
  profile, ensuring that relevant information is available only to those
  who require it according to their role.

---

## Storage System Implementation

### Configuration in TrueNAS Scale

#### Creating the ZFS *pool*

This section creates the *pool* `data_pool` which combines the six 25GB
disks, and the type of fault tolerance will be selected. Using as
reference [Create ZFS pool](https://www.youtube.com/watch?v=vn-r4NNq4iY&t=853s).

1. Go to the Storage > Pools section, within the creation menu, click the
   `ADD` button.
![pasted_image_20251007102432](../assets/pasted_image_20251007102432.png)
2. Select the *Create new pool* option and press the `CREATE POOL` button.
![pasted_image_20251007102518](../assets/pasted_image_20251007102518.png)
3. Choose the name, in this case *data_pool*, then select the disks to be
   added to the *pool* (in this case all six) and they are added to the
   *Data VDevs*, then choose the type of fault tolerance, in this case
   *Raid-z* (RAID 5).
![pasted_image_20251007105450](../assets/pasted_image_20251007105450.png)
4. Select *Confirm* and press the `CREATE POOL` button.
![pasted_image_20251007105548](../assets/pasted_image_20251007105548.png)
5. After creation, the new *pool* `data_pool` can be seen.
![pasted_image_20251007105800](../assets/pasted_image_20251007105800.png)

#### Creating and Configuring the Dataset (NAS)

This section creates and configures the *dataset* for the NAS part that
will be transferred by the NFS protocol. Using as guide
[Configure NFS Dataset](https://www.youtube.com/watch?v=e9LSQbPKY_8),
[Create Datasets in TrueNAS](https://www.youtube.com/watch?v=X2JfU6cqyzc) and
[Storage Management](https://www.youtube.com/watch?v=27MhvLKBKtQ).

1. Due to the space reduction, and that it is now 105Gib instead of 125GB,
   it must be redistributed.

    | Service | Protocol | Space | Characteristic | Client |
    | ---------------- | --------- | ------- | -------------- | ------------ |
    | SAN for VMs | iSCSI | ~85 GiB | Tolerance | ESXi (VMs) |
    | NAS files | NFS | ~12 GiB | Shared | ESXi |
    | Expansion reserve | --- | ~8 GiB | Snapshots | --- |

2. Go to the Storage > Pools section.
![pasted_image_20251008082046](../assets/pasted_image_20251008082046.png)
3. Select the previously created `data_pool` and press the three dots and
   choose the `Add Dataset` option.
![pasted_image_20251008082259](../assets/pasted_image_20251008082259.png)
4. For the *Dataset* creation, select the name, in this case `nas_data`,
   add a comment, the rest of the options are recommended in this case to
   leave them as default, likewise the *Share Type* with the *Generic*
   option. And an important part in the `ADVANCED OPTIONS` section is the
   *Quota for this dataset*, in which the desired capacity must be assigned,
   in this case 12Gib.
![pasted_image_20251008083151](../assets/pasted_image_20251008083151.png)
![pasted_image_20251008084838](../assets/pasted_image_20251008084838.png)
5. After its creation, the new *Dataset* `nas_data` can be seen inside the
   pool.
![pasted_image_20251008085140](../assets/pasted_image_20251008085140.png)

---

#### Creating and Configuring the ZVOL (SAN)

This section creates and configures the *ZVOL* for the NAS part that will
be transferred by the *iSCSI* protocol. Using as guide
[TrueNAS Guide for ZVOLs](https://www.truenas.com/docs/core/13.0/coretutorials/storage/pools/zvols/).

1. Go to the Storage > Pools section.

   ![pasted_image_20251008090022](../assets/pasted_image_20251008090022.png)

2. Select the `data_pool` and press the three dots and choose the `Add
   Zvol` option.

   ![pasted_image_20251008090329](../assets/pasted_image_20251008090329.png)

3. For the *ZVOL* creation, select the name, in this case `san_vol`, add
   a comment, the size part *Size for this zvol* with 85GiB; the rest of
   the options by default, *Sync* as *Standard*, *Compression level* as
   *lz4*, and *ZFS Deduplication*.

   ![pasted_image_20251008091908](../assets/pasted_image_20251008091908.png)

   > *NOTE:* When assigning 85GiB to the *ZVOL*, it dropped to 80GiB,
   > which caused the *Dataset* to be edited from 12GiB to 15GiB.

   | Service | Protocol | Space | Characteristic | Client |
   | ---------------- | --------- | ------- | -------------- | ------------ |
   | SAN for VMs | iSCSI | ~80 GiB | Tolerance | ESXi (VMs) |
   | NAS files | NFS | ~15 GiB | Shared | ESXi |
   | Expansion reserve | --- | ~10 GiB | Snapshots | --- |

4. After its creation, the *Dataset* `nas_data` and the *ZVOL* `san_vol`
   can be seen inside the *pool*.

   ![pasted_image_20251008093048](../assets/pasted_image_20251008093048.png)

---

#### Configuring *Unix Shares* (NFS) *Sharing*

This section configures the *Unix Shares* (NFS) of the *Dataset* `nas_data`,
so that it can be accessed by *ESXi*. Using as reference
[Configure NFS in TrueNAS](https://www.youtube.com/watch?v=e9LSQbPKY_8).

1. Go to the Storage > Shares > Unix Shares (NFS) section and click the
   `ADD` button.
![pasted_image_20251008095456](../assets/pasted_image_20251008095456.png)
2. In the *Paths* section, select the one for the created *Dataset*, which
   is `/mnt/data_pool/nas_data`.
![pasted_image_20251008095724](../assets/pasted_image_20251008095724.png)
3. For the *Access* options, add the *admin* user and the *wheel* group.
![pasted_image_20251008101035](../assets/pasted_image_20251008101035.png)
4. In the *Networks* section, add the internal network and the *ESXi*
   *host*.
![pasted_image_20251008101225](../assets/pasted_image_20251008101225.png)
5. In the *enabled* option, set it to yes, to activate the *NFS* services.
![pasted_image_20251008101400](../assets/pasted_image_20251008101400.png)
6. After the configuration, the creation of the new *share* can be seen in
   the menu.
![pasted_image_20251008101451](../assets/pasted_image_20251008101451.png)

---

#### Configuring *NAS* (NFS) Access from *ESXi*

This section configures from *ESXi* what is necessary to access the *NAS*
storage previously configured in *TrueNAS Scale* with *NFS*.

1. Go to the *Storage* section and press the `New datastore` option.
![pasted_image_20251008105233](../assets/pasted_image_20251008105233.png)
2. Select the `Mount NFS datastore` option and click `Next`.
![pasted_image_20251008105346](../assets/pasted_image_20251008105346.png)
3. Select the name, in this case `nas_data`, in *NFS server* the IP of
   *TrueNAS Scale*, in this case `172.24.131.206`, in *NFS share* the path
   of the shared resource, in this case `/mnt/data_pool/nas_data`, and in
   *NFS version* choose *NFS 3*.
![pasted_image_20251008105735](../assets/pasted_image_20251008105735.png)
4. Confirm the details before finishing.
![pasted_image_20251008110228](../assets/pasted_image_20251008110228.png)
5. After the configuration, `nas_data` now appears in the *Datastores*
   section with a capacity of 15GB and NFS type, as previously configured.
![pasted_image_20251008110702](../assets/pasted_image_20251008110702.png)
![pasted_image_20251008110803](../assets/pasted_image_20251008110803.png)
6. To verify for now that it was done correctly, a test file can be
   created. To do this, click on `Datastore browser` and then on
   *Create directory*.

    ![pasted_image_20251008112325](../assets/pasted_image_20251008112325.png)

    > When creating it, there was an error with permissions; to fix it in
    > the *TrueNAS Scale* terminal, the permissions were changed.

    ```sh
    chown root:wheel /mnt/data_pool/nas_data
    chmod 770 /mnt/data_pool/nas_data
    ```

    > Now there are full permissions to create, delete, and modify files
    > and folders in the NAS *datastore*.

7. Verify that the directory was created; a test file will also be uploaded
   inside it.
![pasted_image_20251008114742](../assets/pasted_image_20251008114742.png)
8. Verify that the file is inside the previously created directory.
![pasted_image_20251008114823](../assets/pasted_image_20251008114823.png)

---

#### Configuring *Block Shares* (iSCSI) *Sharing*

This section configures the *Block Shares* (iSCSI) of the *ZVOL* `san_vol`,
so that it can be accessed by *ESXi* manually, without using the *Wizard*.
Using as reference
[TrueNas Tutorial](https://www.truenas.com/docs/scale/scaletutorials/shares/iscsi/addingiscsishares/).

1. Go to the Storage > Shares > Block Shares (iSCSI) section.
![pasted_image_20251008161422](../assets/pasted_image_20251008161422.png)
2. For the `Target Global Configuration` configuration, leave the *Base
   Name* as default since it follows the standard format, the *ISNS
   Servers* as empty since it is not used in this case, and the *Pool
   Available Space Threshold* is optional to add, in this case 10. Click
   the `SAVE` button and then *Enable service* with `ENABLE SERVICE`.
![pasted_image_20251008161921](../assets/pasted_image_20251008161921.png)
![pasted_image_20251008162128](../assets/pasted_image_20251008162128.png)
![pasted_image_20251008162144](../assets/pasted_image_20251008162144.png)
3. In the `Portals` section, this is where the IP and port where iSCSI
   connections will be accepted are defined. Click `ADD`, then inside add a
   description and the rest of the configurations as they are (by default),
   and in *IP Address* the one at 0.0.0.0 and port 3260.
![pasted_image_20251008162424](../assets/pasted_image_20251008162424.png)
![pasted_image_20251008162840](../assets/pasted_image_20251008162840.png)
![pasted_image_20251008163111](../assets/pasted_image_20251008163111.png)
4. For the `Initiators Groups` configuration, which defines which clients
   can connect, press the `ADD` button. The ESXi IP and the authorized
   network are added.
![pasted_image_20251008163148](../assets/pasted_image_20251008163148.png)
![pasted_image_20251008163518](../assets/pasted_image_20251008163518.png)
![pasted_image_20251008163607](../assets/pasted_image_20251008163607.png)
5. In the `Authorized Access` section, leave it as is, as it comes by
   default.
![pasted_image_20251008163706](../assets/pasted_image_20251008163706.png)
6. For the `Targets` configuration, in which iSCSI targets are defined,
   press the `ADD` button. Select the name and alias, also in the *Portal
   Group* and *Initiator Group*, select the ones previously added.
![pasted_image_20251008163938](../assets/pasted_image_20251008163938.png)
![pasted_image_20251008164533](../assets/pasted_image_20251008164533.png)
![pasted_image_20251008164739](../assets/pasted_image_20251008164739.png)
7. For the `Extents` configuration, which defines what storage is shared,
   press the `ADD` button. Choose the number, the description, in *Devices*
   select the previously created `data_pool/san_vol`, and the rest of the
   configurations as they come by default.
![pasted_image_20251008165258](../assets/pasted_image_20251008165258.png)
![pasted_image_20251008165448](../assets/pasted_image_20251008165448.png)
8. In the `Associated Targets` section, where *extents* are associated with
   *targets*, press the `ADD` button.
![pasted_image_20251008165904](../assets/pasted_image_20251008165904.png)
![pasted_image_20251008165922](../assets/pasted_image_20251008165922.png)

---

#### Configuring *SAN* (iSCSI) Access from *ESXi*

This section configures from *ESXi* what is necessary to access the *SAN*
storage previously configured in *TrueNAS Scale* with *iSCSI*.

1. Go to the *Storage* section, in the `Adapters` section select the
   `Configure iSCSI` option.
![pasted_image_20251008170819](../assets/pasted_image_20251008170819.png)
2. Enable iSCSI by selecting `Enabled`.
![pasted_image_20251008170937](../assets/pasted_image_20251008170937.png)
3. Configure the *Network port bindings* section, where the IP being used
   is added, then in *Dynamic targets*, the *TrueNAS* IP is added.
![pasted_image_20251008171525](../assets/pasted_image_20251008171525.png)
4. After creating it, verify that it was done correctly.
![pasted_image_20251008175735](../assets/pasted_image_20251008175735.png)

    *NOTE:* In the *Devices* section, the disk did not appear, so to test
    in *TrueNAS* *Initiator Groups*, *Allow All Initiators* is selected.
    After this it does appear in the *Devices* list in *ESXi*; after this,
    so that it does not remain this way, within *Initiators Groups*, the
    *ESXi* `iqn.1998-01.com.vmware:vm-131-196-7756ea59` now appears. After
    the previous configuration, the disk now appears in *Devices*, with
    80GB capacity.

    ![pasted_image_20251008182658](../assets/pasted_image_20251008182658.png)

5. In the Storage > Datastores section, select `New datastore`
![pasted_image_20251008183357](../assets/pasted_image_20251008183357.png)
6. Choose the `Create new VMFS datastore` option and click `Next`.
![pasted_image_20251008190040](../assets/pasted_image_20251008190040.png)
7. Select the added device, *TrueNAS iSCSI Disk*.
![pasted_image_20251008190659](../assets/pasted_image_20251008190659.png)
8. Then the partitioning options are chosen.
![pasted_image_20251008190812](../assets/pasted_image_20251008190812.png)
9. Final review of the selected configurations and press `Finish`.
![pasted_image_20251008190851](../assets/pasted_image_20251008190851.png)
10. Choose the `Yes` option.
![pasted_image_20251008190931](../assets/pasted_image_20251008190931.png)
11. Now in the *Datastores* section, both types of storage are found:
    `nas_data` and `san_data`.
![pasted_image_20251008191124](../assets/pasted_image_20251008191124.png)

---

#### Implementation of Test Clients

For this section the test clients are implemented in ESXi; to do this, a
virtual machine must be loaded in the NAS space by loading the image, then
the operating system of the virtual machine must be installed in the SAN
space. Used as reference
[Deploy VMs in ESXi](https://www.youtube.com/watch?v=NIfVTKPas0k) and
[Import OVF in vSphere](https://www.youtube.com/watch?v=kSA8TZu1Gzc).

1. First, what must be done is to export the base image previously created
   from *VMWare Workstation*, in the File > Export to OVF... section.
![pasted_image_20251008200647](../assets/pasted_image_20251008200647.png)
2. Now inside *ESXi* in the Storage > Datastores > nas_data section,
   select *Datastore browser*
![pasted_image_20251008200925](../assets/pasted_image_20251008200925.png)
3. Inside the *Datastore browser*, press `Upload` and browse to the path
   where the *golden image* virtual machine was exported.
![pasted_image_20251008201328](../assets/pasted_image_20251008201328.png)
4. Now select and upload the three files: *.vmdk*, *.ovf*, and *.mf*.
![pasted_image_20251008203551](../assets/pasted_image_20251008203551.png)
5. Then, in the Virtual Machines section, click the `Create / Register VM`
   option.
![pasted_image_20251008203748](../assets/pasted_image_20251008203748.png)
6. Select the `Deploy a virtual machine from OVF or OVA file` option.
![pasted_image_20251008203912](../assets/pasted_image_20251008203912.png)
7. Now choose the name of the virtual machine, in this case `RockyServer9-01`,
   and also the configuration files.
![pasted_image_20251008204405](../assets/pasted_image_20251008204405.png)
8. Now in the *Select storage* section, choose `san_data` as the datastore.
![pasted_image_20251008204519](../assets/pasted_image_20251008204519.png)
9. For the *Deployment options* section, choose the options that are
   already set by default.
![pasted_image_20251008204637](../assets/pasted_image_20251008204637.png)

***NOTE:***

* When clicking `Next`, there was a failure, due to the virtual hardware
  version being newer than that supported by the ESXi host.
![pasted_image_20251008205510](../assets/pasted_image_20251008205510.png)
* To solve this, within *VMWare Workstation*, in the `VM` section, and
  `Manage`, and the *Change Hardware compatibility* section, select
  `Workstation 14.x` and then clone the VM into a new one. After this,
  export it in .ovf and redo everything from that point.

![pasted_image_20251008210937](../assets/pasted_image_20251008210937.png)
10. Verify that the selected configurations are correct.
![pasted_image_20251008212038](../assets/pasted_image_20251008212038.png)

***NOTE***:

* Similarly, despite this, it failed when creating the virtual machine, so
  a virtual machine is created from scratch.
* An Ubuntu was created with all default specifications; additionally, the
  only relevant part of its creation was selecting `san_data` as storage.
![pasted_image_20251008213942](../assets/pasted_image_20251008213942.png)
