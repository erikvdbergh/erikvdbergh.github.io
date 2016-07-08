---
layout: post
title:  "Remove an images and all associated containers in Docker"
date: 2016-05-24 11:35:43
categories: docker
tags: docker bash
image: /images/pic01.jpg
---

In this post I'll explore some Docker automation for deleting old containers that depend on an image you want to remove. 

<!-- more --> Sometimes you have been building an image a few times in different iterations and trying it out. When you want to clean up, you want to remove the image so you run `docker rmi`:

```
vandene@computer:~$ docker rmi mybuiltimage
Error response from daemon: conflict: unable to remove repository reference "mybuiltimage" (must force) - container 8ac1a3a4aa41 is using its referenced image 90d5884b1ee0
vandene@computer:~$ docker rm 8ac1a3a4aa41
8ac1a3a4aa41
vandene@computer:~$ docker rmi mybuiltimage
Error response from daemon: conflict: unable to remove repository reference "mybuiltimage" (must force) - container aa656b16b7bb is using its referenced image 90d5884b1ee0
vandene@computer:~$ docker rm aa656b16b7bb
aa656b16b7bb
vandene@computer:~$ 
```

This is annoying because you have to keep removing the containers one by one, and trying `docker rmi` in between each time to see which one is conflicting next. If you have 10 exited containers from the build process, this becomes tedious. I've written a small bash function to add to your `~/.profile` which introduces a `dockerpurge` command that removes an image and all associated containers in one go:

{% highlight bash %}
function dockerpurge() {
  for cont in `docker ps -a | grep $1 | awk '{print $1}'`; do
    echo removing container $cont
    docker rm $cont
  done 
  echo removing image $1
  docker rmi $1
}
{% endhighlight %}

Now, you can just run the dockerpurge command:

```
vandene@computer:~$ dockerpurge mybuiltimage
removing container 3b4695a71456
3b4695a71456
removing container e1308263aec0
e1308263aec0
removing container 0ffcb7cd8700
0ffcb7cd8700
removing container aa656b16b7bb
aa656b16b7bb
removing container 70c98faf6ae5
70c98faf6ae5
removing container ee2d58fa7dfa
ee2d58fa7dfa
removing container 11f3eb599652
11f3eb599652
removing container 3dbf3f5f55e6
3dbf3f5f55e6
removing container f1202d7c923c
f1202d7c923c
removing image 90d5884b1ee0
Deleted: sha256: 690d5884b1ee0806d59741b4e7e85bcb50339d4746c41aa88abedfb41aaa
```

Should make life a bit easier.
