# AWS EC2 Storage: EBS, Snapshots, AMI, and Instance Store

## 1. Amazon EBS (Elastic Block Store)

**Amazon EBS** provides persistent block storage volumes that can be attached to EC2 instances.

Think of an EBS volume as a **virtual hard drive connected to an EC2 instance through the network**.

### Key Characteristics

- **Persistent Storage:** Data can survive EC2 instance stops and termination, depending on the volume's deletion settings.
- **Network-Based:** Communicates with EC2 over the network, which introduces some latency.
- **Availability Zone (AZ):** Each volume belongs to a specific AZ.
- **Attach/Detach:** Volumes can be detached and attached to other compatible EC2 instances in the same AZ.
- **Capacity:** Storage size, IOPS, and throughput depend on the volume type and configuration.
- **Billing:** You pay for provisioned storage capacity and, depending on the volume type, provisioned performance.

**Important:** EBS volumes normally attach to one instance at a time. Supported io1/io2 volumes can use Multi-Attach with compatible EC2 instances.

### Example

An EC2 instance runs a Node.js application:

- **EC2:** Provides the computing resources to execute the application.
- **EBS:** Stores the operating system, installed dependencies, application files, logs, or other persistent data.

If the EC2 instance stops, the EBS data remains available.

### Availability Zone Restrictions

An EBS volume can only be attached to an EC2 instance in the same Availability Zone.

For example:

- Volume in `us-east-1a` → EC2 in `us-east-1a`: Allowed.
- Volume in `us-east-1a` → EC2 in `us-east-1b`: Not allowed.

To move the data to another AZ, create an EBS snapshot and restore it as a new volume in the destination AZ.

---

## 2. EBS Delete on Termination

The **Delete on Termination** attribute determines whether an EBS volume is automatically deleted when its EC2 instance is terminated.

| Volume | Default Behavior |
|---|---|
| Root EBS volume | Deleted on termination |
| Additional EBS volume | Preserved on termination |

These defaults can be modified through the AWS Console or AWS CLI.

**Use case:** Disable Delete on Termination for a root volume when you need to preserve its data after terminating the EC2 instance.

**Important:** Stopping an EC2 instance does not delete its EBS volumes.

---

## 3. EBS Snapshots

An **EBS Snapshot** is a point-in-time backup of an EBS volume.

### Key Characteristics

- Used for backup, recovery, and volume migration.
- Stored and managed by AWS using Amazon S3 infrastructure.
- Snapshots are incremental after the initial snapshot, storing changed blocks.
- Can be used to create new EBS volumes in different Availability Zones.
- Can be copied across AWS Regions.
- Snapshots can be created while volumes are attached, although application-consistent backups may require temporarily pausing writes.

### Example: Moving EBS Data Between AZs

1. Create a snapshot of an EBS volume in `us-east-1a`.
2. Create a new EBS volume from the snapshot in `us-east-1b`.
3. Attach the new volume to an EC2 instance in `us-east-1b`.

### EBS Snapshot Features

#### Snapshot Archive

A lower-cost storage tier for snapshots that are rarely accessed.

- Can reduce snapshot storage costs by up to approximately 75%.
- Designed for long-term retention.
- Archived snapshots must be restored to the standard tier before use.
- Standard restore typically takes 24–72 hours.

#### Recycle Bin

Protects snapshots against accidental deletion.

- Retains deleted snapshots according to configured retention rules.
- Retention periods range from 1 day to 1 year.
- Allows recovery before the retention period expires.

#### Fast Snapshot Restore (FSR)

Improves performance when creating EBS volumes from snapshots.

- Eliminates the initial block-read latency associated with lazy initialization.
- Provides fully initialized volumes immediately after creation.
- Incurs additional charges.

---

## 4. Amazon Machine Image (AMI)

An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances with predefined software and configurations.

An AMI can include:

- Operating system.
- Installed applications and dependencies.
- System configurations.
- Monitoring software.
- Block device mappings and storage configuration.

### Benefits

- Faster EC2 deployment.
- Consistent configurations across instances.
- Reusable server environments.
- Reduced manual setup.

AMIs are Region-specific but can be copied across Regions.

### AMI Types

| Type | Description |
|---|---|
| AWS-provided AMI | Public images maintained by AWS |
| Custom AMI | Images created and maintained by users |
| AWS Marketplace AMI | Images published by third parties, sometimes requiring payment |

### Example

Instead of manually installing Node.js, dependencies, and monitoring tools on every new EC2 instance, create a custom AMI containing those configurations.

New EC2 instances launched from that AMI will already include the configured software.

**AMI vs. EBS Snapshot:**

- **AMI:** Template for launching an EC2 instance.
- **EBS Snapshot:** Backup of an EBS volume.

An EBS-backed AMI references EBS snapshots containing the instance's disk data.

---

## 5. EC2 Instance Store

**EC2 Instance Store** provides temporary block storage physically attached to the host running an EC2 instance.

It is designed for workloads requiring high disk I/O performance.

