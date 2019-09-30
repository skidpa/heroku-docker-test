heroku-docker-test

#
rebuild docker with the application.properties set to
```` server.port=${PORT}````

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

