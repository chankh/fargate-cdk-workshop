---
title: "Source"
weight: 10
---

The sample Spring application that we will use in this workshop is the [Spring 
PetClinic](https://github.com/spring-projects/spring-petclinic) sample
application. 

Download the application source code from GitHub.

```
cd ~/environment
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
```

#### Optional: Build and run locally on Cloud9

PetClinic is a Spring Boot application built using Maven. Cloud9
comes with OpenJDK version 1.7 install, we will need to install version 1.8 in
order to build and run PetClinic.
```
sudo yum install -y java-1.8.0-openjdk-devel
sudo alternatives --config java
```

At the prompt, enter the number for openjdk 1.8.0, in most cases, it
should be **2** like below.
![java](/images/application/java.png)

Now do the same for `javac` command
```
sudo alternatives --config javac
```

Now you can build the application using Maven and run the jar file:

```
./mvnw package
java -jar target/*.jar
```

You can then access PetClinic by previewing the application from Cloud9. Click
**Preview** in the menu and select **Preview Running Application**. You should
see PetClinic running in your browser window.

![](/images/application/preview.png)

Press `ctrl-c` to stop the application and proceed to next step.