### Key Characteristics

- High-performance local storage.
- Lower latency for suitable workloads.
- Data is temporary (**ephemeral**).
- Data is lost when the instance stops, terminates, or the underlying host fails.
- Data normally survives an instance reboot.
- Backup and replication are the user's responsibility.

### Common Use Cases

- Caching.
- Temporary files.
- Buffers.
- Scratch data.
- Intermediate processing.
- High-performance temporary workloads.

### EBS vs. Instance Store

| Feature | EBS | Instance Store |
|---|---|---|
| Storage | Network-attached | Local host storage |
| Persistence | Persistent | Ephemeral |
| EC2 stop | Data preserved | Data lost |
| Detachable | Yes | No |
| Snapshots | Supported | Not directly supported |
| Typical use | OS, applications, databases | Cache, temporary data |

**Key concept:** Use EBS when data must persist. Use Instance Store for high-performance temporary storage.

---

## 6. EBS Volume Types

Amazon EBS offers six main volume types grouped into SSD and HDD categories.

### Storage Performance Metrics

- **Size:** Total storage capacity, measured in GiB or TiB.
- **IOPS (Input/Output Operations Per Second):** Number of read/write operations the storage can perform per second.
- **Throughput:** Amount of data transferred per second, measured in MiB/s.
- **Latency:** Time required to complete a storage operation.

### Volume Types Overview

| Type | Storage | Main Use Case |
|---|---|---|
| gp3 | SSD | General-purpose workloads |
| gp2 | SSD | General-purpose workloads |
| io2 Block Express | SSD | Critical databases and high-performance applications |
| io1 | SSD | Provisioned IOPS workloads |
| st1 | HDD | Frequently accessed, throughput-intensive data |
| sc1 | HDD | Infrequently accessed data |

**Boot volumes:** SSD volume types (gp2, gp3, io1, io2) support EC2 boot volumes. HDD types (st1, sc1) do not.

### 6.1 General Purpose SSD (gp2 / gp3)

Balanced performance and cost for general applications.

**Use cases:**
- EC2 boot volumes.
- Development and testing.
- Virtual desktops.
- Medium-sized databases.
- Transactional applications.

#### gp3

- Storage: 1 GiB – 64 TiB.
- Baseline: 3,000 IOPS.
- Baseline throughput: 125 MiB/s.
- IOPS and throughput can be provisioned independently of storage size.
- Supports up to 80,000 IOPS and 2,000 MiB/s, subject to applicable limits.

#### gp2

- Storage: 1 GiB – 16 TiB.
- Performance scales with volume size.
- Baseline: 3 IOPS per GiB, with a minimum of 100 IOPS.
- Smaller volumes can burst to 3,000 IOPS.
- Maximum: 16,000 IOPS.

**Key difference:** gp3 allows independent performance configuration, while gp2 performance depends primarily on volume size.

### 6.2 Provisioned IOPS SSD (io1 / io2)

Designed for applications requiring high, consistent I/O performance.

**Use cases:**
- Critical business applications.
- High-performance databases.
- Transaction-intensive workloads.
- Applications requiring predictable latency.

#### io1

- Storage: 4 GiB – 16 TiB.
- Maximum: 64,000 IOPS on supported Nitro instances.
- Maximum: 32,000 IOPS on other supported instances.
- IOPS can be provisioned independently of storage size.
- Maximum IOPS-to-GiB ratio: 50:1.

#### io2 Block Express

- Storage: 4 GiB – 64 TiB.
- Sub-millisecond latency.
- Maximum: 256,000 IOPS.
- Maximum IOPS-to-GiB ratio: 1,000:1.
- Designed for demanding database workloads.

**Multi-Attach:** Supported io1/io2 volumes can attach to multiple compatible EC2 instances in the same AZ, subject to configuration and limits.

### 6.3 Hard Disk Drives (HDD)

Lower-cost storage optimized for throughput-intensive workloads rather than high random IOPS.

HDD EBS volumes cannot be used as boot volumes.

#### Throughput Optimized HDD (st1)

Designed for frequently accessed, sequential workloads.

**Use cases:**
- Big Data.
- Data warehouses.
- Log processing.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 500 MiB/s.
- Maximum IOPS: 500.

#### Cold HDD (sc1)

Designed for infrequently accessed data where low storage cost is important.

**Use cases:**
- Infrequently accessed datasets.
- Low-cost storage.
- Workloads with limited performance requirements.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 250 MiB/s.
- Maximum IOPS: 250.

---

## 7. Choosing the Right EBS Volume

| Requirement | Recommended Type |
|---|---|
| General application storage | gp3 |
| EC2 operating system | gp3 |
| Development and testing | gp3 |
| High-performance databases | io2 Block Express |
| Consistent provisioned IOPS | io1 / io2 |
| Big Data and log processing | st1 |
| Infrequently accessed data | sc1 |
| Temporary high-speed storage | EC2 Instance Store |

**General recommendation:** gp3 is a common default for new EC2 workloads because it balances cost, performance, and flexibility.

