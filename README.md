# Ansible Builder Repository

## Setup Environment

```shell
python -m venv builder
source builder/bin/activate

pip install ansible
pip install ansible-builder
```

## Build the container image

To re-build and push to docker hub. Make sure to update the tag.

```shell
ansible-builder build --tag zbiles/awx-network-ee:<tag> --context ./context --container-runtime docker

docker push zbiles/awx-network-ee:<tag>
```

The requirements files can be modified and the image/tag changed to build EE's with different prerequisites installed.
