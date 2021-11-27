---
title: Practical Tekton
author: Mark
date: '2020-09-12T10:11:00.001Z'
categories:
  - Containers
  - Development
tags:
  - Tekton
  - Pipelines
  - Development
thumbnail: images/thumbnails/tekton.png
featured: true
toc: true
---

## Introduction
This post gives an overview of Tekton based on my initial testing and exploration of the project within a RedHat OpenShift (Kubernetes) cluster (using v4.4 and v4.5)

My goal was to explore the new technologies to see if I could simplify the developer experience for our developers and users. A simple task: to take a VS Code commit (running on Windows 10 desktop I used in my day job), trigger a pipeline to build the application into a container image, run unit tests and vulnerability scanning, and then deploy the application image to the OpenShift cluster using KNative. 

Note that Tekton is known as OpenShift Pipelines in OpenShift, and KNative is known as OpenShift Serverless. They're essentially the same thing, and I may use those terms interchangably in this document. 

I'm not going to cover installation of Tekton or KNative here, because there are plenty of documents out there that already cover it, both in vanilla Kubernetes clusters and OpenShift clusters. In any case with OpenShift versions of these products, it's relatively easy to install using the Operators feature found in OpenShift.

## What is Tekton?

Tekton is a tool for creating CI/CD application container build pipelines within Kubernetes. Tekton is *kubernetes-native*, meaning it was designed from the ground up as a CI/CD system running directly inside a Kubernetes cluster. It is implemented using various CRDs (`Pipeline, PipelineRun, Tasks, ClusterTasks, Triggers, etc`) provided under `tekton.dev` and `triggers.tekton.dev` API groups. As such, if you're experienced Kubernetes user and comfortable with applying, patching and editing the YAML of Kubernetes resources, you'll be right at home.

## Pipelines, Tasks and Steps

To deploy a Tekton CI/CD pipeline, you start of by creating a `Pipeline` YAML resources which consists of a number of `Tasks`. Each `Task` in the `Pipeline` gets deployed as its own pod within the namespace, and is made up of a series of steps, which each step being a separate container image in the pod. 

So, for example if you have a `Task` with three build, push, scan steps, a single pod with three different container images is normally deployed. In addition, Tekton itself adds init containers and other things to the pod in order to execute the pipeline (for example, the pipelines-creds-init-rhel8 init container which setups secrets and any script resources specified in your tasks, etc). The order of execution of each `Task` is determined by setting the `spec.tasks[*].runAfter:` parameter, which can be used to ensure the Pipeline runs in a specific order (steps can also run in parallel by setting these to be the same on different `Tasks`). Below is a fabricated example to illustrate this:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: my-pipeline
spec:
  tasks:
  - name: git-clone
    taskRef:
      kind: Task
      name: git-clone
  - name: build
    taskRef:
      kind: Task
      name: build
    runAfter:
      - git-clone
  - name: scan
    taskRef:
      kind: Task
      name: scan
    runAfter:
      - build
  - name: push
    taskRef:
      kind: Task
      name: push
    runAfter:
      - scan
```

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build
spec:
  steps:
  - name: build-binary
    image: registry/builder:latest
  - name: test
    image: registry/test:latest
```

