# Ansible Builder Repository

Network automation execution environment for AWX, built on top of the EE that ships with AWX 24.6.1.

| Component    | Version                                   |
| ------------ | ----------------------------------------- |
| Base image   | `quay.io/ansible/awx-ee:24.6.1`           |
| OS           | CentOS Stream 9                           |
| Python       | 3.9 (`/usr/bin/python3`)                  |
| ansible-core | 2.15.12                                   |
| Runner       | 2.4.0                                     |

The image adds `netmiko` and `ansible-pylibssh` plus the network collections in `requirements.yml`.
Everything else (Python, ansible-core, ansible-runner, paramiko, git-lfs, sshpass, subversion, `ansible.posix`) comes from the base image.

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

# Step 3: Push to registry
docker push zbiles/awx-network-ee:<tag>
```

### Smoke test

```shell
docker run --rm --platform linux/amd64 zbiles/awx-network-ee:<tag> bash -c "
  ansible --version | head -1
  python3 -c 'import netmiko, pylibsshext; print(netmiko.__version__)'
  ansible-galaxy collection list
  git config --system --get filter.lfs.process
"
```

## Customizing the Image

- **Collections** (`requirements.yml`): versions are pinned to the newest releases that still support ansible-core 2.15.
  Most current releases require ansible-core >= 2.16, and `ansible-galaxy` does not skip incompatible versions, so check
  `requires_ansible` on Galaxy before bumping a pin.
- **Python packages** (`requirements.txt`): must support Python 3.9 (for example, netmiko 4.7+ requires Python 3.10).
- **System packages** (`bindep.txt`): RPM names only; bindep only installs what the base image is missing.

When AWX is upgraded, move `images.base_image` to the matching `awx-ee` tag, re-check its Python and ansible-core versions,
and revisit the pins above.
