# my old ansible playbook to setup K8S environment
## what does it do
* creates namespaces
* creates infra namaspace
* adds `tiller` sa to the infra ns
* adds `tiller-manager` role to ns
* binds sa and role with `tiller-binding` rolebinding
* installs `helm`
* deploys `tiller` to infra ns
* deploys `gitlab-runner` to infra ns where registry token configured and registers to the belonging gitlab project 

## usage
### inventory
```ini
[kube-cluster-conf]
host1
host2
host3
```
the playbook will be executed on the kubernetes master hosts (manifest files will be applied only in the 1st node of list)

### variables

```yaml
docker_registry_url: "https://iac-registry.corp"
docker_registry_email: <email of the gitab user>
docker_registry_username: <impersonation token name>
docker_registry_token: <impersonation token>

default_gitlab_url: "https://iac-gitlab.corp"
default_gitlab_net: 10.229.230.0/27
default_gitlab_port: 443

helm_url: "https://iac-nexus.corp/repository/kubernetes/helm/helm-v2.10.0-linux-amd64.tar.gz"

projects:
  - name: fooproj
    runner:
      registration_token: <gitlab project CI/CD registration token>
      gitlab_url: "https://iac-gitlab.corp/"
      gitlab_net: 10.229.230.0/27
      gitlab_port: 443
    namespaces:
      - name: "fe"
        labels:
          - key: environment
            value: frontend
          - key: foo
            value: bar
      - name: "be"
        labels:
          - key: environment
            value: backend
      - name: "int"
        labels:
          - key: environment
            value: integration
  - name: barproj
    namespaces:
      - name: "uno"
      - name: "due"
```

---
* docker-registry settings: docker registry connection parameters -- Create technical user and impersonation token with registry read access right
  * ```docker_registry_url```
  * ```docker_registry_email```
  * ```docker_registry_username```
  * ```docker_registry_token```

* ```default_gitlab_net```: default network of gitlab (used for network policy settings)
* ```default_gitlab_port```: default port of gitlab (used for network policy settings)
* ```default_gitlab_url```: default gitlab url
* ```helm_url```: url to download helm

---

* ```projects```: list of projects to configure 
  * ```name```: name of the project
  * ```runner```: gitlab-runner configuration. runners will be installed to the ````<projects.name>_infra``` namespace
    * ```registration_token```: registration token
    * ```gitlab_net```: network of gitlab (for network policy settings). if not specified, default will be used
    * ```gitlab_port```: ip port of gitlab (for network policy settings). if not specified, default will be used
    * ```gitlab_url```: gitlab url. if not specified, default will be used
  * ```namespaces```: namespaces to be created. the namespaces will be created under the name of ```<projects.name>```_<```projects.namespaces.name```>. ```<projects.name>_infra``` namespace will be created by default with tiller. All the service accounts, roles, role bindings will be created. Default deny (except DNS) network policy will be applied
    * ```name```: name of the namespace
    * ```labels```: labels to be assigned to the namespace. by default ```project:<projects.name>```, ```name:<projects.namespaces.name>``` will be assigned to all namaspaces plus ```environment: infra``` in case of infra namespace
      * ```key```: key of the label
      * ```value```: value of the label

---