## 8. Key Concepts to Remember

- **EBS:** Persistent network-attached block storage for EC2.
- **Availability Zone:** EBS volumes are restricted to a specific AZ.
- **Delete on Termination:** Controls whether EBS volumes are deleted when EC2 terminates.
- **Snapshots:** Point-in-time backups used for recovery and migration.
- **AMI:** Reusable template for launching configured EC2 instances.
- **Instance Store:** High-performance temporary local storage.
- **gp2/gp3:** General-purpose SSD storage.
- **io1/io2:** High-performance provisioned IOPS SSD storage.
- **st1:** Throughput-optimized HDD.
- **sc1:** Low-cost HDD for infrequently accessed data.


# AWS EC2 Storage: EBS, Snapshots, AMI, and Instance Store

## 1. Amazon EBS (Elastic Block Store)

**Amazon EBS** provides persistent block storage volumes that can be attached to EC2 instances.

Think of an EBS volume as a **virtual hard drive connected to an EC2 instance through the network**.

### Key Characteristics

- **Persistent Storage:** Data can survive EC2 instance stops and termination, depending on the volume's deletion settings.
- **Network-Based:** Communicates with EC2 over the network, which introduces some latency.
- **Availability Zone (AZ):** Each volume belongs to a specific AZ.
- **Attach/Detach:** Volumes can be detached and attached to other compatible EC2 instances in the same AZ.
- **Capacity:** Storage size, IOPS, and throughput depend on the volume type and configuration.
- **Billing:** You pay for provisioned storage capacity and, depending on the volume type, provisioned performance.

**Important:** EBS volumes normally attach to one instance at a time. Supported io1/io2 volumes can use Multi-Attach with compatible EC2 instances.

### Example

An EC2 instance runs a Node.js application:

- **EC2:** Provides the computing resources to execute the application.
- **EBS:** Stores the operating system, installed dependencies, application files, logs, or other persistent data.

If the EC2 instance stops, the EBS data remains available.

### Availability Zone Restrictions

An EBS volume can only be attached to an EC2 instance in the same Availability Zone.

For example:

- Volume in `us-east-1a` → EC2 in `us-east-1a`: Allowed.
- Volume in `us-east-1a` → EC2 in `us-east-1b`: Not allowed.

To move the data to another AZ, create an EBS snapshot and restore it as a new volume in the destination AZ.

### EBS Multi-Attach (io1/io2)

**EBS Multi-Attach** allows a single Provisioned IOPS SSD volume (io1/io2) to be attached to multiple EC2 instances simultaneously.

#### Key Characteristics

- Supports up to **16 EC2 instances** attached to the same volume.
- All instances must be in the **same Availability Zone**.
- Each instance can read and write to the shared volume.
- Requires supported Nitro-based EC2 instances.
- Designed for clustered applications requiring shared storage.

#### Use Cases

- High-availability clustered Linux applications.
- Shared storage for clustered databases, such as Teradata.
- Applications requiring multiple EC2 instances to access the same data.

#### Important Considerations

- **Concurrent Writes:** Applications must coordinate simultaneous write operations to prevent data corruption.
- **Cluster-Aware File System:** Shared file systems must support concurrent access from multiple instances.
- Standard file systems such as **EXT4 and XFS** are not designed for simultaneous mounting by multiple instances.

**Example:** Three EC2 instances in `us-east-1a` attach to the same io2 EBS volume, allowing a clustered application to access shared storage.

---

## 2. EBS Delete on Termination

The **Delete on Termination** attribute determines whether an EBS volume is automatically deleted when its EC2 instance is terminated.

| Volume | Default Behavior |
|---|---|
| Root EBS volume | Deleted on termination |
| Additional EBS volume | Preserved on termination |

These defaults can be modified through the AWS Console or AWS CLI.

**Use case:** Disable Delete on Termination for a root volume when you need to preserve its data after terminating the EC2 instance.

**Important:** Stopping an EC2 instance does not delete its EBS volumes.

---

## 3. EBS Snapshots

An **EBS Snapshot** is a point-in-time backup of an EBS volume.

### Key Characteristics

- Used for backup, recovery, and volume migration.
- Stored and managed by AWS using Amazon S3 infrastructure.
- Snapshots are incremental after the initial snapshot, storing changed blocks.
- Can be used to create new EBS volumes in different Availability Zones.
- Can be copied across AWS Regions.
- Snapshots can be created while volumes are attached, although application-consistent backups may require temporarily pausing writes.

### Example: Moving EBS Data Between AZs

1. Create a snapshot of an EBS volume in `us-east-1a`.
2. Create a new EBS volume from the snapshot in `us-east-1b`.
3. Attach the new volume to an EC2 instance in `us-east-1b`.

### EBS Snapshot Features

#### Snapshot Archive

A lower-cost storage tier for snapshots that are rarely accessed.

- Can reduce snapshot storage costs by up to approximately 75%.
- Designed for long-term retention.
- Archived snapshots must be restored to the standard tier before use.
- Standard restore typically takes 24–72 hours.

#### Recycle Bin

Protects snapshots against accidental deletion.

- Retains deleted snapshots according to configured retention rules.
- Retention periods range from 1 day to 1 year.
- Allows recovery before the retention period expires.

#### Fast Snapshot Restore (FSR)

Improves performance when creating EBS volumes from snapshots.

- Eliminates the initial block-read latency associated with lazy initialization.
- Provides fully initialized volumes immediately after creation.
- Incurs additional charges.

---

## 4. Amazon Machine Image (AMI)

An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances with predefined software and configurations.

An AMI can include:

- Operating system.
- Installed applications and dependencies.
- System configurations.
- Monitoring software.
- Block device mappings and storage configuration.

### Benefits

- Faster EC2 deployment.
- Consistent configurations across instances.
- Reusable server environments.
- Reduced manual setup.

AMIs are Region-specific but can be copied across Regions.

### AMI Types

| Type | Description |
|---|---|
| AWS-provided AMI | Public images maintained by AWS |
| Custom AMI | Images created and maintained by users |
| AWS Marketplace AMI | Images published by third parties, sometimes requiring payment |

### Example

Instead of manually installing Node.js, dependencies, and monitoring tools on every new EC2 instance, create a custom AMI containing those configurations.

New EC2 instances launched from that AMI will already include the configured software.

**AMI vs. EBS Snapshot:**

- **AMI:** Template for launching an EC2 instance.
- **EBS Snapshot:** Backup of an EBS volume.

An EBS-backed AMI references EBS snapshots containing the instance's disk data.

---

## 5. EC2 Instance Store

**EC2 Instance Store** provides temporary block storage physically attached to the host running an EC2 instance.

It is designed for workloads requiring high disk I/O performance.

### Key Characteristics

- High-performance local storage.
- Lower latency for suitable workloads.
- Data is temporary (**ephemeral**).
- Data is lost when the instance stops, terminates, or the underlying host fails.
- Data normally survives an instance reboot.
- Backup and replication are the user's responsibility.

### Common Use Cases

- Caching.
- Temporary files.
- Buffers.
- Scratch data.
- Intermediate processing.
- High-performance temporary workloads.

### EBS vs. Instance Store

| Feature | EBS | Instance Store |
|---|---|---|
| Storage | Network-attached | Local host storage |
| Persistence | Persistent | Ephemeral |
| EC2 stop | Data preserved | Data lost |
| Detachable | Yes | No |
| Snapshots | Supported | Not directly supported |
| Typical use | OS, applications, databases | Cache, temporary data |

**Key concept:** Use EBS when data must persist. Use Instance Store for high-performance temporary storage.

---

## 6. EBS Volume Types

Amazon EBS offers six main volume types grouped into SSD and HDD categories.

### Storage Performance Metrics

- **Size:** Total storage capacity, measured in GiB or TiB.
- **IOPS (Input/Output Operations Per Second):** Number of read/write operations the storage can perform per second.
- **Throughput:** Amount of data transferred per second, measured in MiB/s.
- **Latency:** Time required to complete a storage operation.

### Volume Types Overview

| Type | Storage | Main Use Case |
|---|---|---|
| gp3 | SSD | General-purpose workloads |
| gp2 | SSD | General-purpose workloads |
| io2 Block Express | SSD | Critical databases and high-performance applications |
| io1 | SSD | Provisioned IOPS workloads |
| st1 | HDD | Frequently accessed, throughput-intensive data |
| sc1 | HDD | Infrequently accessed data |

**Boot volumes:** SSD volume types (gp2, gp3, io1, io2) support EC2 boot volumes. HDD types (st1, sc1) do not.

### 6.1 General Purpose SSD (gp2 / gp3)

Balanced performance and cost for general applications.

**Use cases:**
- EC2 boot volumes.
- Development and testing.
- Virtual desktops.
- Medium-sized databases.
- Transactional applications.

#### gp3

- Storage: 1 GiB – 64 TiB.
- Baseline: 3,000 IOPS.
- Baseline throughput: 125 MiB/s.
- IOPS and throughput can be provisioned independently of storage size.
- Supports up to 80,000 IOPS and 2,000 MiB/s, subject to applicable limits.

#### gp2

- Storage: 1 GiB – 16 TiB.
- Performance scales with volume size.
- Baseline: 3 IOPS per GiB, with a minimum of 100 IOPS.
- Smaller volumes can burst to 3,000 IOPS.
- Maximum: 16,000 IOPS.

**Key difference:** gp3 allows independent performance configuration, while gp2 performance depends primarily on volume size.

### 6.2 Provisioned IOPS SSD (io1 / io2)

Designed for applications requiring high, consistent I/O performance.

**Use cases:**
- Critical business applications.
- High-performance databases.
- Transaction-intensive workloads.
- Applications requiring predictable latency.

#### io1

