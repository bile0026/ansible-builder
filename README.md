# Ansible Builder Repository

## Setup Environment

```shell
python -m venv builder
source builder/bin/activate

pip install -r requirements_development.txt
pip install --upgrade pip
```

## Build the container image

To re-build and push to docker hub. Make sure to update the tag.

```shell
# build image
ansible-builder build --tag zbiles/awx-base-ee:<tag> --context ./context --container-runtime docker

# Login to docker hub (if required)
docker login

# push the image to docker hub
docker push zbiles/awx-base-ee:<tag>
```

The requirements files can be modified and the image/tag changed to build EE's with different prerequisites installed.
