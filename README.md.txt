## Key Details

- WordPress version: 6.8.2
- MySQL version: latest
- Domain: cyruslifc.online
- Ports open on Docker host / EC2:
  - HTTP: 80
  - SSH: 22
  - DNS: 53 (mapped)
  - NFS: 2049 (default NFS port)

## Features / Highlights

- WordPress + MySQL running in Docker containers
- Persistent storage using NFS for both WordPress files and MySQL database
- Easy deployment using docker-compose
- Infrastructure scripts included to setup NFS server and mount volumes
- Demo screenshots and sample database dump included
