# OC Commands
## Login & Auth & User
* Openshift Cluster Login
    ```bash
      oc login -u ${USER} -p {PASSWORD} ${CLUSTER}
    ```
* Htpasswd Oauth Create  
    - reference 
        * Guide Document > Auth & Auth > Config Identity > Htpasswd
        > https://docs.openshift.com/container-platform/4.5/authentication/identity_providers/configuring-htpasswd-identity-provider.html
    1. Create Htpasswd
        ```bash
          # -c Option is for creating a new file
          htpasswd -c -B -b [file name] [user] [password]
          # new user add in exist htpasswd file 
          htpasswd -B -b [file name] [user] [password]
        ```
    2. Create Secret from htpasswd
        ```bash
          oc create secret generic [secretname] --from-file htpasswd=[file-name] -n openshift-config
        ```
    3. Create Oauth 
        ```bash
          # Change Oauth 
          oc get oauth [OauthName] -o yaml > [dest.yaml]
        ```
        ```yaml
          spec:
            identityProviders:
            -name: name
              mappingMethod: claim
              type: HTPasswd
              htapsswd:
                fileData:
                  name: file-name
        ```
        ```bash
          oc replace -f [yamlfilename.yaml]
        ```
* Groups
    - Create Group
        ```bash
          oc adm groups new [group-name]
        ```
    - User add to group
        ```bash
          oc adm groups add-users [groupname] [username]
        ```
* Role control
    - Reference
        * DOC > Auth & Auth > Using RBAC
        > https://docs.openshift.com/container-platform/4.5/authentication/using-rbac.html
    - Role Binding
        ```bash
          # Cluster Role Add
          oc adm policy add-cluster-role-to-group [Cluster role name] [group name]
          # Namespace Role Add
          oc adm policy add-role-to-group [Namespace Role name] [group name]
          # Role get 
          oc get role
          oc get rolebinding
          # Delete Role from Authorized Users
          # DOC > Application > Porjects > Config Project
          #  https://docs.openshift.com/container-platform/4.5/applications/projects/configuring-project-creation.html
          oc adm policy remove-cluster-role-from-group [Rolename] system:authenticated:oauth
        ```
## APPLICATION Control
* Application Control
    - New Application Create
        ```bash
          # New Application From Github Folder Deployment-Config
          oc new-app --as-deployment-config --name [Deploy Name] https://github.com/Repo --context-dir [dirname] -e [ENV Key]=[ENV Value]

          # New App From Docker Image
          oc new-app --name [Deployname] --docker-image docker.io/[name]/[repo]:[tag] -e [Env Key]=[Env Value]

          # SECRCT Create
          oc create secret generic [SEC NAME] --from-literal [KEY]=[VALUE]

          # SET Env to Deploy from Secret
          oc set env [Resource]/[ResourceName] --prefix [PREFIX_] --from secrct/[SC NAME]

          # SET Volume Claim
          oc set volumes deployment/[DEPLOY name] --add --name [volname] --type pvc --claim-size [SIZE] --claim-mode rwo --mount-path [PATH]
        ```
* Service Account
    - Create SA & Give SCC
        ```bash
          # Create SA
          oc create serviceaccount [SA NAME]
          # Get SCC
          oc get scc
          # Give SCC
          oc adm policy add-scc-to-user anyuid -z [SANAME]
          # Set SA to Deploy
          oc set serviceaccount deployment/[deployname] [SA Name]
        ```
* Service & Route
    - Expose SVC
        ```bash
          oc expose service [SVC Name] --hostname test.test.test.com
        ```
* Template Control
    - Create Project Template
        * Reference
            - DOC > Application > Porjects > Config Project
            >  https://docs.openshift.com/container-platform/4.5/applications/projects/configuring-project-creation.html
            - Network Policy 
            - DOC > Networking > Network Policy > Defining a Default Network Policy
            > https://docs.openshift.com/container-platform/4.5/networking/network_policy/default-network-policy.html
            - Limit Range
            - DOCK > Nodes > Working With Clusters > Setting Limit
            > https://docs.openshift.com/container-platform/4.5/nodes/clusters/nodes-cluster-limit-ranges.html
            - Resource Quota
            - DOC > Applications > Quotas > Resource Quota 
            > https://docs.openshift.com/container-platform/4.5/applications/quotas/quotas-setting-per-project.html
            
        ```bash
          # Get Template
          oc adm create-bootstrap-projcet-template -o yaml > [File.yaml]

          # Edit Template
          # If you want to label with projectname
          # Add metadata.labels.name: ${PROJECT_NAME}
          # ADD Project NetworkPolicy & Limitrange & ResourceQuotas

          # Create Project Template
          oc create -f [template.yaml]

          # Edit Resource
          oc edit projects.config.openshift.io/cluster
          # edit Yaml
          # spec.projectRequestTemplate.name: [TemplateName]
        ```

* PassThrough
    - Create Passthrough
        ```bash
          # Create TLS Secret
          oc create secret tls [SEC Name] --cert [CRT NAME] --key [KEY name]
          
          # Mount TLS TO Deploy
          oc set volume deployment/[DEPOY_NAME] --add --tpye secret --secret-name [SEC Name] --mount-path [TLS Path]

          # Double Check TLS SAN Info
          openssl -in [CRT file.cert] -noout -ext subjectAltName

          # Create Passthrough
          oc create route passthrough --service [SVC Name] --hostname Domain.name.com
        ```
* AutoScale
    - Create Autoscale
        ```bash
          # Create Autoscale
          oc autoscale deployment/[DEPLOY_NAME] --min [amount] --max [amount] --cpu-percent [PERCENT]
        ```