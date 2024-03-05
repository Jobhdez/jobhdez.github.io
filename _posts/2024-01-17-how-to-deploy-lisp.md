---
layout: post
title: "How to deploy a Lisp server to AWS"
author: Job Hernandez
tags: aws how-to
---

### Intoduction

I have been programming server side programs with Lisp and the Hunchentoot web server for a while. I am writing this blog post as a future reminder for myself and help anybody out there with the basics.  

The way I managed to deploy my Lisp server was by using docker and building the image within the EC2 instance. Once you run the image you  navigate to the EC2 instance's public IP and you will see your server there.

But some disclaimers:

My friend who has around 20 years of experience and is really good said that for production you probably do not need to manage docker yourself. This is what he said:

> specifically the thing that makes it not for production is that you probably don't want to be managing docker yourself on an ec2 instance. It's not completely out of the question if you really really know what you're doing, but you need to deal with things like the ec2 instance rebooting, or the process crashing, or the docker host process crashing.
ECS and EKS are services (one native to amazon, the other using kubernetes) that will manage a lot of that for you and more

The problem with this is that if you use this approach for an app that cannot go down then if the EC2 instance restarts your app will be down because you have type `docker run` to restart your server again. But since my app is not critical I used an EC2 instance.

My friend did offer an alternative:

> next time around, rather than using ec2 at all, I would push the image to ecr (its a private image repository) and then build an ecs instance that starts that image - that way you don't have to worry about docker at all beyond builiding the image

### The Dockerfile
I happened to stumble accross this [video](https://www.youtube.com/watch?v=QuG2ByK-Cwg&t=390s) and I was able to make the following Dockerfile for my project:

```
FROM clfoundation/sbcl:alpine3.14 as builder

COPY . /root/common-lisp/
WORKDIR /root/common-lisp/

# https://www.reddit.com/r/Common_Lisp/comments/pdsqbe/installing_quicklisp/
# https://github.com/yitzchak/common-lisp-jupyter/blob/master/Dockerfile
ENV QUICKLISP_ADD_TO_INIT_FILE=true
RUN sbcl --non-interactive --load quicklisp2.lisp \
      --eval "(quicklisp-quickstart:install)" \
      --eval "(ql-util:without-prompting (ql:add-to-init-file))"

RUN sbcl --eval "(ql:quickload :compiler-web)" \
         --load server.lisp \
         --eval "(sb-ext:save-lisp-and-die \"core\" :toplevel #'lambda-server::start-server :executable t)"

FROM clfoundation/sbcl:alpine3.14

RUN adduser -D app
USER app

COPY --from=builder /root/common-lisp/core .

EXPOSE 4243

ENTRYPOINT [ "sbcl", "--core", "core" ]
```

You are going to have to change the package name in

```
RUN sbcl --eval "(ql:quickload :compiler-web)" \
         --load server.lisp \
         --eval "(sb-ext:save-lisp-and-die \"core\" :toplevel #'lambda-server::start-server :executable t)"
```

and load your server and replace `'lambda-server::start-server` with your function that starts the server. Other than that you can keep it as it is. You also need to change `EXPOSE 4243` to `EXPOSE <your-lisp-server-port>`.

### EC2

Once you have the Dockerfile you then need to make an AWS account and launch an EC2 instance. You should keep it within the free tier limits and use a linux instance.

Once you have launched the EC2 instance you need install git and docker:

```
* sudo yum update -y
* sudo yum install docker -y
* sudo service docker start
* sudo usermod -a -G docker ec2-user
* sudo yum install git -y
```

Now, you need to clone your github repo from the EC2 instance.

Your repo should contain a `Dockerfile`.

Then you need to build it so you need to move to your repo's directory and type:

```
* sudo docker build -t compiler-web .
* sudo docker run -p 80:4243 --rm compiler-web
```

 By doing `-t` you are tagging your image. In my case the tag for my image is `compiler-web`. You can change this as you wish. My Lisp server is listening at the port `4243` so you need to change this to your port. But keep the port 80 since this the port of your EC2 instance I believe.

After trying to rebuild your image again you may get a cache error. To solve this you can build it like this:

```
* sudo docker build --no-cache -t compiler-web .
```

After you have built the docker image you just need to point to the EC2's public IP and you should be able to see your server.

To deploy, you need to setup an Elastic IP within your EC2 instance and then associate the Elastic IP to an A record (in DNS settings in your domain provider) with a `@`. After this you just wait until the changes propagates.

### Conclusion

I hope this was helpful :-). Have a nice day.
