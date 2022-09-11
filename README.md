# README

## Requirements

- Docker/ Podman
- docker-compose

## Quickstart

### Build all images using Docker

```shell
# Source environment variables
source .env

# Build nfs image
docker build \
  --tag ${REGISTRY}/jupyter_nfs \
  ./nfs

# Build proxy image
docker build \
  --tag ${REGISTRY}/jupyter_proxy \
  ./traefik

# Build jupyterhub image
docker build \
  --tag ${REGISTRY}/jupyter_jupyterhub \
  --env MINICONDA_VERSION=${MINICONDA_VERSION} \
  --env R_VERSION=${R_VERSION} \
  --env NFS_SERVER=${NFS_SERVER} \
  ./jupyterhub
```

### Run

- Go to the main directory and run `docker-compose up -d` to start all
containers in detached mode.

- Create test user `jupyter` with password `jupyter`

```shell
docker exec jupyter_jupyterhub sh -c "useradd jupyter"
docker exec jupyter_jupyterhub sh -c "echo \"jupyter:jupyter\" | sudo chpasswd"
```

- Go to: <https://localhost/jupyterhub/>
