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
| root | Enjoyday |

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

## Add site replication (2 node minio01 - minio02)
**docker-compose.yml in minio01 (Config in node "minio01")**
```
services:
  minio01:
    image: 'quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z'
    restart: always
    environment:
      MINIO_ROOT_USER: "root"
      MINIO_ROOT_PASSWORD: "Enjoyday"
      TZ: "Asia/Ho_Chi_Minh"
      MC_HOST_minio01: "http://root:Enjoyday@minio01:9000"
      MC_HOST_minio02: "http://root:Enjoyday@minio02:9000"
      MC_HOST_minio03: "http://root:Enjoyday@minio03:9000"
    command: server /data --console-address ":9001"
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - /mnt/data:/data
    networks:
      - minio-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 1m
networks:
  minio-net:
    driver: bridge
```

**docker-compose.yml in minio02 (Config in node "minio02")**
```
services:
  minio02:
    image: 'quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z'
    restart: always
    environment:
      MINIO_ROOT_USER: "root"
      MINIO_ROOT_PASSWORD: "Enjoyday"
      TZ: "Asia/Ho_Chi_Minh"
      MC_HOST_minio01: "http://root:Enjoyday@minio01:9000"
      MC_HOST_minio02: "http://root:Enjoyday@minio02:9000"
      MC_HOST_minio03: "http://root:Enjoyday@minio03:9000"
    command: server /data --console-address ":9001"
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - /mnt/data:/data
    networks:
      - minio-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 1m
networks:
  minio-net:
    driver: bridge
```

**Deploy service (Execute on all nodes)**
```
# docker-compose up -d
```

**Add site replication (Execute on minio01 or minio02)**
```
root@minio01:~/minio-deploy# docker ps
CONTAINER ID   IMAGE                                              COMMAND                  CREATED              STATUS                        PORTS                              NAMES
b47c3db88b69   quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z   "/usr/bin/docker-ent…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:9000-9001->9000-9001/tcp   minio-deploy-minio01-1
```
```
root@minio01:~/minio-deploy# docker exec -it minio-deploy-minio01-1 bash
```
```
bash-5.1# mc admin replicate add minio01 minio02
mc: Configuration written to `/tmp/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/tmp/.mc/share`.
mc: Initialized share uploads `/tmp/.mc/share/uploads.json` file.
mc: Initialized share downloads `/tmp/.mc/share/downloads.json` file.
Requested sites were configured for replication successfully.
```
**Site replication configuration verify**
```
bash-5.1# mc admin replicate info  minio01
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint                                       | Sync | Bandwidth  | ILM Expiry Replication   
                                     |                 |                                                |      | Per Bucket |                          
4bc1ff09-2783-4f04-9f07-3f27ed43f66f | minio01         | http://minio01:9000                            |      | N/A        | false                    
8a3cade1-8ada-4bdd-a7a3-c1c082e568c3 | minio02         | http://minio02:9000                            |      | N/A        | false                    
```
```
bash-5.1# mc admin replicate info  minio02
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint                                       | Sync | Bandwidth  | ILM Expiry Replication   
                                     |                 |                                                |      | Per Bucket |                          
4bc1ff09-2783-4f04-9f07-3f27ed43f66f | minio01         | http://minio01:9000                            |      | N/A        | false                    
8a3cade1-8ada-4bdd-a7a3-c1c082e568c3 | minio02         | http://minio02:9000                            |      | N/A        | false                    
```
**Check replicate status in node minio01**
```
bash-5.1# mc admin replicate status minio01
Bucket replication status:
No Buckets present

Policy replication status:
●  5/5 Policies in sync

User replication status:
No Users present

Group replication status:
No Groups present

ILM Expiry Rules replication status:
No ILM Expiry Rules present

Object replication status:
Replication status since 8 minutes 
Summary:
Replicated:    0 objects (0 B)
Queued:        ● 0 objects, (0 B) (avg: 0 objects, 0 B; max: 0 objects, 0 B)
Received:      0 objects (0 B)
```
**Check replicate status in node minio02**
```
bash-5.1# mc admin replicate status minio02
Bucket replication status:
No Buckets present

Policy replication status:
●  5/5 Policies in sync

User replication status:
No Users present

Group replication status:
No Groups present

ILM Expiry Rules replication status:
No ILM Expiry Rules present

Object replication status:
Replication status since 8 minutes 
Summary:
Replicated:    0 objects (0 B)
Queued:        ● 0 objects, (0 B) (avg: 0 objects, 0 B; max: 0 objects, 0 B)
Received:      0 objects (0 B)
```

**Test replicate**
- **Create bucket in node minio01/minio02 and check the remaining node**

![Alt Text](bucket-replicate-1.png)

![Alt Text](bucket-replicate-2.png)

- **Upload item object in node minio01/minio02 and verify object metadata in remaining node**

![Alt Text](object-replicate-1.png)

![Alt Text](metric.png)


## (Optinal) Add site replication to expand (add node minio03)
> [!NOTE]
> Node need to add more to expand replicate size, there must be no data (Empty data)

**Execute on minio01 or minio02**
```
root@minio01:~/minio-deploy# docker ps
CONTAINER ID   IMAGE                                              COMMAND                  CREATED          STATUS                    PORTS                              NAMES
b47c3db88b69   quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z   "/usr/bin/docker-ent…"   33 minutes ago   Up 32 minutes (healthy)   0.0.0.0:9000-9001->9000-9001/tcp   minio-deploy-minio01-1
```
```
root@minio01:~/minio-deploy# docker exec -it minio-deploy-minio01-1 bash
```
```
bash-5.1# mc admin replicate add minio01 minio02 minio03
Requested sites were configured for replication successfully.
```
**Site replication configuration verify**
```
bash-5.1# mc admin replicate info  minio01
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint                                       | Sync | Bandwidth  | ILM Expiry Replication   
                                     |                 |                                                |      | Per Bucket |                          
4bc1ff09-2783-4f04-9f07-3f27ed43f66f | minio01         | http://minio01:9000                            |      | N/A        | false                    
8a3cade1-8ada-4bdd-a7a3-c1c082e568c3 | minio02         | http://minio02:9000                            |      | N/A        | false                    
ec02fcd6-0986-4792-8b15-01e43105eae4 | minio03         | http://minio03:9000                            |      | N/A        | false                    
```
```
bash-5.1# mc admin replicate info  minio02
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint                                       | Sync | Bandwidth  | ILM Expiry Replication   
                                     |                 |                                                |      | Per Bucket |                          
4bc1ff09-2783-4f04-9f07-3f27ed43f66f | minio01         | http://minio01:9000                            |      | N/A        | false                    
8a3cade1-8ada-4bdd-a7a3-c1c082e568c3 | minio02         | http://minio02:9000                            |      | N/A        | false                    
ec02fcd6-0986-4792-8b15-01e43105eae4 | minio03         | http://minio03:9000                            |      | N/A        | false                    
```
```
bash-5.1# mc admin replicate info  minio03
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint                                       | Sync | Bandwidth  | ILM Expiry Replication   
                                     |                 |                                                |      | Per Bucket |                          
4bc1ff09-2783-4f04-9f07-3f27ed43f66f | minio01         | http://minio01:9000                            |      | N/A        | false                    
8a3cade1-8ada-4bdd-a7a3-c1c082e568c3 | minio02         | http://minio02:9000                            |      | N/A        | false                    
ec02fcd6-0986-4792-8b15-01e43105eae4 | minio03         | http://minio03:9000                            |      | N/A        | false             
``` 

**Test replicate**

![Alt Text](expand.png)