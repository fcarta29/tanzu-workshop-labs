# LABS-02 - TAS4k8s Setup

[Main](../../README.md)

# Introduction

Kubernetes has become the standard for managing and running containerized applications. By running TAS on Kubernetes, you can reiterate TAS developer experience on K8s "Here is my code, run it on Kubernetes, I don't care how.”

TAS (Tanzu Application Service) on K8s is a Kubernetes native artifact to deploy TAS on a Kubernetes cluster. This project shows how to run TAS on K8s. 


- [Prerequisites](#prerequisites)
  * [Required Tools](#required-tools)
  * [Infrastructure Requirements](#Infrastructure-requirements)
- [Steps to deploy](#steps-to-deploy)
- [Validate the deployment](#validate-the-deployment)
- [Delete the cf-for-k8s deployment](#delete-the-cf-for-k8s-deployment)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Prerequisites

### Required Tools

You need the following CLIs on your system to be able to run the script:

- `ytt` [install link](https://carvel.dev/#install) [github repo](https://github.com/k14s/ytt)
  - cf-for-k8s uses `ytt` to create and maintain reusable YAML templates. You can visit the ytt [playground](https://get-ytt.io/) to learn more about its templating features.
- `kapp` [install link](https://carvel.dev/#install) [github repo](https://github.com/k14s/kapp)
  - cf-for-k8s uses `kapp` to manage its lifecycle. `kapp` will first show you a list of resources it plans to install on the cluster and then will attempt to install those resources. `kapp` will not exit until all resources are deployed and their status is running. See all options by running `kapp help`.
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [`cf cli`](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html) (v7+)

> Make sure that your Kubernetes config (e.g, `~/.kube/config`) is pointing to the cluster you intend to deploy cf-for-k8s to.

### Infrastructure Requirements

To deploy cf-for-k8s as is, the cluster should:

- be running Kubernetes version within range 1.16.x to 1.19.x
- have a minimum of 3 nodes
- have a minimum of 4 CPU, 15GB memory per node
- if PodSecurityPolicies are enforced on the cluster, [pods must be allowed to
  have `NET_ADMIN` and `NET_RAW` capabilities](https://istio.io/latest/docs/ops/deployment/requirements/#required-pod-capabilities)
- have a CNI plugin (Container Network Interface plugin) that supports network policies (otherwise, the NetworkPolicy resources applied by cf-for-k8s will have no effect)
- support `LoadBalancer` services
- most IaaSes come with `metrics-server`, but if yours does not come with one (for example, if you are using `kind`), you will need to include `add_metrics_server_components: true` in your values file.
- defines a default StorageClass
  - requires [additional config on vSphere](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/storageclass.html), for example
- OCI-compliant container registry


## Steps to deploy

1. Clone and initialize this git repository:

git clone https://github.com/cloudfoundry/cf-for-k8s.git -b v1.0.0

2. Create a "CF Installation Values" file and configure it<a name="cf-values"></a>:

    Use the included hack-script to generate the install values

   >  **NOTE:** The script requires the [BOSH CLI](https://bosh.io/docs/cli-v2-install/#install) in installed on your machine. The BOSH CLI is an handy tool to generate self signed certs and passwords. You can generate certificates for the above domains and paste them in `crt`, `key`, `ca` values
      - **IMPORTANT** Your certificates must include a subject alternative name entry for the internal `*.cf-system.svc.cluster.local` domain in addition to your chosen external domain. 

   ```console
   ./cf-for-k8s/hack/generate-values.sh -d <cf-domain> > cf-values.yml
   ```

   Replace `<cf-domain>` with _your_ registered DNS domain name(Ex:- workshop-XX.domain.com) for your CF installation.


3. Open the file and append the below values at the end. Provide your credentials to an external app registry:

   vi cf-values.yml

    ```console
    app_registry:
        hostname: "harbor.workshop.frankcarta.com"
        repository_prefix: "harbor.workshop.frankcarta.com/workshop-xx"
        username: "workshop-xx"
        password: "password"

    remove_resource_requirements: false
    add_metrics_server_components: true
    allow_prometheus_metrics_access: true
    use_external_dns_for_wildcard: true
    enable_automount_service_account_token: true
    metrics_server_prefer_internal_kubelet_address: false
    use_first_party_jwt_tokens: true

    load_balancer:
        enable: true
    ```

4. Run the following commands to install Cloud Foundry on your Kubernetes cluster:

      1. Render the final K8s template to raw K8s configuration

         ```console
         ytt -f cf-for-k8s/config -f cf-values.yml > cf-for-k8s-rendered.yml
         ```

      2. Install using `kapp` and pass the above K8s configuration file

         ```console
         kapp deploy -a cf -f cf-for-k8s-rendered.yml -y
         ```

   Once you run the command, it should take about 10 minutes or less, depending on your cluster bandwidth and size. `kapp` will provide updates on pending resource creations in the cluster and will wait until all resources are created and running. Here is a sample snippet from `kapp` output:

   ```console
   4:08:19PM: ---- waiting on 1 changes [0/1 done] ----
   4:08:19PM: ok: reconcile serviceaccount/cc-kpack-registry-service-account (v1) namespace: cf-workloads-staging
   4:08:19PM: ---- waiting complete [5/10 done] ----
   ...
   ```

5. Configure DNS on your IaaS provider to point the wildcard subdomain of your system domain and the wildcard subdomain of all apps domains to point to hostname of the Istio Ingress Gateway service. You can retrieve the load balancer of this service by running:

   ```console
   kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[*].hostname}'
   ```
   > If you used a single DNS record for both `system_domain` and `app_domains`, then have it resolve to the Ingress Gateway's load balancer by defining A record. Wait for DNS changes to take effect  

## Explore KAPP CLI

   1. List applications deployed using kapp 
            
      ```console
      kapp list
      ```

   2. Check the logs of deployment using kapp
      
      ```console
      kapp logs -a cf
      ```

   3. Inspect application using kapp cli

      ```console
      kapp inspect -a cf
      ```



## Validate the deployment

1. Target your CF CLI to point to the new CF instance:

   ```console
   cf api --skip-ssl-validation https://api.<cf-domain>
   ```

   Replace `<cf-domain>` with your desired domain address.

2. Login using the admin credentials for key `cf_admin_password` in `${TMP_DIR}/cf-values.yml`:

   ```console
   cf auth admin <cf-values.yml.cf-admin_password>
   # or using yq: cf auth admin "$(yq -r '.cf_admin_password' cf-values.yml)"
   ```

3. Create an org/space for your app:

   ```console
   cf create-org test-org
   cf create-space -o test-org test-space
   cf target -o test-org -s test-space
   ```

4. Deploy a source code based app:

   ```console
   cf push test-node-app -p cf-for-k8s/tests/smoke/assets/test-node-app
   ```

   You should see the following output from the above command:
   ```console
   Pushing app test-node-app to org test-org / space test-space as admin...
   Getting app info...
   Creating app with these attributes...

   ... omitted for brevity ...

   type: web
   instances: 1/1
   memory usage: 1024M
   routes: test-node-app.<cf-domain>
   state since cpu memory disk details
   #0 running 2020-03-18T02:24:51Z 0.0% 0 of 1G 0 of 1G
   ```

   <br />

5. Validate the app is reachable over **https**:

   ```console
   curl -k https://test-node-app.<cf-domain>
   ```

   You should see the following output:
   ```console
   Hello World
   ```

## Delete the cf-for-k8s deployment (Optional)

You can delete the cf-for-k8s deployment by running the following command:

   ```console
   kapp delete -a cf
   ```