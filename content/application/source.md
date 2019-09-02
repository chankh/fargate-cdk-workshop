---
title: "Source"
date: 2018-08-07T12:37:34-07:00
weight: 10
---

The sample Spring application that we will use in this workshop is the [Spring 
PetClinic](https://github.com/spring-projects/spring-petclinic) sample
application. PetClinic is a Spring Boot application built using Maven. Cloud9
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

Now you can download the source code, build a jar file and run it from the
command line:

```
cd ~/environment
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar
```

You can then access PetClinic by previewing the application from Cloud9. Click
**Preview** in the menu and select **Preview Running Application**. You should
see PetClinic running in your browser window.

![](/images/application/preview.png)

Press `ctrl-c` to stop the application and proceed to next step.
