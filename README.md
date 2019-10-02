<p align="center">
  <img src="https://github.com/ehemmerlin/gitlab-dok8s/raw/master/images/logo.jpg" width="882" alt="GitLab stages">
  <p align="center">
    <strong>For full documentation, visit <a href="https://medium.com/@erichemmerlin/gitlab-auto-devops-on-digitalocean-kubernetes-cluster-f8b744b1e64c">this Medium post</a>.</strong>
  </p>
  <p align="center">
    <a href="https://github.com/ehemmerlin/gitlab-dok8s/actions"><img src="https://github.com/ehemmerlin/gitlab-dok8s/workflows/Build/badge.svg" alt="Build Status"></a> 
  </p>
</p>

# GitLab Auto DevOps on DigitalOcean Kubernetes cluster

Provision a DigitalOcean-managed Kubernetes cluster and deploy a NodeJs application to production using GitLab Auto DevOps with Prometheus monitoring, in less time than it takes to walk to your nearest bakery.

## Before your start

You'll need to :
* Register for an account on [DigitalOcean](https://www.digitalocean.com) and [generate a personal access token](https://www.digitalocean.com/docs/api/create-personal-access-token)
* Register for an account on [GitLab](https://www.gitlab.com)
* Install **Docker** and **git** on your machine 

Be aware that the following commands will set up a **DigitalOcean-managed cluster** with 2 nodes (1vCPU, 2GB) and some extra resources like a **Load Balancer**, which will be billed by DigitalOcean. [Check the pricing](https://www.digitalocean.com/pricing/).

## Getting Started

### Kubernetes

Clone this repository (which is based on https://github.com/snormore/doks-examples):
```
git clone git@github.com:ehemmerlin/gitlab-dok8s
cd gitlab-dok8s
```

Run a golang Docker container by typing:
```
docker run -it -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.13 bash
```

Inside the docker container, launch the **setup** script : it will install jq, kubectl and doctl inside a docker container then ask for your access key to login to DigitalOcean.
```
script/setup
```

Enter your DigitalOcean access token.

Note: you can also set the token by replacing "your_DO_token" with your DigitalOcean access token in the following command, before lauching the **setup** script.
```
docker run -it --env=DIGITALOCEAN_ACCESS_TOKEN="your_DO_token" -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.13 bash
```

Inside the docker container, launch the **up** script : it will create a DigitalOcean-managed Kubernetes cluster with default name `gitlab-demo` based on [Adding an existing Kubernetes cluster to Gitlab](https://gitlab.com/help/user/project/clusters/index.md#add-existing-kubernetes-cluster) to retrieve the API URL, CA certificate and Service Token used to integrate this Kubernetes cluster with GitLab.
```
script/up
```

You can specify a cluster name with:
```
script/up my-cluster-name
```

Note: If later you need the API URL, CA certificate or Service Token, run this script again with the same cluster name.

### GitLab

#### Configure Group-level Kubernetes cluster

Create a new group within gitlab.com :
https://gitlab.com/groups/new

Give the group a name and click on **Create group**.

In the menu panel on the left hand side of the group page, select **Kubernetes** and click on **Add Kubernetes cluster**.

Select **Add existing cluster** and give the cluster a name, then fill in the fields you got from the previous step (see the Kubernetes part above) : **API URL**, **CA certificate** and **Service Token**.

Enable both **RBAC-enabled cluster** and **GitLab-managed cluster** (if asked leave the **Project namespace** prefix blank) and click on **Add Kubernetes cluster**.

#### Install Tiller, Ingress and Prometheus

Once you add the cluster to GitLab, a list of applications to install into the cluster will appear. The first of these is **Helm Tiller**. Go ahead and click on **Install** to add it to the cluster.

The second one to install is **Ingress**. It will provision a DigitalOcean Load Balancer. Once installed copy the value of the **Ingress Endpoint** that appears then install **Prometheus** which is the third application we need.

Once these three applications are installed, scroll to the top of the page and locate the **Base domain** field. Enter the value of the **Ingress Endpoint** you copied, followed by .nip.io and click on **Save changes**.

Every project you'll add to this group can now be deployed in your Kubernetes cluster if you enable Auto DevOps on this project. So let's give it a try!

### Set up a GitLab project

Create a new project within gitlab.com : https://gitlab.com/projects/new

On the New Project page, select the **Create from template** tab. This will provide you with a list of template projects to use. Select **NodeJS Express** (which includes CI/CD with Auto DevOps feature) by clicking on **Use template**. Give the project a name and make sure you select the **Project URL** of the group you created. Click **Create project** when you are finished.

Next, in the navigation panel, select **Settings > CI/CD**. Expand the **Auto DevOps** section, and check the **Default to Auto DevOps pipeline** box. Leave the **Deployment strategy** set to **Continuous deployment to production**.

A new pipeline is created and a job is starting. In the navigation panel, select **CI/CD > Pipelines** to see this pipeline.

After the pipeline has finished, in the navigation panel, select **Operations > Environments**. Next to **production** on the right hand side, click on the first icon (Open live environment) to open your newly deployed NodeJS Express application, then click on the icon next to the right (Monitoring) to monitor your Kubernetes cluster using Prometheus.

Congratulations! You have just connected GitLab’s Auto DevOps with a DigitalOcean-managed Kubernetes cluster monitored by Prometheus.

## Cleaning Up

### GitLab

In the navigation panel of the group you created, select **Operations > Kubernetes** and select the name of your cluster. Uninstall **Ingress** to remove the DigitalOcean Load Balancer which was created. It can take some time at this step, so grab a cup of tea and refresh the page if needed. Then, scroll to the bottom of the page and expand **Advanced settings**. Under **Remove Kubernetes cluster integration** click **Remove integration**. Confirm by clicking Ok.

Optionally delete the group you created (it will also delete the project): in the navigation panel of the group you created, select **Settings > General**, scroll to the bottom of the page and expand **Path, transfer, remove** then select **Remove group** and confirm its name before clicking **Confirm**.

### Kubernetes

Inside the Docker container you started at the beginning, launch the **down** script : it removes the cluster:
```
script/down
```

You can specify a cluster name by typing (be careful if you used your own DigitalOcean-managed Kubernetes cluster in the first step because the following command will remove it!):
```
script/down my-cluster-name
```


Note: if for some reason you exited the Docker container prematurely, go to the [Getting started - Kubernetes](#kubernetes) section above to run it again. You'll need to enter your DigitalOcean access token again.

All the provisioned ressources have now been deleted.

To log out of the golang Docker container type:
````
exit
````

Everything has now been cleaned up.

## Troubleshooting

To control this Kubernetes cluster using kubectl, run the following command in the Docker container (if you exited it, go to the [Getting started - Kubernetes](#kubernetes) section above to run it again):
````
export KUBECONFIG="tmp/<your-cluster-name>-kubeconfig.yaml"
````

Note: by default the cluster name is : gitlab-demo

Now you can run all the kubectl commands you want:
```
kubectl get pods --all-namespaces
kubectl config set-context --current --namespace=gitlab-managed-apps
kubectl logs pod tiller-deploy-xxxx-yyyy # depending on the name of the tiller deploy pod
```

## References

 - https://gitlab.com/help/user/project/clusters/index.md#add-existing-kubernetes-cluster

 ## Credits

 - https://github.com/snormore/doks-examples
 - https://gitlab.com/gitlab-org/project-templates/express
 - <a href="https://fr.freepik.com/photos-vecteurs-libre/technologie">Image created by vectorpocket - fr.freepik.com</a>