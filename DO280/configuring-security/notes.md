### Secrets and Configmaps
#### Create a generic secret containing key-value pairs from literal values
    oc create secret generic secret_name --from-literal key1=secret1 --from-literal key2=secret2

#### Create a generic secret using key names specified on the command line and values from files
    oc create secret generic secret_name \
        --from-file id_rsa=/path-to/id_rsa \
        --from-file id_rsa.pub=/path-to/id_rsa.pub

#### Create a TLS secret specifying a certificate and the associated key
    oc create secret tls secret-tls \
        --cert /path-to-certificate --key /path-to-key

#### Exposing Secrets to Pods
    oc create secret generic demo-secret \
        --from-literal user=demo-user \
        --from-literal root_password=zT1KTgk
```yaml
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: demo-secret 
          key: root_password
```
>**Important**
> You can also use the oc set env command to set application environment variables from either secrets or configuration maps. In some cases, the names of the keys can be modified to match the names of environment variables by using the --prefix option. In the following example, the user key sets the MYSQL_USER environment variable, and the root_password key sets the MYSQL_ROOT_PASSWORD environment variable. If the key name is lowercase, then the corresponding environment variable is converted to uppercase

    oc set env deployment/demo --from secret/demo-secret --prefix MYSQL_

#### Secrets as Files in a Pod
    oc set volume deployment/demo \
        --add --type secret \
        --secret-name demo-secret \
        --mount-path /app-secrets

>**Important**
> If the mount point already exists in the pod, then any existing files at the mount point are obscured by the mounted secret. The existing files are not visible and are not accessible.

#### Create ConfigMap
    oc create configmap my-config \
        --from-literal key1=config1 --from-literal key2=config2

>**Note**
> Secrets and configuration maps occasionally require updates. Use the oc extract command to ensure you have the latest data. Save the data to a specific directory using the --to option. Each key in the secret or configuration map creates a file with the same name as the key. The content of each file is the value of the associated key. If you run the oc extract command more than once, then use the --confirm option to overwrite the existing files

    oc extract secret/htpasswd-ppklq -n openshift-config --to /tmp/ --confirm
    oc set data secret/htpasswd-ppklq -n openshift-config --from-file /tmp/htpasswd

### Security Context Constraints (SCC)

You can run the following command as a cluster administrator to list the SCCs defined by OpenShift.

    oc get scc

>**Note**
> OpenShift provides eight default SCCs
* **anyuid**
* **hostaccess**
* **hostmount-anyuid**
* **hostnetwork**
* **node-exporter**
* **nonroot**
* **privileged**
* **restricted**

#### Get additional information about an SCC
    oc describe scc anyuid

#### Use the scc-subject-review subcommand to list all the security context constraints that can overcome the limitations of a container
    oc get pod podname -o yaml | oc adm policy scc-subject-review -f -

#### Create a Serviceaccount
    oc create serviceaccount service-account-name

>**Note**
> To associate the service account with an SCC, use the oc adm policy command. Use the - z option to identify a service account, and use the -n option if the service account exists in a namespace different than the current one

    oc adm policy add-scc-to-user SCC -z service-account

>**Note**
> Assigning an SCC to a service account or removing an SCC from a service account must be performed by a cluster administrator. Allowing pods to run with a less restrictive SCC can make your cluster less secure. Use with caution.

    oc set serviceaccount deployment/deployment-name service-account-name