- Storage: 4 GiB – 16 TiB.
- Maximum: 64,000 IOPS on supported Nitro instances.
- Maximum: 32,000 IOPS on other supported instances.
- IOPS can be provisioned independently of storage size.
- Maximum IOPS-to-GiB ratio: 50:1.

#### io2 Block Express

- Storage: 4 GiB – 64 TiB.
- Sub-millisecond latency.
- Maximum: 256,000 IOPS.
- Maximum IOPS-to-GiB ratio: 1,000:1.
- Designed for demanding database workloads.

**Multi-Attach:** Supported io1/io2 volumes can attach to multiple compatible EC2 instances in the same AZ, subject to configuration and limits.

### 6.3 Hard Disk Drives (HDD)

Lower-cost storage optimized for throughput-intensive workloads rather than high random IOPS.

HDD EBS volumes cannot be used as boot volumes.

#### Throughput Optimized HDD (st1)

Designed for frequently accessed, sequential workloads.

**Use cases:**
- Big Data.
- Data warehouses.
- Log processing.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 500 MiB/s.
- Maximum IOPS: 500.

#### Cold HDD (sc1)

Designed for infrequently accessed data where low storage cost is important.

**Use cases:**
- Infrequently accessed datasets.
- Low-cost storage.
- Workloads with limited performance requirements.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 250 MiB/s.
- Maximum IOPS: 250.

---

## 7. Choosing the Right EBS Volume

| Requirement | Recommended Type |
|---|---|
| General application storage | gp3 |
| EC2 operating system | gp3 |
| Development and testing | gp3 |
| High-performance databases | io2 Block Express |
| Consistent provisioned IOPS | io1 / io2 |
| Big Data and log processing | st1 |
| Infrequently accessed data | sc1 |
| Temporary high-speed storage | EC2 Instance Store |

**General recommendation:** gp3 is a common default for new EC2 workloads because it balances cost, performance, and flexibility.

## 8. Key Concepts to Remember

- **EBS:** Persistent network-attached block storage for EC2.
- **Availability Zone:** EBS volumes are restricted to a specific AZ.
- **EBS Multi-Attach:** Allows up to 16 compatible EC2 instances in the same AZ to share an io1/io2 volume.
- **Delete on Termination:** Controls whether EBS volumes are deleted when EC2 terminates.
- **Snapshots:** Point-in-time backups used for recovery and migration.
- **AMI:** Reusable template for launching configured EC2 instances.
- **Instance Store:** High-performance temporary local storage.
- **gp2/gp3:** General-purpose SSD storage.
- **io1/io2:** High-performance provisioned IOPS SSD storage.
- **st1:** Throughput-optimized HDD.
- **sc1:** Low-cost HDD for infrequently accessed data.k

# AWS EC2 Storage: EBS, EFS, AMI, and Instance Store

## 1. Amazon EBS (Elastic Block Store)

**Amazon EBS** provides persistent block storage volumes that can be attached to EC2 instances.

Think of an EBS volume as a **virtual hard drive connected to an EC2 instance through the network**.

### Key Characteristics

- **Persistent Storage:** Data survives EC2 instance stops and can survive termination, depending on deletion settings.
- **Network-Based:** Communicates with EC2 over the network, introducing some latency.
- **Availability Zone (AZ):** Each EBS volume belongs to a specific AZ.
- **Attach/Detach:** Volumes can be detached and attached to other EC2 instances within the same AZ.
- **Capacity:** Storage size, IOPS, and throughput depend on the volume type.
- **Billing:** You pay for provisioned storage and, depending on the volume type, additional provisioned performance.

### Example

An EC2 instance runs a Node.js application:

- **EC2:** Provides computing resources to execute the application.
- **EBS:** Stores the operating system, dependencies, application files, logs, and persistent data.

If the EC2 instance stops, its EBS data remains available.

### Availability Zone Restrictions

An EBS volume can only attach to an EC2 instance in the same AZ.

- `us-east-1a` → `us-east-1a`: Allowed.
- `us-east-1a` → `us-east-1b`: Not allowed.

To move data between AZs, create an EBS snapshot and restore it into a new volume in the destination AZ.

### EBS Multi-Attach (io1/io2)

**EBS Multi-Attach** allows a single Provisioned IOPS SSD volume to attach to multiple EC2 instances simultaneously.

**Key Characteristics:**

- Supports up to **16 EC2 instances**.
- Instances must belong to the **same Availability Zone**.
- Each instance can read and write to the shared volume.
- Requires compatible Nitro-based EC2 instances.
- Available for supported io1/io2 volumes.

**Use Cases:**
- High-availability clustered Linux applications.
- Shared storage for clustered databases.
- Applications requiring multiple EC2 instances to access the same storage.

**Important Considerations:**

- Applications must coordinate concurrent writes to prevent data corruption.
- A **cluster-aware file system** is required when multiple instances mount the same file system simultaneously.
- Standard EXT4 and XFS file systems are not suitable for simultaneous multi-instance mounting.

---

## 2. EBS Delete on Termination

