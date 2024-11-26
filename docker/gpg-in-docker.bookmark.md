# Using GPG inside a Docker container (nixaid.com)

<https://nixaid.com/using-gpg-inside-a-docker-container/>

## Tags

#docker #gpg

## Content

# Using GPG inside a Docker container {#using-gpg-inside-a-docker-container .reader-title}

Andy

------------------------------------------------------------------------

![Using GPG inside a Docker container](https://images.unsplash.com/photo-1500313749317-0105e4b72ec1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&fit=max&s=79823ee7a75c2c4e5f87b89a540e8ff5&w=2000){sizes="(min-width: 1400px) 1400px, 92vw" srcset="https://images.unsplash.com/photo-1500313749317-0105e4b72ec1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&fit=max&s=79823ee7a75c2c4e5f87b89a540e8ff5&w=300 300w,
                            https://images.unsplash.com/photo-1500313749317-0105e4b72ec1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&fit=max&s=79823ee7a75c2c4e5f87b89a540e8ff5&w=600 600w,
                            https://images.unsplash.com/photo-1500313749317-0105e4b72ec1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&fit=max&s=79823ee7a75c2c4e5f87b89a540e8ff5&w=1000 1000w,
                            https://images.unsplash.com/photo-1500313749317-0105e4b72ec1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&fit=max&s=79823ee7a75c2c4e5f87b89a540e8ff5&w=2000 2000w"}

One day you may need to run a GPG 2.x inside a Docker container.

In Ubuntu Artful the gpg-agent creates its agent-socket under `/run/user`, instead\
of `$GNUPGHOME/` which by default is `~/.gnupg`.

You can check what path does your gpg agent-socket use by running:

    $ gpgconf --list-dirs | grep agent-socket
    agent-socket:/run/user/1000/gnupg/S.gpg-agent

Check whether your current GPG supports a `socketdir` by running:

-   ubuntu-artful with gpg 2.1.15

<!-- -->

    $ gpgconf --dry-run --create-socketdir
    gpgconf: socketdir is '/run/user/1000/gnupg'

-   ubuntu-xenial with gpg 2.1.11

<!-- -->

    $ gpgconf --dry-run --create-socketdir
    gpgconf: invalid option "--create-socketdir"

There are two ways of using GPG 2.x in a Docker container.

Before you continue, make sure you have the image with the GPG 2.x installed.

-   Create the new image with GPG 2.x:

<!-- -->

    docker run --rm -ti ubuntu:artful bash
    apt-get update && apt-get -y install gnupg2
    docker commit $(docker ps -lq) ubuntu:artful-gpg2

-   A\) Using a running gpg-agent on your host:

<!-- -->

    docker run --rm -ti -u $(id -u):$(id -g) -v ${HOME}/.gnupg/:/.gnupg/:ro \
               -v /run/user/$(id -u)/:/run/user/$(id -u)/:ro \
           ubuntu:artful-gpg2 bash

-   B\) Running a new gpg-agent instance inside a container:

<!-- -->

    docker run --rm -ti -u $(id -u):$(id -g) -v ${HOME}/.gnupg/:/.gnupg/:ro \
               --tmpfs /run/user/$(id -u)/:mode=0700,uid=$(id -u),gid=$(id -g) \
               ubuntu:artful-gpg2 bash
    $ gpg-agent --daemon

Now that you picked either path A or B, you should be ready to use the GPG.

    gpg2 -K
    echo test | gpg2 -e -a -r recipient | gpg2 -d
    echo test | gpg2 --clearsign

Please feel free to comment in case of questions or suggestions.

#### 3 Comments

Type Comment Here (at least 3 chars)
