# minio-site-replication
Deploy MinIO: Site Replication

![Alt Text](site-replicate-diagram.png)

## Requirement
- Format disk XFS for high performance
- Minimum: 2 nodes (2 servers)
- Deployment environment: Docker
- Deployment tools: Docker Compose


## Information about the servers deploying the lab

| Hostname | IP Address | Description |
| :--- | :--- | :--- |
| minio01 | 172.31.47.91 | |
| minio02 | 172.31.37.11 | |
| minio03 | 172.31.36.54 | Add Later |

## Deploy
**Default minio admin user & password (Change if necessary)**
| Default Root User | Default Root Password |
| :--- | :--- |
| root | Enjoyd@y |

**Default container Timezone (Modify to appropriate timezone)**
| Default container Timezone |
| :--- |
| Asia/Ho_Chi_Minh |

**Default docker volume mount path**
| Default volume mount path |
| :--- |
| /mnt/data | 

**Default MinIO docker image**
| Default MinIO docker image |
| :--- |
| quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z |

**Setting hosts file (Set on all nodes)**
```
## MINIO
172.31.47.91 minio01 
172.31.37.11 minio02 
172.31.36.54 minio03
```