The **Delete on Termination** attribute determines whether an EBS volume is automatically deleted when an EC2 instance terminates.

| Volume | Default Behavior |
|---|---|
| Root EBS volume | Deleted on termination |
| Additional EBS volume | Preserved on termination |

These settings can be modified through the AWS Console or AWS CLI.

**Use case:** Disable Delete on Termination for a root volume when its data must be preserved after terminating the EC2 instance.

**Important:** Stopping an EC2 instance does not delete its EBS volumes.

---

## 3. EBS Snapshots

An **EBS Snapshot** is a point-in-time backup of an EBS volume.

### Key Characteristics

- Used for backup, recovery, and migration.
- Stored using AWS-managed Amazon S3 infrastructure.
- Incremental after the initial snapshot.
- Can restore volumes into different Availability Zones.
- Can be copied across AWS Regions.
- Can be created while volumes are attached, although application-consistent backups may require pausing writes.

### Example: Moving EBS Data Between AZs

1. Create a snapshot of an EBS volume in `us-east-1a`.
2. Create a new volume from the snapshot in `us-east-1b`.
3. Attach the new volume to an EC2 instance in `us-east-1b`.

### Snapshot Features

#### Snapshot Archive

Lower-cost storage for rarely accessed snapshots.

- Can reduce storage costs by up to approximately 75%.
- Designed for long-term retention.
- Archived snapshots must be restored before use.
- Standard restoration typically takes 24–72 hours.

#### Recycle Bin

Protects snapshots against accidental deletion.

- Retains deleted snapshots through retention rules.
- Supports retention periods from 1 day to 1 year.
- Allows snapshots to be recovered before the retention period expires.

#### Fast Snapshot Restore (FSR)

Provides faster access to volumes created from snapshots.

- Eliminates initial block-read latency caused by lazy initialization.
- Provides fully initialized volumes immediately.
- Incurs additional charges.

---

## 4. Amazon Machine Image (AMI)

An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances with predefined software and configurations.

An AMI can include:

- Operating system.
- Installed software and dependencies.
- System configurations.
- Monitoring software.
- Storage configuration and block device mappings.

### Benefits

- Faster EC2 deployment.
- Consistent configurations.
- Reusable server environments.
- Reduced manual setup.

AMIs are Region-specific but can be copied across Regions.

### AMI Types

| Type | Description |
|---|---|
| AWS-provided AMI | Public images maintained by AWS |
| Custom AMI | Images created and maintained by users |
| AWS Marketplace AMI | Third-party images, sometimes requiring payment |

### Example

Instead of installing Node.js and dependencies manually on every EC2 instance, create a custom AMI containing those configurations.

New instances launched from that AMI already include the configured software.

**AMI vs. EBS Snapshot:**

- **AMI:** Template for launching EC2 instances.
- **EBS Snapshot:** Backup of an EBS volume.

An EBS-backed AMI references snapshots containing the instance's disk data.

---

## 5. EC2 Instance Store

**EC2 Instance Store** provides temporary block storage physically attached to the host running an EC2 instance.

It is designed for workloads requiring high disk I/O performance.

### Key Characteristics

- High-performance local storage.
- Low latency for suitable workloads.
- Temporary (**ephemeral**) storage.
- Data is lost when the instance stops, terminates, or the underlying host fails.
- Data normally survives an instance reboot.
- Backup and replication are the user's responsibility.

### Common Use Cases

- Caching.
- Temporary files.
- Buffers.
- Scratch data.
- Intermediate processing.

### EBS vs. Instance Store

| Feature | EBS | Instance Store |
|---|---|---|
| Storage | Network-attached | Local host storage |
| Persistence | Persistent | Ephemeral |
| EC2 stop | Data preserved | Data lost |
| Detachable | Yes | No |
| Snapshots | Supported | Not directly supported |
| Typical use | OS, applications, databases | Cache, temporary data |

**Key concept:** Use EBS for persistent storage and Instance Store for high-performance temporary storage.

---

## 6. EBS Volume Types

Amazon EBS offers six main volume types grouped into SSD and HDD categories.

### Storage Performance Metrics

- **Size:** Storage capacity measured in GiB or TiB.
- **IOPS:** Number of input/output operations performed per second.
- **Throughput:** Amount of data transferred per second, measured in MiB/s.
- **Latency:** Time required to complete a storage operation.

### Volume Types Overview

| Type | Storage | Main Use Case |
|---|---|---|
| gp3 | SSD | General-purpose workloads |
| gp2 | SSD | General-purpose workloads |
| io2 Block Express | SSD | High-performance databases |
| io1 | SSD | Provisioned IOPS workloads |
| st1 | HDD | Throughput-intensive workloads |
| sc1 | HDD | Infrequently accessed data |

**Boot volumes:** SSD types support EC2 boot volumes. HDD types (st1/sc1) do not.

### 6.1 General Purpose SSD (gp2 / gp3)

Balanced performance and cost for general applications.

