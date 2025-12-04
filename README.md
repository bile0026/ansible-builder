# Ansible Builder Repository

## Setup Environment

```shell
python -m venv builder
source builder/bin/activate

pip install -r requirements_development.txt
pip install --upgrade pip
```

## Build the container image

Build the execution environment using the standard files (`execution-environment.yml`, `requirements.txt`, etc.):

**For Apple Silicon (M1/M2) Macs** - To force AMD64 builds (recommended for AWX which typically runs on AMD64):

```shell
# Step 1: Generate the build context (this will also attempt a build, but we'll rebuild with the correct platform)
ansible-builder build --tag zbiles/awx-base-ee:<tag> --context ./context --container-runtime docker

# Step 2: Rebuild with AMD64 platform using docker buildx
docker buildx build --platform linux/amd64 -f ./context/Dockerfile -t zbiles/awx-base-ee:<tag> ./context --load

# Step 3: Push to registry
docker push zbiles/awx-base-ee:<tag>
```

**For regular builds (native platform):**

```shell
ansible-builder build --tag zbiles/awx-base-ee:<tag> --context ./context --container-runtime docker

docker push zbiles/awx-base-ee:<tag>
```

### Customizing the Image

Modify `requirements.txt` and `requirements.yml` to add/remove Python packages and Ansible collections
