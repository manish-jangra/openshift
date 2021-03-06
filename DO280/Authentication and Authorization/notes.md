#### Authentication and Authorization
    oc auth can-i verb resource

#### Creating an HTPasswd  File
    htpasswd -c -B -b /tmp/htpasswd student redhat123
>**Important**
> Use the -c option only when creating a new file. The -c option replaces all file content if the file already exists.

#### Add or update credentials
    htpasswd -b /tmp/htpasswd student redhat1234

#### Delete credentials
    htpasswd -D /tmp/htpasswd student

#### Creating the HTPasswd Secret
    oc create secret generic htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config

#### Updating the OAuth Custom Resource
    oc get oauth cluster -o yaml > oauth.yaml
    
#### Modify oauth.yaml file
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
```    
#### update the oauth
    oc replace -f oauth.yaml

#### Check the status of pods after changing oauth
    watch oc get pods -n openshift-authentication

#### Extracting Secret Data
    oc extract secret/htpasswd-secret -n openshift-config --to /tmp/ --confirm /tmp/htpasswd

#### Updating the HTPasswd Secret
    oc set data secret/htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config

#### Deleting Users and Identities
    oc extract secret/htpasswd-secret -n openshift-config --to /tmp/ --confirm /tmp/htpasswd

    htpasswd -D /tmp/htpasswd manager

    oc set data secret/htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config

    oc delete user manager

    oc get identities | grep manager

    oc delete identity my_htpasswd_provider:manager

#### Assigning Administrative Privileges
    oc adm policy add-cluster-role-to-user cluster-admin student

#### Delete secret created from htpasswd file, all users and their identities
    oc delete secret htpasswd-secret -n openshift-config

    oc delete user --all

    oc delete identity --all


## RBAC (Role Based Access Control)

#### Default Roles

| Default Roles            | Description                                                                                                                         |
| -------------------------| ------------------------------------------------------------------------------------------------------------------------------------|
| admin                    | Users with this role can manage all project resources, including granting access to other users to access the project               |
| basic-user               | Users with this role have read access to the project.                                                                               |
| cluster-admin            | Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster              |
| cluster-status           | Users with this role can get cluster status information.                                                                            |
| edit                     | Users with this role can create, change, and delete common application resources but not management resources                       |
| self-provisioner         | Users with this role can create new projects. This is a cluster role, not a project role.                                           |
| view                     | Users with this role can view project resources, but cannot modify project resources.                                               |

### Managing RBAC Using the CLI
#### Add a cluster role to a user:
    oc adm policy add-cluster-role-to-user cluster-role username

#### Remove a cluster role from a user:
    oc adm policy remove-cluster-role-from-user cluster-role username

#### Add a specified role to a user at project level
    oc policy add-role-to-user role-name username -n project

#### Remove self-provisioners cluster role from a group called authenticated users
    oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

#### Create a user
    oc adm groups new group-name

#### Add user to a group
    oc adm groups add-users group-name username

#### Get groups
    oc get groups

#### Add role to a group at project level
    oc policy add-role-to-group role-name group-name -n project

#### Get role binding for a project
    oc get rolebindings -o wide -n project

#### Add self-provisioners role to system:authenticated:oauth
    oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth
