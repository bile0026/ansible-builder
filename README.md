# Ansible Builder Repository

Network automation execution environment for AWX 24.6.1, built the same way as upstream
[awx-ee](https://github.com/ansible/awx-ee) (`devel`).

| Component      | Version                          |
| -------------- | -------------------------------- |
| Base image     | `quay.io/centos/centos:stream9`  |
| Python         | 3.12 (`/usr/bin/python3.12`)     |
| ansible-core   | 2.18.x (held below 2.19)         |
| ansible-runner | 2.4.x                            |

The image contains ansible-core, ansible-runner, the network collections in `requirements.yml`,
and the Python packages in `requirements.txt` (netmiko, ansible-pylibssh, paramiko, ncclient, ...).

ansible-core is held below 2.19, matching upstream awx-ee: 2.19 reworked templating ("data tagging"),
which can break existing playbooks.

## Versions

| Tag      | Base                             | Python | ansible-core |
| -------- | -------------------------------- | ------ | ------------ |
| `3.0.0`  | `quay.io/centos/centos:stream9`  | 3.12   | 2.18.x       |
| `2.0.0`  | `quay.io/ansible/awx-ee:24.6.1`  | 3.9    | 2.15.12      |

`latest` points to the newest release that has been tested in AWX. Reference versioned tags
(e.g. `zbiles/awx-network-ee:3.0.0`) in AWX execution environments so an image push never changes
running jobs unexpectedly.

## Setup Environment

```shell
python3 -m venv builder
source builder/bin/activate

pip install --upgrade pip "ansible-builder==3.1.1"
```

## Build the container image

AWX execution nodes run AMD64, so build for that platform explicitly (required on Apple Silicon):

```shell
# Step 1: Generate the build context
ansible-builder create -c ./context --output-filename Dockerfile

# Step 2: Build for AMD64
docker buildx build --platform linux/amd64 -f context/Dockerfile -t zbiles/awx-network-ee:<tag> context --load

# Step 3: Push the versioned tag
docker push zbiles/awx-network-ee:<tag>
```

### Smoke test

```shell
docker run --rm --platform linux/amd64 zbiles/awx-network-ee:<tag> bash -c "
  python --version
  ansible --version | head -1
  ansible-runner --version
  python -c 'import netmiko, pylibsshext, ncclient, jmespath; print(netmiko.__version__)'
  ansible-galaxy collection list
  git config --system --get filter.lfs.process
"
```

### Promote to `latest`

After the versioned image has passed a real job run in AWX:

```shell
docker tag zbiles/awx-network-ee:<tag> zbiles/awx-network-ee:latest
docker push zbiles/awx-network-ee:latest
```

## Customizing the Image

- **Collections** (`requirements.yml`): pinned to exact versions. `ansible-galaxy` does not skip releases
  that need a newer ansible-core, so check `requires_ansible` on Galaxy before bumping a pin
  (it must allow ansible-core 2.18).
- **Python packages** (`requirements.txt`): must support Python 3.12.
- **System packages** (`bindep.txt`): RPM names only; `[compile]` packages are only used during the build.

When moving to ansible-core 2.19 or later, test existing playbooks against the new templating behavior first.
