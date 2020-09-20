<!-- markdownlint-disable -->
# kubernetes-up-and-running

## Prerequisite:
* Check that docker is installed and running.
```
docker version
```
* Check that kubernetes is stalled and runnning.
    * kuberneties is installed by following this documentation.
    https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview

    * created alias in .bashrc file.

    * ```alias kctl='microk8s kubectl'```
    * this version command should return client and server version without any error
    * ```kctl version```
