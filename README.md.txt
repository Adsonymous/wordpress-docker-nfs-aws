## 🌍 Project Overview

This project demonstrates a **production-like WordPress deployment** on AWS using **Docker**, **MySQL**, and **NFS** for persistent storage.  
The application is accessible via the domain: **[cyruslifc.online](http://cyruslifc.online)**.

## Key Details

- WordPress version: 6.8.2
- MySQL version: latest
- Domain: cyruslifc.online
- Ports open on Docker host / EC2:
  - HTTP: 80
  - SSH: 22
  - DNS: 53 (mapped)
  - NFS: 2049 (default NFS port)

- **MySQL Ports (3306, 33060)**: Not exposed publicly. Accessible only within Docker network for security.

## Features / Highlights

- WordPress + MySQL running in Docker containers
- Persistent storage using NFS for both WordPress files and MySQL database
- Easy deployment using docker-compose
- Infrastructure scripts included to setup NFS server and mount volumes
- Demo screenshots and sample database dump included

🏗️ Architecture

This project's architecture uses Docker's native volume management capabilities to connect directly to an NFS server. This approach is cleaner as the Docker engine itself handles the NFS mounting, making the setup more portable and managed entirely through the docker-compose file.

NFS Server: A dedicated EC2 instance exports two directories (/wp-data and /db-data) over the network.

Docker Host: This EC2 instance runs the Docker engine. It does not manually mount the NFS shares. Instead, it requires the NFS client utility so that the Docker engine can use it.

Docker NFS Volumes: Within the docker-compose.yml file, we define two named volumes (wp-vol and db-vol). We instruct Docker to use the local driver with specific options that tell it to mount the NFS shares. Docker then manages these volumes, making them available to the containers. The actual data is stored on the NFS server at the paths you specified (/var/lib/docker/volumes/.../_data on the host is just the Docker-managed mount point).

This design ensures data persistence and is tightly integrated with Docker's ecosystem.

Plaintext

                           ┌───────────────────┐
                           │    NFS Server     │
                           │   (EC2 Instance)  │
                           │ Exports: /wp-data │
                           │         /db-data  │
                           └─────────┬─────────┘
                                     │ (NFS Protocol)
                                     │
┌────────────────────────────────────┼───────────────────────────────────┐
│                        Docker Host (EC2 Instance)                      │
│                                                                        │
│  ┌─────────────────────────┐         ┌─────────────────────────┐       │
│  │   Docker Volume (wp-vol)│         │   Docker Volume (db-vol)│       │
│  │ Driver: NFS             │         │ Driver: NFS             │       │
│  └──────────┬──────────────┘         └──────────┬──────────────┘       │
│             │                                   │                      │
│   ┌─────────▼────────────┐           ┌─────────▼────────────┐          │
│   │      WordPress       │           │        MySQL         │          │
│   │   /var/www/html      ├<──────────┤   /var/lib/mysql     │          │
│   └──────────────────────┘           └──────────────────────┘          │
│         (Container)                        (Container)                 │
└────────────────────────────────────────────────────────────────────────┘

Key Components
WordPress Container: Runs the WordPress application. Its /var/www/html directory is mapped to the Docker-managed volume named wp-vol. This volume is configured to directly mount the /wp-data share from the NFS server, ensuring all themes, plugins, and uploads are saved on the persistent network storage.

MySQL Container: Runs the MySQL database server. Its /var/lib/mysql data directory is mapped to the db-vol Docker volume. This volume connects directly to the /db-data share on the NFS server, ensuring the entire database is securely stored and persists across container restarts.

Docker NFS Volumes (wp-vol, db-vol): These are the core of the storage architecture. Defined in the docker-compose.yml file, they use Docker's native volume driver capabilities to mount the remote NFS shares. They act as the crucial link between the containers and the persistent storage, centralizing the storage configuration within the application's deployment definition.

Docker Network (app-network): A private virtual network created by Docker Compose. Both containers are attached to this network, allowing the WordPress container to communicate with the MySQL container securely by its service name (mysql), without exposing the database port to the public internet.
