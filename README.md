# Ansible Builder Repository

To re-build and push to docker hub. Make sure to update the tag. 

```shell
ansible-builder build --tag zbiles/awx-base-ee:<tag> --context ./context --container-runtime docker

docker push zbiles/awx-base-ee:<tag>
```

The requirements files can be modified and the image/tag changed to build EE's with different prerequisites installed. 


