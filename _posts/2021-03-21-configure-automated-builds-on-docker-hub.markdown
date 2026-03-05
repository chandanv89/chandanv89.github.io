---
layout: post
title: "Configure Automated Builds on Docker Hub"
date: 2021-03-21
categories: [devops, docker, engineering]
tags: [docker, docker-hub, ci-cd, github, automated-builds, devops]
description: "How to integrate GitHub with Docker Hub to set up automated Docker image builds triggered by code pushes."
---

![Integrate Github with Docker Hub to setup Automated Builds](/assets/images/post-2-header-image.png)

In the [previous article](/2021/01/17/dockerize-your-spring-boot-application.html) successfully dockerized our demo Spring Boot application. Now, let's take a look at how we can leverage the Github and Docker Hub infrastructure to setup auto builds to trigger the creation of Docker images for our application whenever we push some changes to the code repository.

Docker Hub can be integrated with most of the code versioning and hosting providers. For this tutorial, however, we'll consider GitHub. So, it is fair to assume that you already have a GitHub account and that you have an example repository created. If not, you can create a free GitHub account [here](https://github.com/) and push the sample project you might have on to your new repository or you can use the one I'm using through out this tutorial [here](https://github.com/chandanv89/Hateoas-Example).

If you don't have a Docker Hub account already, go ahead and create one for free [here](https://hub.docker.com/signup). Though there multiple [pricing options](https://www.docker.com/pricing) available, for this tutorial, the free version is all that you need. Once you verify your email ID and log back in, you'll be greeted with a landing page similar to the one below.

![Default Landing Page at https://hub.docker.com](/assets/images/post-2-docker-hub-homepage.png)

If this is the first time you're signing in to your Docker Hub account, you need to create your first repository for the docker image. Just like a GitHub code repository, a Docker Hub repository provides you the capability to version your Docker images. So, go ahead and click on the **Create a Repository** button.

![Create Repository Page](/assets/images/post-2-docker-hub-create-repository.png)

On the **Create Repository** page, fill in the repository name and description and select if it is a public or private repository. Note that with a free Hub account, you can have only one private account. Under optional **Build Settings**, you get options to connect your GitHub or BitBucket account. Since we're here enable auto triggering of the Docker image builds, let's go ahead and link GitHub (or, BitBucket) account.

After successfully linking your GitHub account, you can select the repository that you want to link to this build. In this case, let's select the project that we used in out previous article - SpringBoot Hateoas Demo application.

![Build Settings](/assets/images/post-2-docker-hub-build-rule.png)

Next, we need to define the a **build rule**. This will define how the new Docker images are versioned/tagged. You can get more idea on how these settings work for you, you can refer to the sample Build Rules under _View example build rules_. For now, I'm going with the **Branch** source type with my source branch as _main_ (not master) and all other options as given. Click on **Create**, to create your _Build Configuration_.

Now, for the repository, under Builds, you should be able to see the newly created configuration.

![Build Activity Page](/assets/images/post-2-docker-hub-build-activity.png)

Go ahead and trigger your first build by clicking the **Trigger** button under Automated Builds. Make sure you have provided the valid **Source** branch name - _master_ or _main_. Assuming everything is configured properly, your build should successfully be able to generate the Docker image with the configured _build rules_.

![Docker image created and tagged](/assets/images/post-2-docker-hub-tags.png)

You can now pull this image - _chandanv892/springboot-hateoas-demo_ - and run a container.

![Running the docker image locally](/assets/images/post-2-local-docker-pull.png)

To verify if the build is indeed triggered automatically upon each change that you push to your code repository, you can push a change and see a new image generated. As you can see below, the new commit _bfeb771_ has triggered a new build and is in progress.

![Recent builds](/assets/images/post-2-docker-hub-recent-builds.png)

Well done! You now have an auto-build configuration that takes care of building new Docker images on every change that you push to your repository.

Stay safe!