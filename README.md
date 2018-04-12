# bluecatalog
A generated Bluemix application

[![](https://img.shields.io/badge/bluemix-powered-blue.svg)](https://bluemix.net)

## Table of Contents
* [Table of Contents](#table-of-contents)
* [Introduction](#introduction)
* [Requirements](#requirements)
* [Getting Started](#getting-started)
* [Install Bluecompute Reference Architecture Application](#install-bluecompute-reference-architecture-application)
* [Import this project into Microclimate](#import-this-project-into-microclimate)
* [Exploring Microclimate Workspace](#exploring-microclimate-workspace)
    + [Edit code Section](#edit-code-section)
    + [Open app Section](#open-app-section)
    + [App logs Section](#app-logs-section)
    + [App monitor Section](#app-monitor-section)
      - [Optional: Trigger a Stress Load](#optional-trigger-a-stress-load)
    + [Pipeline Section](#pipeline-section)
* [Create and Run an Automated Jenkins Pipelines](#create-and-run-an-automated-jenkins-pipelines)
    + [Create Jenkins Pipeline](#create-jenkins-pipeline)
    + [View Jenkins Pipeline](#view-jenkins-pipeline)
    + [Create GitHub Web Hook](#create-github-web-hook)
    + [Trigger the pipeline through a Git Commit Push](#trigger-the-pipeline-through-a-git-commit-push)
    + [Open Jenkins and see the pipeline in action](#open-jenkins-and-see-the-pipeline-in-action)
    + [Open the new Jenkins deployment](#open-the-new-jenkins-deployment)
* [Conclusion](#conclusion)

## Introduction
*This project is part of the 'IBM Cloud Native Reference Architecture' suite, available at
https://github.com/ibm-cloud-architecture/refarch-cloudnative-kubernetes*

The purpose of this project is to demonstrate how to create, deploy, and monitor an application via [`Microclimate`](https://microclimate-dev2ops.github.io/about) on an [`IBM Cloud Private`](https://www.ibm.com/cloud/private)(ICP) Environment. Also, we are going to demonstrate how this application can interact with other applications deployed in the cluster through a helm chart.

## Requirements
- [Fork](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-microclimate#fork-destination-box) this repository
    + This is needed in order to setup automated Jenkins pipelines.
- [GitHub Personal Access Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
    + This token is needed to import the project into microclimate.
    + It will also be used to create automated Jenkins pipelines.
- [IBM Cloud Private](https://www.ibm.com/cloud-computing/products/ibm-cloud-private/) Cluster
    + Create a Kubernetes cluster in an on-premise datacenter. The community edition (IBM Cloud private-ce) is free of charge. 
    + Follow the instructions [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.2/installing/install_containers_CE.html) to install IBM Cloud private-ce.
- [Helm](https://github.com/kubernetes/helm) (Kubernetes package manager)
    + Follow the instructions [here](https://github.com/kubernetes/helm/blob/master/docs/install.md) to install it on your platform.
    + If using `IBM Cloud Private` version `2.1.0.2` or newer, we recommend you follow these [instructions](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.2/app_center/create_helm_cli.html) to install `helm`.
- [Microclimate](https://microclimate-dev2ops.github.io/about)
    + Once your ICP cluster is ready open this [link](https://microclimate-dev2ops.github.io/installicp) and follow the instructions in the `Create a docker-registry secret` and `Patch the docker-registry secret into your ICP service account` sections to setup your ICP cluster for microclimate.
    + Once you are done with the above, use the following command to install the microclimate chart:
    ```bash
    # At the time of this writing, 18.03 is the latest version
    $ cd microclimate-18.03

    # Install microclimate chart
    $ helm install --name microclimate --set persistence.enabled=false \
        --set jenkins.Persistence.Enabled=false \
        --set jenkins.Master.ServiceType=NodePort \
        --set jenkins.Master.NodePort=30127 \
        stable/ibm-microclimate --tls
    ```

    Once deployed, the above command will give you instructions on how to access microclimate web ui. Please not that it will take 5-10 minutes for microclimate to be fully initialized.

    Also, the above command will install microclimate without persistence, which is fine for quick testing. However, we recommend to use persistence in order to retain microclimate and Jenkins data. To configure persistance on microclimate, please checkout the `Provide persistent storage volumes` section on microclimate [installation](https://microclimate-dev2ops.github.io/installicp) page.

    NOTE: If you are using a version of ICP older than 2.1.0.2, you don't need to add the `--tls` at the end of the helm command.

## Getting Started
For this demo, we will be doing the following things:
- Install bluecompute reference architecture application:
- Create a microclimate application by importing this repository
- Setup automated Jenkins CICD pipelines
- Trigger Jenkins pipelines

## Install Bluecompute Reference Architecture Application
The microclimate application requires that the Bluecompute Microservices Reference Architecture Chart is installed as this application pulls data from it's catalog microservice to showcase how it can communicate with existing Kubernetes applications.

To deploy the Bluecompute Chart, use [these](https://github.com/ibm-cloud-architecture/refarch-cloudnative-kubernetes#deploy-bluecompute-to-ibm-cloud-private) instructions. Please note that it takes Bluecompute about 5-10 minutes to be fully ready after deployment.

To learn more about Bluecompute Microservices Reference Architecture, please checkout its repo [here](https://github.com/ibm-cloud-architecture/refarch-cloudnative-kubernetes#introduction).

## Import this project into Microclimate
Assuming you already [forked](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-microclimate#fork-destination-box) this repository this project and installed microclimate, let's go ahead and open microclimate in your browser. If you lost the microclimate access instructions from the helm install command, you can get them back using this command:
```bash
$ helm status microclimate --tls
```

Once you open microclimate in the browser, click `Accept` to accept the Microclimate license agreement.

Now you click the `Import project` button.

![Application Architecture](static/01_import.png?raw=true)

This will open the following view, where you have enter the repository details and finalize the import.

![Application Architecture](static/02_import_details.png?raw=true)

After a few seconds, you will get the following view confirming the import. Now click the `Edit code` button, which takes you to the microclimate workspace.

![Application Architecture](static/03_edit_code.png?raw=true)

You have successfully imported this project into Microclimate.

## Exploring Microclimate Workspace
You will now be presented to the Microclimate workspace view for the `bluecatalog` project. In here you have access to 5 views, where `Edit code` is presented by default. Before explaining each view separately, let's first go over what happened after you imported the project into Microclimate:
- A new docker image was built & pushed to ICP's internal Docker registry.
    + You can see the `bluecatalog` image if you open your browser and go to `https://ICP_IP:8443/console/images`.
    + Alternatively, you can open your terminal and enter ```$ kubectl get images``` to see the `bluecatalog` image.
- A helm chart was deployed using the freshly pushed docker image.
    + You can see the `bluecatalog-deployment` by opening a browser and going to `https://ICP_IP:8443/console/workloads/deployments`, then search for `bluecatalog-deployment`.
    + Alternatively, you can open your terminal and enter the ```kubectl get deployments bluecatalog-deployment``` command to see the deployment.
    + **NOTE**: Since microclimate uses its own helm client, you won't be able to see the helm release in `https://ICP_IP:8443/catalog/instances` 

Now that you know what happened in the background, let's explore microclimate project views a bit more.

### Edit code Section
This section presents you with the online code editor, which has a built-in git commands and a CLI where you can push/pull commits from remote repositories. By editing and saving changes from this view, you are triggering a new docker image build & push and an update to the existing helm chart deployment.

![Application Architecture](static/04_code_editor.png?raw=true)

### Open app Section
This section (shown below) presents you with a view that lets you see what your app looks like, which is very useful for debugging. There is an `Application URL` that let's you enter different paths and queries. There is also an `Open in new tab` button that lets you open you app in a separate tab away from microclimate.

![Application Architecture](static/05_open_app.png?raw=true)

Since we are here, let's test adding the `/catalog` path to the `Application URL` and pressing `Reload` to see if it displays the data that it gathers from the Bluecompute Application, which demonstrates a successful connection.

![Application Architecture](static/06_catalog.png?raw=true)

If you see an output similar to the above, then that means that the bluecatalog microclimate app was successfully deployed and connected to the Bluecompute Application!

### App logs Section
This section presents you with a view of the logs produced by the application, which is great for debugging.

![Application Architecture](static/07_app_logs.png?raw=true)

Notice the logs we see above is generated by the `/catalog` request.

### App monitor Section
This section presents you with 3 tabs. 
- The `Dashboard` tab is a collection of dashboards for things like CPU, HTTP Incoming Requests, Memory, etc.
- The `Profiling` tab displays the data as Flame Graphs that you can click on to see the call stacks in the graph nodes.
- The `Summary` tab gives you a more concise summary of all the data presented in the other tabs, such as HTTP Requests (endpoints and correspinding # of hits and response times), environment, and resource usage.

Notice in the screenshot below the values generated by our requests in the CPU, HTTP Incoming Requests, and Memory graphs.

![Application Architecture](static/08_app_monitor.png?raw=true)

For more information on the Application Metrics for Node.js, check out this [link](https://developer.ibm.com/node/monitoring-post-mortem/application-metrics-node-js/).

#### Optional: Trigger a Stress Load
To trigger a stress load on the app, click the `Run load` button above and see how the dashboard metrics change with the new load. 

![Application Architecture](static/09_load.png?raw=true)

Notice how the graph values change once the workloads are triggered. The values go up as the load increases, then go down as either the app crashes or the load decreases.

### Pipeline Section
Last but not least, there is the Pipeline section, in which you can setup Jenkins pipelines that can be configured to automatically be triggered by git commit.

![Application Architecture](static/10_create_pipeline.png?raw=true)

Click the `Create pipeline` button and let's examine how to create an automated Jenkins pipelines in the next section.

## Create and Run an Automated Jenkins Pipelines
The Microclimate workspace is great for developing tweaks in your application that you can quickly see in the same workspace. However, if you want a separate environment that other colleagues (i.e. QA) can see and test after pushing your updates upstream, it helps to have an automated CICD pipeline that does all of that for you.

Luckily, Microclimate comes with a Jenkins instance that it can setup for you with minor configuration.

### Create Jenkins Pipeline
On the previous section you were asked to click the `Create pipeline` button, which should present a view as follows:

![Application Architecture](static/11_pipeline_details.png?raw=true)

Follow the instructions above to to setup your pipeline, then click the `Create pipeline` button. If successful, you will get a confirmation view as follows:

![Application Architecture](static/12_pipeline_created.png?raw=true)

### View Jenkins Pipeline
To view your Jenkins pipeline on Jenkins, click the `Open pipeline` button shown in the above image, which will open Jenkins in a new tab. If for some reason, your browser gives you an error or an incomplete URL, try using `http://ICP_IP:30127/job/default/job/bluecatalog/`.

If you are asked to login, use the credentials below:
- **Username**: admin
- **Password**: admin

**Note:** We recommend that you change the above password to something more secure.

After successful login, you should see a browser window similar to the following:

![Application Architecture](static/13_jenkins.png?raw=true)

### Create GitHub Web Hook
In order for the Jenkins pipeline to trigger automatically when you push a git commit, you need to setup a GitHub webhook that notifies Jenkins to start the pipeline.

To create the web hook in GitHub, do the following:
1. From your repository in GitHub, click `Settings`, `Webhooks`, and then click `Add webhook`.
2. Enter the Payload URL: `JENKINS_URL/git/notifyCommit?url=GITHUB_URL`, replacing `JENKINS_URL` with the base URL of your Jenkins deployment (In this case, it is `http://ICP_IP:30127`) and `GITHUB_URL` with the URL of the GitHub repository. For example, `http://ICP_IP:30127/git/notifyCommit?url=https://github.com/microclimate-demo/node.git`.
3. Select the `Let me select individual events` option.
4. From the list of events that are displayed, select `Pull requests` and `Pushes`.
5. Click `Add webhook`.

Now your Jenkins pipeline should automatically run every time you do a `git push` with new commits.

### Trigger the pipeline through a Git Commit Push
Now it's time to trigger the pipeline with a new commit and test whether the GitHub web hook in your fork works. To do that, let's do the following:
- Go to the built-in editor and open the [public/index.html](public/index.html) file.
- Edit the text in line 10 from `Hello world! This is a StarterKit!` to `Hello world! This is a new change!` or whatever you like.
- Save the file.
- Open a terminal window.
- Change to the project directory.
```bash
$ cd bluecatalog
```
    
**NOTE:** It is called `bluecatalog` instead of `refarch-cloudnative-bluecompute-microclimate` because microclimate names the project folder after the `name` field in [package.json](package.json). If you cloned the repo in your machine, the project folder would be `refarch-cloudnative-bluecompute-microclimate`.

![Application Architecture](static/14_git.png?raw=true)

- Remove the current remote repository and add a new one.
```bash
$ git remote rm origin
$ got remote add origin https://github.com/YOUR_GITHUB_USERNAME/refarch-cloudnative-bluecompute-microclimate.git
```
- Commit & Push the change to the remote repository.
```bash
$ git add public/index.html
$ git commit -m "Changed the home page"
$ git push origin master
```

**Note:** You can perform the same steps above on your local machine by cloning your fork repo. When you do this, there is no need to add an `origin` remote url.

### Open Jenkins and see the pipeline in action
Now that you pushed a commit to the remote repository and assuming that the webhooks were setup properly, the Jenkins pipeline should be triggered and you can see it running. To view the running pipeline and its console output, open the URL `http://ICP_IP:30127/job/default/job/bluecatalog/job/master` in your browser and do the following.

![Application Architecture](static/15_jenkins_pipeline.png?raw=true)

Once in the console output, you should see output for the following:
- Git checkout of remote URL.
- Docker image build and image push to ICP Docker registry.
- Setup of Microclimate Helm client.
- Deployment/Upgrade of Helm chart.
- You should see a `Finished: SUCCESS` text at the end of the output if everything was successful.

![Application Architecture](static/16_jenkins_output.png?raw=true)

**Note:** The Jenkins pipelines use the `microclimate-pipeline-deployments` Kubernetes namespace to deploy and update the charts. This is separate from the microclimate deployments that occur by saving a file in the built-in code editor, which in our case would deploy the chart in the `default` namespace. Also, since the Jenkins pipelines use their own Helm client, the resulting Helm releases are not visible inside the ICP GUI, though the actual Kubernetes resources are. 

For example, if you open your browser and go to `https://ICP_IP:8443/console/workloads/deployments`, you would be able to see both the microclimate deployment and the Jenkins pipeline deployment in the `default` and `microclimate-pipeline-deployments` namespaces, respectively. Here is what that would look like:

![Application Architecture](static/17_icp_deployments.png?raw=true)

### Open the new Jenkins deployment
To open the new Jenkins pipeline deployment in your browser. Open the browser to `https://ICP_IP:8443/console/access/services` and perform the following steps:

![Application Architecture](static/18_icp_services.png?raw=true)

To open the application URL in a new browser tab, perform the following steps:

![Application Architecture](static/19_icp_service.png?raw=true)

Now a new browser tab will open greeting you with the new message:

![Application Architecture](static/20_updated_app.png?raw=true)

You have successfully created and ran a Jenkins pipeline that created a new Helm release with your new changes!

## Conclusion
Congratulations on making it to the end! Let's recap on what you just did.
* You imported your fork of this project into Microclimate, which by itself does the following:
    + Build and push the project's Docker image into ICP's Docker Registry.
    + Updated the project's helm chart with the new Docker image and deployed it into `default` namespace.
    + Starts collecting the app logs.
    + Starts collecting Application Metrics.
* You explored Microclimate workspace built-in features, which include:
    + Code editor and CLI.
    + Test browser to view your application UI.
    + App logs.
    + Application Metrics.
    + Jenkins Pipelines.
* You tested the application endpoints and its connection with the Bluecompute Reference Architecture app.
* You ran a JMeter load from the Application Metrics GUI and saw the metrics value change.
* You created a Jenkins pipeline and setup GitHub triggers to automatically trigger the pipeline when you push new commits to the repository.
* You used the built-in code editor to make changes and push them to the remote repository, which triggered the pipeline.
* You opened Jenkins to see the pipeline run and saw its console output.
* You used ICP GUI to see the resulting Kubernetes resources.
* Lastly, you used ICP GUI to open the newly deployed application in a new browser tab.

You now have the knowledge to use Microclimate to set up a fully automated DevOps experience on IBM Cloud Private. Now you just need to rinse and repeat for any other applications.