heroku-docker-test

# Push and build docker image to heroku
set the port in application.properties to (might have to create the file in main/resources)
```` server.port=${PORT}````

and in the Dockerfile change the following
```` ARG DEPENDENCY=target/dependency ````
to
```` ARG DEPENDENCY=build/docker/dependency ````

in IntelliJ this path can be found by first building the docker and then
in the Project tree open up build/docker and right click on dependency 
then select copy relative path shortcut: ``Ctrl + Alt + Shift + C``

change ``ENTRYPOINT`` to ``CMD``
so that it looks like this 

```` CMD ["java","-cp","app:app/lib/*","package_name.Application_name"] ````

got the CMD thing from
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#cmd
have no clue if this is good or bad but hey it works...

so now the Dockerfile looks like this

````
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=build/docker/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
CMD ["java","-cp","app:app/lib/*","se.experis.Application"]
ENTRYPOINT ["java","-cp","app:app/lib/*","se.experis.Application"]

````
#### _note:_ 
_as long as the Docker file contains either bot the CMD and ENTRYPOINT
it seems to work regardless of order but we could just run with only the CMD and it would still work_


#### inside gitbash or whatever

build the docker this could also be done in intelliJ

login to heroku.
````heroku container:login````
which should respond successful

Create your heroku app
````heroku create appname --region en````

push and build the thing
``heroku container:push --app pa-annoying-docker web``

this should now return something like
````
=== Building web (C:\Users\pakarl\Desktop\herokushit\heroku-docker-test\Dockerfile)
Sending build context to Docker daemon  113.6MB
Step 1/7 : FROM openjdk:8-jdk-alpine
 ---> a3562aa0b991
Step 2/7 : VOLUME /tmp
 ---> Using cache
 ---> 34c88f51ee4d
Step 3/7 : ARG DEPENDENCY=build/docker/dependency
 ---> Using cache
 ---> 3cbafd619a66
Step 4/7 : COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
 ---> 267b1a08ceb6
Step 5/7 : COPY ${DEPENDENCY}/META-INF /app/META-INF
 ---> 3cae04af98af
Step 6/7 : COPY ${DEPENDENCY}/BOOT-INF/classes /app
 ---> 686a3f214493
Step 7/7 : ENTRYPOINT ["java","-cp","app:app/lib/*","se.experis.Application"]
 ---> Running in 3ec33176593d
Removing intermediate container 3ec33176593d
 ---> e75669fcd41d
Successfully built e75669fcd41d
Successfully tagged registry.heroku.com/pa-annoying-docker/web:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
=== Pushing web (C:\Users\pakarl\Desktop\herokushit\heroku-docker-test\Dockerfile)
The push refers to repository [registry.heroku.com/pa-annoying-docker/web]
4c8d464c4923: Preparing
3aae0e105dac: Preparing
4f8fcb4322eb: Preparing
ceaf9e1ebef5: Preparing
9b9b7f3d56a0: Preparing
f1b5933fe4b5: Preparing
f1b5933fe4b5: Waiting
ceaf9e1ebef5: Mounted from pa-heroku-docker-test/web
3aae0e105dac: Pushed
4c8d464c4923: Pushed
9b9b7f3d56a0: Pushed
f1b5933fe4b5: Mounted from pa-heroku-docker-test/web
4f8fcb4322eb: Pushed
latest: digest: sha256:d460bd9b60562416e14a79d31dd0253aea9e860781e35c173786d73632384264 size: 1573
Your image has been successfully pushed. You can now release it with the 'container:release' command.

````

#### _note: this is when the Dockerfile contains only the ``CMD...``_
_Above at step 7 it looks like the  ``CMD...`` has changed back to ``ENTRYPOINT``
 according to the website linked above_
 
 > Understand how CMD and ENTRYPOINT interact
 
 > Both CMD and ENTRYPOINT instructions define what command gets executed when running a container. There are few rules that describe their co-operation.
 
 > 1. Dockerfile should specify at least one of CMD or ENTRYPOINT commands.
 
 > 2. ENTRYPOINT should be defined when using the container as an executable.
 
 > 3. CMD should be used as a way of defining default arguments for an ENTRYPOINT command or for executing an ad-hoc command in a container.
 
 > 4. CMD will be overridden when running the container with alternative arguments.

So i assume there are some magical wizardry going on behind the scenes
because the push with ``ENTRYPOINT`` instead of ``CMD`` in the
``Dockerfile`` seems to work fine but when we try to (or at least when I do) release the thing we get the following error.

````
(node:3160) UnhandledPromiseRejectionWarning: Error: Expected response to be successful, got 422
    at Request.handleFailure [as _handleFailure] (C:\Program Files\heroku\client\node_modules\heroku-client\lib\request.js:254:11)
    at Request.<anonymous> (C:\Program Files\heroku\client\node_modules\heroku-cli-util\lib\command.js:56:12)
    at concat.then (C:\Program Files\heroku\client\node_modules\heroku-client\lib\request.js:148:16)
    at processTicksAndRejections (internal/process/task_queues.js:86:5)
(node:3160) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:3160) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
Releasing images web to pa-annoying-docker... !
 !    No command specified for process type web

```` 

then finally release the thing to the world with

```heroku container:release web -a pa-annoying-docker```

and if that's successful we will get some thing like

````Releasing images web to pa-annoying-docker... done```` 




# Push docker image to heroku
rebuild docker with the application.properties set to
```` server.port=${PORT}````

and in the Dockerfile
change ``ENTRYPOINT`` to ``CMD``
so that it looks like this

````
CMD ["java","-cp","app:app/lib/*","package_name.Application_name"]
````

got the CMD thing from
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#cmd
have no clue if this is good or bad but hey it works...

inside gitbash or whatever
````heroku create appname --region en````

then login.
````heroku container:login````
which should respond successful

then do a docker login thing

````
docker login --username=yourdockername --password=$(heroku auth:token) registry.heroku.com
````

should respond with something like
````
 »   Warning: token will expire 09/29/2020
 »   Use heroku authorizations:create to generate a long-term token
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded

````

then tag the docker image
````
docker tag your_docker_id/docker_img_name registry.heroku.com/heroku_app_name
````

and then finally push the image to heroku
````
docker push registry.heroku.com/heroku_app_name/web
````

note that web is a heroku process type.

the respons will be something like this
````
The push refers to repository [registry.heroku.com/heroku_app_name/web]
2f9ec839efe4: Preparing
9ae0bde26d5b: Preparing
335ec8cb38bc: Preparing
ceaf9e1ebef5: Preparing
9b9b7f3d56a0: Preparing
f1b5933fe4b5: Preparing
f1b5933fe4b5: Waiting
9b9b7f3d56a0: Mounted from heroku_app_name/web
ceaf9e1ebef5: Mounted from heroku_app_name/web
335ec8cb38bc: Mounted from heroku_app_name/web
2f9ec839efe4: Pushed
9ae0bde26d5b: Pushed
f1b5933fe4b5: Mounted from heroku_app_name/web
latest: digest: sha256:6892bcd0b848bfdde8723ad96be9303b7293392deaad1586915385b8622201a4 size: 1573
````

then we release the thing to the world.

````
heroku container:release -a heroku_app_name web
````

which might respond with
````
Releasing images web to heroku_app_name... done
````