Tekton has a [catalog](https://github.com/tektoncd/catalog) of pre-created `Tasks`, and RedHat have a curated list of these that are then bundled as ClusterTasks and installed when OpenShift Pipelines is. `ClusterTasks` are identical to `Tasks` except that they are global resources and available for use by all users across all namespaces in a cluster, whereas `Tasks` are namespaced.


## Parameters

Inputs to your pipeline are specified in `spec.params` of the `Pipeline` object, and these can then be made available to and referenced by each task (`spec.tasks[*].params`):

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: my-pipeline
spec:
  params:
  - name: GIT_REPO
    type: string
    description: URL of the Git repository
  - name: GIT_BRANCH
    type: string
    description: Git Branch
  - name: REGISTRY
    type: string
    description: Container registry name
  - name: DEST_IMAGE
    type: string
    description: Name of image that will be built
  - name: DEST_IMAGE_TAG
    type: string
    description: Image tag
  tasks:
  - name: git-clone
    taskRef:
      kind: Task
      name: git-clone
    params:
    - name: source_dir
      value: "$(params.GIT_REPO)"
  - name: build
    taskRef:
      kind: Task
      name: build
    params:
    - name: image-name
      value: "$(params.REGISTRY)/${DEST_IMAGE}/${DEST_IMAGE_TAG}
```

You can also use parameters that were the outputs from other `Tasks` that have already executed. For example, you could use the Git commit SHA from the HEAD of the repository you clone in a git-clone `Task`, and pass it as an input into the build `Task` that builds the container (for example, perhaps you want to add a container image LABEL with the commit sha). These are held in a parameter called `$(tasks.<task_name>.results)` (tou can also create these yourself, more on that in a different post)

``` yaml
 tasks:
  - name: git-clone
    taskRef:
      kind: Task
      name: git-clone
  - name: build
    taskRef:
      kind: Task
      name: build
    params:
    - name: commit
      value: "$(tasks.git-clone.results.commit)"
```

## PipelineResources
There is also a Tekton resource type called a `PipelineResource`, which is an input or output object that can be defined as a `spec.resource` and provided to a `Task`. For example, you might have a Git Repo `PipelineResource` input which defines a Git repo (url, branch) a Git clone type `Task` can use as an input, or an output object for a container image output from a build `Tasks` (eg `$(resources.outputs.image.url)`). 

In my experience with Tekton 0.7, these were hideously complicated to debug, I found them very opaque and hard to troubleshoot, and had compatability issues using them where they didn't fully support the auth mechanism of our container registry (JFrog Artifactory). In most cases using parameters was easier and just as functional, so I quickly stopped using PipelineResources. 

In additon, during OpenShift Pipelines version updates I noticed that more ClusterTasks were switching from using PipelineResouurces as inputs to using standard parameters as inputs (eg `$(params.git_url`) instead of `$(resources.inputs.git.url)`). There is a section of the Tetkon docs [Why Aren't PipelineResources in Beta?](https://github.com/tektoncd/pipeline/blob/master/docs/resources.md#why-arent-pipelineresources-in-beta) which also suggests there have been issues, and so it appears that they are falling out of favour or at least need some more work. In any case, I've not found a compelling reason for using them over parameters.

## Secrets

When using `Secrets` with Tekton it is worth noting that in comparison to others users of `Secrets`. Tekton is very particular about their type and annotations. Most of our "Git secrets" (`Secrets` containing private SSH keys that are attached to `BuildConfigs`) were initially created as `type: Opaque`. Whilst this allows them to be used with `BuildConfigs`, they are not picked up and used with Tekton. To use them, you will need to ensure the `Secrets` type is set to `type: kubernetes.io/ssh-auth`.

In addition, Tekton secrets must have annotations that explictly define hosts where they will be used. Lastly, a `data.known_hosts` (which you can get from $HOME/.ssh/know_hosts after you've connected to it via SSH) is also required to avoid error messages. The example below shows an example of a correctly annotated SSH key `Secret` for use with Tekton.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my_git_ssh_key
  annotations:
    tekton.dev/git-0: mygithost.example.com 
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: <base64 encoded>
  known_hosts: <base64 encoded>
```

## Workspaces
Workspaces were introduced into Tekton as a new feature whilst I was working on it. They're kind of like standard `Volumes` in a Pod in that they present `Secrets`, `ConfigMaps` or `PersistentVolumeClaims` into a location in the filesystem of a given `Task's` pod. I only used a `PVC` in my testing. The value of this is that you can share data between tasks (eg the source code your application build task needs after it has cloned from your git clone task).

## Running a Pipeline
When you execute a `Pipeline` a `PipelineRun` object is created that connects the `Workspace`, `ServiceAccounts`, `Pipelines` etc together and deploys the Pods needed to run your `Pipeline` - or in other words, there is one `PipelineRun` object created for each time the `Pipeline` is executed.

As all Tekton objects are implemented natively in Kubernetes as CRDs, you could just use oc/kubectl to use Tekton, but using tkn CLI client is much easier (in OpenShift, you can also use the Web Console UI, navigate Pipelines menu then simply click Start in the context menu) for interacting and using Tekton pipelines. For example, to start your pipeline run the following command:

```
tkn start mypipeline
```

By default, tkn will go into an interactive mode and prompt you to enter all the input information it requires. But I quickly got into the habit of passing these all as additional command line parameters, and immediately displaying any logs.

```
tkn pipeline start mypipeline --user-param-defaults -w name=workspace,claimName=workspace --showlog
```

## Conclusion

Though I have been using OpenShift and Kubernetes for quite a few years, I was new to both KNative and Tekton. Both are fast moving projects, with many resources in alpha or beta (RedHat Tech Preview) at best.

As with any new software, it can be a little tricky to workout and understand how it all integrates together. This is particulary true in a company like the one where I work where we run OpenShift/Kubernetes clusters in a more secure, disconnected, on-premise environment. 

In particular, we tend to run with on-prem instances of commercial products (eg JFrog Artifactory Container Registry) instead of cloud or externally hosted service (like Docker Hub or Quay.io). Even though these services run on a private disconnected network, we are also required to adhere to stricter security policies than most organisations (eg never public open access Git repos or registries, always requiring credentials for any type of pull or push, using private CAs and PKI). This means being an early adopter of new software- that developers and testers have developed with standard cloud services out on the internet rather than on-prem commerical products - you're likely to encounter more bugs for the first time using new projects.

The table below shows the versions I've been using, and each version fixed bugs I'd actually seen myself in the prior version. I can't stress enough that if you're an OpenShift or non-cloud Kubernetes user where you manage the versions, use both the latest OpenShift and Pipeline versions. 

| OpenShift | Pipelines | Base Tekton | tkn CLI |
|-----------|-----------|-------------|---------|
| OpenShift 4.6 | Tech Preview 1.2 | Tekton 0.16.3 | tkn 0.13.1 |
| OpenShift 4.5 | Tech Preview 1.1 | Tekton 0.14.3 | tkn 0.11.0 |
| OpenShift 4.4 | Tech Preview 1.0 | Tekton  0.11.3 | tkn 0.9.0 |

The other thing with new alpha/beta software is that it changes quite quickly, and this means that many of the articles, tutorials, GitHub comments and issues reference objects that have been deprecated or now have a new `spec:` or something (for example one of the prominent Serverless tutorials still in the top of Google searchs as I write this even details the KNative build component, which has now been deprecated in favour of Tekton in a separate project). Be very careful using older examples and tutorials found on the internet.

Anyway, after a few teething issues I got a working Python pipeline where I built, scanned and deployed a Python container image as a Knative service and I can say I really like Tekton. In particular, I never really enjoyed writing Groovy and much prefer creating pipelines from combination of bash scripts and YAML editing of Kubernetes resources.

In my next post, I will detail how I'm using KNative (Serverless) as a lightweight way to deploy your application at the end of the Pipeline. Knative promises lots, but its use in development and build systems is a solid use case that can easily be used today.