**Use cases:**
- EC2 boot volumes.
- Development and testing.
- Virtual desktops.
- Medium-sized databases.
- Transactional workloads.

#### gp3

- Storage: 1 GiB – 64 TiB.
- Baseline: 3,000 IOPS.
- Baseline throughput: 125 MiB/s.
- IOPS and throughput can be configured independently of storage size.
- Maximum: 80,000 IOPS and 2,000 MiB/s, subject to applicable limits.

#### gp2

- Storage: 1 GiB – 16 TiB.
- Performance scales with volume size.
- Baseline: 3 IOPS per GiB, minimum 100 IOPS.
- Smaller volumes can burst to 3,000 IOPS.
- Maximum: 16,000 IOPS.

**Key difference:** gp3 allows independent performance configuration, while gp2 performance depends mainly on storage size.

### 6.2 Provisioned IOPS SSD (io1 / io2)

Designed for applications requiring high and consistent I/O performance.

**Use cases:**
- Critical business applications.
- High-performance databases.
- Transaction-intensive workloads.
- Applications requiring predictable latency.

#### io1

- Storage: 4 GiB – 16 TiB.
- Maximum: 64,000 IOPS on supported Nitro instances.
- Maximum: 32,000 IOPS on other supported instances.
- IOPS can be configured independently.
- Maximum IOPS-to-GiB ratio: 50:1.

#### io2 Block Express

- Storage: 4 GiB – 64 TiB.
- Sub-millisecond latency.
- Maximum: 256,000 IOPS.
- Maximum IOPS-to-GiB ratio: 1,000:1.
- Optimized for demanding database workloads.

### 6.3 Hard Disk Drives (HDD)

Lower-cost storage optimized for throughput-intensive workloads rather than high random IOPS.

HDD volumes cannot be used as boot volumes.

#### Throughput Optimized HDD (st1)

Designed for frequently accessed sequential workloads.

**Use cases:**
- Big Data.
- Data warehouses.
- Log processing.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 500 MiB/s.
- Maximum IOPS: 500.

#### Cold HDD (sc1)

Designed for infrequently accessed data.

**Use cases:**
- Infrequently accessed datasets.
- Low-cost storage.
- Workloads with limited performance requirements.

**Specifications:**
- Storage: 125 GiB – 16 TiB.
- Maximum throughput: 250 MiB/s.
- Maximum IOPS: 250.

---

## 7. Amazon EFS (Elastic File System)

**Amazon EFS** is a fully managed, scalable network file system that allows multiple EC2 instances to access shared files simultaneously.

Unlike EBS, which provides block storage, EFS provides **shared file storage** accessible through the network.

### Key Characteristics

- **Managed NFS:** Uses the Network File System (NFSv4.1) protocol.
- **Multi-AZ:** Regional EFS file systems can be accessed by instances across multiple Availability Zones.
- **Shared Storage:** Multiple EC2 instances can mount the same file system simultaneously.
- **Automatic Scaling:** Storage grows and shrinks as files are added or removed.
- **High Availability:** Regional file systems store data redundantly across multiple AZs.
- **Pay Per Use:** Storage costs are based primarily on data stored, with additional charges depending on storage class and access patterns.
- **Linux Compatibility:** Designed for Linux-based workloads and POSIX-compatible file access.

### Security

- **Security Groups:** Control network access to EFS mount targets.
- **AWS KMS:** Supports encryption at rest.
- **Encryption in Transit:** Protects data transferred between EC2 and EFS.
- **POSIX Permissions:** Supports standard Linux file permissions and ownership.

### Common Use Cases

- Content management systems (CMS).
- Web serving.
- Shared application files.
- WordPress installations.
- Data sharing between EC2 instances.
- Applications requiring shared directories.

### Example: Multiple EC2 Instances Sharing Files

Consider a WordPress application running on two EC2 instances:

- EC2 Instance A serves website requests.
- EC2 Instance B serves additional website requests.
- Both instances mount the same EFS file system.

When a user uploads an image through Instance A, the file becomes available to Instance B because both access the same shared storage.

**Key benefit:** EFS allows multiple servers to share files without manually synchronizing them.

---

## 8. EFS Performance and Throughput

Amazon EFS is designed to scale automatically as storage and application demand increase.

### EFS Scalability

- Supports thousands of concurrent NFS clients.
- Can scale to petabytes of storage.
- Supports high aggregate throughput, depending on configuration and service quotas.
- Automatically adjusts storage capacity based on stored files.

### 8.1 Performance Modes

Performance modes determine the latency and parallel-processing characteristics of an EFS file system.

| Mode | Description | Use Case |
|---|---|---|
| General Purpose | Low latency for most applications | Web servers, CMS, shared files |
| Max I/O | Higher aggregate parallelism with increased latency | Big Data, media processing |

**General Purpose** is the standard performance mode.

**Note:** Max I/O is a legacy performance mode and is not available for new EFS file systems.

### 8.2 Throughput Modes

Throughput determines how quickly data can be read from and written to EFS.

#### Bursting Throughput

