# Step : Test and packages
FROM maven:3.5.3-jdk-8-alpine as build
WORKDIR /build
COPY pom.xml .
COPY src/ /build/src/
RUN mvn package -DskipTests

# Step : Package image
FROM openjdk:8-jre-alpine
WORKDIR /app
COPY --from=build /build/target/*.jar /app
EXPOSE 8080
#simple hack to avoid using the full filename as parameter
RUN echo "java -jar /app/*.jar" > /app/docker-entrypoint.sh
ENTRYPOINT ["sh", "/app/docker-entrypoint.sh"]