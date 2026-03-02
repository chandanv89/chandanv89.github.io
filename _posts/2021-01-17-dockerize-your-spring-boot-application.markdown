![](/assets/images/post-1-header-image.png)

In this article, I’d like to walk you through how you can create a Docker image for a demo Spring Boot application. And, subsequently, we'll see how to setup a Github trigger to auto build the Docker image using the Docker Hub infrastructure.

The demo application that we’re making use of in this example can be downloaded from [here](https://github.com/chandanv89/Hateoas-Example). It’s a simple Spring Boot application that illustrates HATEOAS links generation. How the app works, maybe that’s for another discussion. All we’ll be concentrating here is dockerizing it!

## Docker Image and Docker Container
For the uninitiated, Docker image is a self-contained deployment artefact that contains your application plus all the dependencies that the application needs to run. When you run this image on a Docker engine, you get a container that is running your application. A vague analogy could be an archived file (image) and its expanded contents (container).

![Dockerfile to Docker Image to Containers](/assets/images/post-1-docker-flow.png)

You define the steps to package your application along with its dependencies into a Docker image, using a _Dockerfile_. Once you have an image, you can run it on the Docker Engine to create multiple container instances.

## Why Dockerize?
One of the key benefits of Dockerizing your application is **quick and easy environment setup**. All you need is a Docker image and an instance of Docker Engine to run the image on. Since all the artefacts required to run your application are available within the image, **it also eliminates _"It works on my machine; but doesn't work on yours"_ problems**.

Also, it gives you a standard way of deployments and facilitates easier deployments across environments like development, staging and production. With proper use of tags, it even allows for better version control and easy rollbacks.

This is not an exhaustive list of advantages of dockerizing your application. There are many more. I'll leave for you to explore more on it.

## The Demo Application
The demo app can be download from [Github](https://github.com/chandanv89/Hateoas-Example). The app exposes a few RESTful endpoints at port 8080. The app is built using the Spring Framework v2.4.1 and all the required dependent JARs are pulled when you build it.

```
GET http://localhost:8080/rest/v1/customers
GET http://localhost:8080/rest/v1/customers/{customerId}
GET http://localhost:8080/rest/v1/customers/{customerId}/orders
GET http://localhost:8080/rest/v1/orders
GET http://localhost:8080/rest/v1/orders
GET http://localhost:8080/rest/v1/orders/{orderId}
```

## The Dockerfile Explained
While there are different ways to package your application, the one I'm using here generates the application JAR using the project source and then, copies only the resultant JAR to the container and runs it. That way, we can reduce the size of the resultant image and container. The downside of it, however, is that you cannot use the local cached maven repository. Each time you build the image, maven will download the required JARs from the central repository.

The root of this repository contains a few folders and files. We're interested in one file - `Dockerfile`. Let's take a look at its contents:

![cat Dockerfile](/assets/images/post-1-cat-dockerfile.png)

`FROM maven:3.6.3-openjdk-11-slim AS MAVEN_BUILD`: Use the slim version (on Alpine) of Maven v3.6.3 for OpenJDK-11. You can refer to the application JAR generated at this stage using the alias _MAVEN\_BUILD_.

`COPY ./ /opt/source`: Copies the entire project structure from the current folder (relative to where the Dockerfile resides) to a path on the container.

`RUN mvn clean package -f /opt/source/pom.xml`: Runs a given command on the container. In this case, generate the application JAR using the specified pom. This results in a fat JAR at `target/hateoas-example-1.0.0.jar`. This is our executable application.

`FROM openjdk:11-jre-slim`: Again, to run the application, we need a JRE. So, let's pull a slim version of the OpenJDK-11 JRE.

`COPY src/main/resources/UnlimitedJCEPolicyJDK8/*.jar /usr/lib/jvm/java-11-openjdk/jre/lib/security/`: In order to run this app on Docker Engine, you need to copy the relevant JCE JARs to the specified path on the container. You can download the JCE polilcy files from [Oracle](https://www.oracle.com/java/technologies/javase-jce8-downloads.html).

`COPY --from=MAVEN_BUILD /opt/source/target/*.jar /opt/app.jar`: Copies the generated jar from the previous stage to a custom path on the container. Let's call it `app.jar`.

`ENTRYPOINT ["/usr/local/openjdk-11/bin/java"]`: Specifies the entrypoint to your container - what needs to be run once the container is initialised. We need to run our application JAR file. So, let's start with **java** binary.

`CMD ["-jar", "/opt/app.jar"]` And pass a few arguments to the above command.

`EXPOSE 8080` And, expose the application port from container to the outside world.

That's a lot to take in? Just build the image and you'll see it in action!

![terminal-output](/assets/images/post-1-terminal-output.png)

Take a moment to see what's happening with the build, see how all the layers are coming together to create the Docker image. The last line tells you that the image is built successfully, with id `8f148...` You can verify it by running `docker images` command.

![docker-images](/assets/images/post-1-docker-images.png)

The image `16f...`, as you can see above, is for Maven, with all the dependencies downloaded and is substantially a bigger image at 600+MB. While the image `8f1...`, with JDK and the JAR built previously with the Maven stage is less than 250MB in size.

You can run the image and create a container now with `docker run -dp 8080:8080 8f14`. With `-d` option, you can run the container detached (background) from the terminal and with option `-p`, you're exposing the application port `8080` to the host port `8080`. `8f14` is a shorthand image ID. You just need to specify enough characters from the image id to uniquely identify the image.

![docker-run](/assets/images/post-1-docker-run.png)

Your Spring Boot application is now up and running as a docker container! Give it a try...

![API Response on localhost](/assets/images/post-1-localhost.png)

In the next article, I'll help you setup auto builds to create images on Docker Hub infrastructure. Keep safe!