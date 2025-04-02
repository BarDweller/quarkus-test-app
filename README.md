# A simple test project

This project is designed to allow containerised build testing with developer builds of quarkus. 

## Why?

If you build quarkus locally, you end up with quarkus as version 999-SNAPSHOT in your local maven repo. You can then use this version of quarkus with test projects to try out your updates to quarkus. This all works fine locally, but if you are trying to test aspects of the quarkus build, that will be run within a container, then your custom 999-SNAPSHOT version of quarkus will not be know to the maven instance running inside the container, and the build inside will fail. 

## How?

This project uses maven profiles, to allow selection of the 999-SNAPSHOT level of quarkus only when `-Ddevbuild=true` is passed to maven. This way the local build can be told to use the development build, while the in-container build will use the release build. Because the development build is only needed to initiate the container image build, this allows the application to build as expected. 

## What?

`./mvnw clean package -Dquarkus.container-image.build=true -Ddevbuild=true`

## Debugging...

The extension code is using JBoss logging api, which is sending to Maven Mojo logger, which allows configuration via global levels, eg, `mvn -X` .. this will output a vast amount of log info, but you can switch the logger to the Maven slf4j backend, which then allows for per package debug.. 

_(at the cost of some formatting, such as the name of the logger being part of the output. Although you can add that back using `-Dorg.slf4j.simpleLogger.showLogName=true` this will then show the logger for ALL output, including the normal expected output from maven itself, which can look a little odd)_

`./mvnw clean package -Dquarkus.container-image.build=true -Ddevbuild=true -Dorg.jboss.logging.provider=slf4j -Dorg.slf4j.simpleLogger.log.io.quarkus.container.image.buildpack.deployment=DEBUG`

## Testing buildpack container-image extension options

If testing the container image buildpack extension, with registry mode, or push ability, you may wish to pass credentials in various ways to ensure the configuration of the underlying snowdrop lib is as expected.

Quarkus Stock approach.. 

```bash
export IMAGE="myregistry/stiletto/patent:shiny"
export REGISTRY_USER=myusername
export REGISTRY_PASSWORD=mypassword
./mvnw install -Ddevbuild=true -Dquarkus.container-image.build=true -Dquarkus.container-image.image=${IMAGE} -Dquarkus.container-image.username=${REGISTRY_USER} -Dquarkus.container-image.password=${REGISTRY_PASSWORD}
```

Per registry approach..
```bash
export IMAGE="myregistry/stiletto/patent:shiny"
export REGISTRY=quay.io
export REGISTRY_USER=myusername
export REGISTRY_PASSWORD=mypassword
./mvnw install -Ddevbuild=true -Dquarkus.container-image.build=true -Dquarkus.container-image.image=${IMAGE} -Dquarkus.buildpack.registry-user.\"${REGISTRY}\"=${REGISTRY_USER} -Dquarkus.buildpack.registry-password.\"${REGISTRY}\"=${REGISTRY_PASSWORD}
```


