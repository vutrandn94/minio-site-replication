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