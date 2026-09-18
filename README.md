# Ansible Builder Repository

General-purpose execution environment for AWX, built on top of the EE that ships with AWX 24.6.1.

| Component      | Version                          |
| -------------- | -------------------------------- |
| Base image     | `quay.io/ansible/awx-ee:24.6.1`  |
| OS             | CentOS Stream 9                  |
| Python         | 3.9 (`/usr/bin/python3`)         |
| ansible-core   | 2.15.12                          |
| ansible-runner | 2.4.0                            |

The image adds `textfsm` plus `community.general`, `community.docker`, and `networktocode.nautobot`.
Everything else (Python, ansible-core, ansible-runner, jmespath, git-lfs, sshpass, subversion, `ansible.posix`)
comes from the base image.

## Versions

| Image tag | Git tag       | Base                            | Python | ansible-core |
| --------- | ------------- | ------------------------------- | ------ | ------------ |
| `2.0.0`   | `base-v2.0.0` | `quay.io/ansible/awx-ee:24.6.1` | 3.9    | 2.15.12      |

`latest` points to the newest release that has been tested in AWX. Reference versioned tags
(e.g. `zbiles/awx-base-ee:2.0.0`) in AWX execution environments so an image push never changes
running jobs unexpectedly.

## Setup Environment

```shell
python3 -m venv builder
source builder/bin/activate

pip install --upgrade pip
pip install -r requirements_development.txt
```

## Build the container image

AWX execution nodes run AMD64, so build for that platform explicitly (required on Apple Silicon):

```shell
# Step 1: Generate the build context
ansible-builder create -c ./context --output-filename Dockerfile

# Step 2: Build for AMD64
docker buildx build --platform linux/amd64 -f context/Dockerfile -t zbiles/awx-base-ee:<tag> context --load

# Step 3: Push the versioned tag
docker push zbiles/awx-base-ee:<tag>
```

### Smoke test

```shell
docker run --rm --platform linux/amd64 zbiles/awx-base-ee:<tag> bash -c "
  ansible --version | head -1
  python3 -c 'import textfsm, jmespath; print(\"python deps ok\")'
  ansible-galaxy collection list
  git config --system --get filter.lfs.process
"
```

### Promote to `latest`

```shell
docker tag zbiles/awx-base-ee:<tag> zbiles/awx-base-ee:latest
docker push zbiles/awx-base-ee:latest
```

## Customizing the Image

- **Collections** (`requirements.yml`): versions are pinned to the newest releases that still support ansible-core 2.15.
  Most current releases require ansible-core >= 2.16, and `ansible-galaxy` does not skip incompatible versions, so check
  `requires_ansible` on Galaxy before bumping a pin.
- **Python packages** (`requirements.txt`): must support Python 3.9.
- **System packages** (`bindep.txt`): RPM names only; bindep only installs what the base image is missing.

When AWX is upgraded, move `images.base_image` to the matching `awx-ee` tag, re-check its Python and ansible-core versions,
and revisit the pins above.