- Throughput scales with the amount of stored data.
- Uses burst credits for temporary increases in performance.
- Suitable for workloads with variable throughput requirements.

#### Provisioned Throughput

- Allows configuring throughput independently of storage capacity.
- Useful when applications require sustained throughput with relatively little stored data.
- Charges are based on provisioned throughput.

#### Elastic Throughput

- Automatically scales throughput according to workload demand.
- No need to provision throughput manually.
- Suitable for unpredictable or frequently changing workloads.

**Key difference:**

- **Bursting:** Performance depends on storage size and available burst credits.
- **Provisioned:** Throughput is configured manually.
- **Elastic:** Throughput automatically adjusts to application demand.

---

## 9. EFS Storage Classes

Amazon EFS provides storage classes to optimize costs based on how frequently files are accessed.

### Storage Classes

| Storage Class | Description | Use Case |
|---|---|---|
| EFS Standard | Frequently accessed files | Active applications |
| EFS Infrequent Access (IA) | Less frequently accessed files | Older or occasionally accessed data |
| EFS Archive | Rarely accessed files | Long-term retention |

### EFS Standard

- Designed for frequently accessed files.
- Provides low-latency access.
- Appropriate for active application workloads.

### EFS Infrequent Access (IA)

- Lower storage cost than Standard.
- Additional charges when accessing stored data.
- Suitable for files that are accessed occasionally.

### EFS Archive

- Designed for data accessed only a few times per year.
- Offers lower storage costs.
- Additional data-access charges apply.

### EFS Lifecycle Management

**Lifecycle Management** automatically moves files between storage classes based on configured policies.

Example:

1. A file is uploaded to EFS Standard.
2. The file remains unused for 60 days.
3. A lifecycle policy moves it to EFS Infrequent Access.
4. Storage costs decrease.

Lifecycle policies can also transition eligible files to Archive and return files to Standard when accessed, depending on the configuration.

**Benefit:** Reduces costs without manually moving files.

---

## 10. EFS Availability and Durability

Amazon EFS provides two main file system deployment options.

### Regional EFS

- Stores data redundantly across multiple Availability Zones.
- Designed for high availability and durability.
- Suitable for production applications.
- Allows EC2 instances across AZs to access shared files.

### EFS One Zone

- Stores data within a single Availability Zone.
- Lower storage cost.
- Suitable for development, testing, or workloads that can tolerate AZ-level data loss.
- Has less resilience than Regional EFS.

**Important:** One Zone storage is not replicated across Availability Zones.

---

## 11. EBS vs. EFS vs. Instance Store

These three services provide different approaches to EC2 storage.

| Feature | EBS | EFS | Instance Store |
|---|---|---|---|
| Storage Type | Block | File | Block |
| Connection | Network | Network (NFS) | Local |
| Persistence | Persistent | Persistent | Temporary |
| Multiple EC2 Access | Limited Multi-Attach | Yes | No |
| Multi-AZ Access | No | Yes (Regional) | No |
| Automatic Storage Scaling | No | Yes | No |
| Typical Performance | Configurable IOPS | Shared file access | High local I/O |
| Primary Use Case | OS, databases, application data | Shared application files | Cache, temporary data |

### When to Use Each Service

**Choose EBS when:**
- An EC2 instance needs persistent disk storage.
- Running operating systems or databases.
- High IOPS or consistent block-storage performance is required.

**Choose EFS when:**
- Multiple EC2 instances need access to the same files.
- Applications run across multiple Availability Zones.
- Shared storage must scale automatically.
- Running shared web content or CMS applications.

**Choose Instance Store when:**
- High-performance temporary storage is required.
- Data can be recreated if lost.
- Running caches, buffers, or temporary processing workloads.

---

## 12. Key Concepts to Remember

- **EBS:** Persistent block storage attached to EC2.
- **EBS Multi-Attach:** Allows up to 16 compatible EC2 instances in one AZ to share an io1/io2 volume.
- **Delete on Termination:** Controls EBS deletion when EC2 terminates.
- **EBS Snapshots:** Point-in-time volume backups.
- **AMI:** Template for launching configured EC2 instances.
- **Instance Store:** Temporary, high-performance local storage.
- **EFS:** Managed, shared network file storage using NFS.
- **EFS Regional:** Shared file storage replicated across multiple AZs.
- **EFS One Zone:** Lower-cost file storage in one AZ.
- **EFS Lifecycle Management:** Automatically transitions files between storage classes.
- **EFS Elastic Throughput:** Automatically scales throughput to match workload demand.
- **gp2/gp3:** General-purpose SSD EBS volumes.
- **io1/io2:** High-performance Provisioned IOPS SSD volumes.
- **st1:** Throughput-optimized HDD.
- **sc1:** Cold HDD for infrequently accessed data.

**Core distinction:**
- **EBS = Persistent virtual hard drive.**
- **EFS = Shared network file system.**
- **Instance Store = Temporary local disk.**
- **AMI = EC2 instance template.**