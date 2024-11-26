# Deploying firecracker VMs (dev.to)

<https://dev.to/librehash/deploying-firecracker-vms-1d99>

## Description

GitHub URL for the project - https://github.com/firecracker-microvm/firecracker Description -...

## Tags

#firecracker #vm

## Content

# Deploying Firecracker VMs {#deploying-firecracker-vms .reader-title}

librehash

------------------------------------------------------------------------

GitHub URL for the project - [https://github.com/firecracker-microvm/firecracker](https://github.com/firecracker-microvm/firecracker){rel="noopener noreferrer"}

**Description** - \"*Secure and fast microVMs for serverless computing*.\"

Also - \"*Firecracker is an open source virtualization technology that is purpose-built for creating and managing secure, multi-tenant container and function-based services that provide serverless operational models. Firecracker runs workloads in lightweight virtual machines, called microVMs, which combine the security and isolation properties provided by hardware virtualization technology with the speed and flexibility of containers*.\"

### [](#more-on-the-project){#more-on-the-project} More on the Project

Written in Rust (that\'s a delectable plus). Looks like this was actually created by Amazon, I believe (still open source ; relax Pedro).

Yup, confirmed within the Git repo - \"Firecracker was developed at Amazon Web Services to accelerate the speed and efficiency of services like AWS Lambda and AWS Fargate. Firecracker is open sourced under Apache version 2.0.\"

The VM operates with a VMM (virtual machine monitor), which leverages the KVM (Linux) to create & run \'microVMs\'.

1.  The virtualized environments are designed to be secure by default (according to the project specs); this is done by \"\[excluding\] unnecessary devices and guest-facing functionality to reduce the memory footprint and attack surface area of each microVM\".

2.  Firecracker can be integrated with other container runtimes like \'Kata Containers\' (this is another project that we\'re going to get to in a second) + \'Weaveworks Ignite\' (fuck them)

More information on the project can be found here - [https://firecracker-microvm.github.io/](https://firecracker-microvm.github.io/){rel="noopener noreferrer"}

**More Helpful Info About the Project**

1.  Supports all modern processor types pretty much ; Intel / AMD / ARM (last one is a present surprise as well)

2.  Is built w security in mind in its design (via the minimalism of the VM creation); also, its meant to isolate VM instances entirely from the actual OS / root system in a manner that plugs up leaks pretty comprehensively.

### [](#more-on-the-inner-workings-of-firecracker){#more-on-the-inner-workings-of-firecracker} More on the Inner Workings of \'FireCracker\'

Below is a picture from the main site that the project claims illustrates an \"example host running Firecracker microVMs\".

[![Image description](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fndc7lmz9juj4w2e06em8.png){height="457" width="800" loading="lazy"}](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fndc7lmz9juj4w2e06em8.png)

-   Claim that the VMs use so little resources that one could plausibly \"run thousands of microVMs onto the same machine\"

-   \"This means that every function, container, or container group can be encapsulated with a virtual machine barrier, enabling workloads from different customers to run on the same machine, without any tradeoffs to security or efficiency\" (okay! this is a pretty big claim, but I like it ; would like to see the 3rd-party evaluations of how much this project adheres to that promise in practice)

-   \"Each Firecracker microVM is further isoalted with common Linux user-space security barriers by a companion p rogram called \'jailer\'\"; \'the jailer provides a second line of defense in case the virtualization barrier is ever compromised\' (so they\'re really gun-ho on the idea that the VMs constructed with this tool are of a caliber capable of preventing any and all \'escapes\' / \'leaks\' from the VMs \[people will always try to find a way though\]

-   \"The Firecracker VMM is built to be processor agnostic. Intel processors are supported for production workloads\" (that last statement implies that this project is only built for Intel in any practical sense) ; especially with the follow-up that, \"Support for AMd and ARM processors is in \'developer preview\'\" (what\'s that?)

-   The virtualization software that they most closely compare this tool to is \'QEMU\' (which is fucking amazing, I will accept **zero slander on QEMU ever or on the developer - Fabian - that created QEMU virtualization because he\'s a fucking genius when you look at what he\'s managed to do**)

There are a \'gaggle\' of projects that are also able to run \'Firecracker\'. Those are (from the website):

1.  \'Appfleet\'

2.  \'firecracker-containerd\'

3.  \'fly.io\'

4.  \'koyeb\'

5.  \'kata containers\'

6.  \'OpenNebula\' (used this a couple of times; wasn\'t a huge fan of it although it was easy to setup at the time)

7.  \'Qovery\'

8.  \'UniK\'

9.  \'Weave FireKube\' + \'Ignite\'

### [](#how-to-actually-deploy-firecracker){#how-to-actually-deploy-firecracker} How to Actually Deploy Firecracker

Wasn\'t even going to get this fire down the rabbit hole, but these folks tempted me - so here goes nothing.

> \*\*\"You can build Firecracker on any Unix/Linux system that has Docker running and \'bash\' installed\" (using the following code):\

    git clone https://github.com/firecracker-microvm/firecracker
    cd firecracker
    tools/devtool build
    toolchain="$(uname -m)-unknown-linux-musl"

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

Additionally, \"*The Firecracker binary will be placed at build/cargo_target/\${toolchain}/debug/firecracker. For more information on building, testing, and running Firecracker, go to the quickstart guide.*\"

**Quick Start Guide** - [https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md){rel="noopener noreferrer"}

### [](#quick-start-guide-amp-instructions-once-the-initial-build-installation-code-has-been-run){#quick-start-guide-amp-instructions-once-the-initial-build-installation-code-has-been-run} Quick Start Guide & Instructions Once the Initial Build + Installation Code Has Been Run

> *Firecracker uses KVM and needs read/write access that can be granted as shown below*\

    sudo setfacl -m u:${USER}:rw /dev/kvm

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

Also need to ensure that the user running the VMs has \"read write access to \'/dev/kvm\'\"

**Directions on How to Set Up Read/Write Access to /dev/kvm** - [https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#appendix-a-setting-up-kvm-access](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#appendix-a-setting-up-kvm-access){rel="noopener noreferrer"}

> *\"Firecracker is linked statically against musl, having no library dependencies. You can just download the latest binary from our release page, and run it on your x86_64 or aarch64 Linux machine.*\"

(Once the firecracker binary has been downloaded \[instructions above\], we just need to ensure that we\'ve moved the binary into `/usr/bin` so that we can use it liberally in CLI)

> **Additionally, here\'s a \'building from the source section** - [https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#building-from-source](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#building-from-source){rel="noopener noreferrer"}

### [](#running-firecracker){#running-firecracker} Running Firecracker

> \"*In production, Firecracker is designed to be run securely, inside an execution jail, carefully set up by the jailer binary. This is how our integration test suite does it. However, if you just want to see Firecracker booting up a guest Linux machine, you can do that as well.*\"

1.  We need to first obtain an \"uncompressed Linux kernel binary, and an ext4 file system image (to use as rootfs)\" ; great, these are two things that we need to seek out before we move forward in our \'adventure\' (*this really feels like a \"quest\" of some sort, like the ones that they forced you to play on Runescape back in the days*)

**How to Decompress Linux Kernel** (explicit instructions to be honest here) - [https://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-5.html](https://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-5.html){rel="noopener noreferrer"}

**Linux-Hardened Kernel** - [https://github.com/anthraxx/linux-hardened](https://github.com/anthraxx/linux-hardened){rel="noopener noreferrer"} (this is something that they\'re all still actively working on at this very point in time)

They also say that we need an \'ext4 file system image\' (where do we obtain this from?) - found it

**Full Guide on How to Create an EXT4 filesystem image here** -[https://fabianlee.org/2020/01/13/linux-mounting-a-loopback-ext4-xfs-filesystem-to-isolate-or-enforce-storage-limits/](https://fabianlee.org/2020/01/13/linux-mounting-a-loopback-ext4-xfs-filesystem-to-isolate-or-enforce-storage-limits/){rel="noopener noreferrer"}

Assuming that the above has been handled, the directions insist that we create two separate shell prompts, (one to run Firecracker, and another one to control it \[by writing to the API socket\]; both shells have to run \"in the same directory where the firecracker binary was placed\")

\^\^ What?

-   This is a pain in the ass because this is something that they should\'ve mentioned earlier (obv. everyone is going to move a binary where the rest of their binaries go ; and you\'re not going to just load up some random project to be used in that manner)

-   Not even sure what the end goal of opening up an API socket here would really be

But fuck it, let\'s just assume that we play ball and we adhere to all of these (additional) steps that we\'re being put through (just for the setup up this virtualization tool!).

### [](#following-through-on-the-next-steps){#following-through-on-the-next-steps} Following Through on the Next Steps

1.  Ensuring that Firecracker can create its own API: `rm -f /tmp/firecracker.socket`

2.  Then start Firecracker: `./firecracker --api-sock /tmp/firecracker.socket`

\^\^ That\'s just for the \"first CLI window\" ; for the second one, we must do the following.

-   \"Set the guest kernel (assuming you are in the same directory as the above script was run)\"; they\'re saying this because they provided code for folks that don\'t have the necessary kernel image to be running this out of the gate - but we will be picking up a kernel from elsewhere. Either way, the following code should be copasthetic

<!-- -->

    arch=`uname -m`
    kernel_path=$(pwd)"/hello-vmlinux.bin"

    if [ ${arch} = "x86_64" ]; then
        curl --unix-socket /tmp/firecracker.socket -i \
          -X PUT 'http://localhost/boot-source'   \
          -H 'Accept: application/json'           \
          -H 'Content-Type: application/json'     \
          -d "{
                \"kernel_image_path\": \"${kernel_path}\",
                \"boot_args\": \"console=ttyS0 reboot=k panic=1 pci=off\"
           }"
    elif [ ${arch} = "aarch64" ]; then
        curl --unix-socket /tmp/firecracker.socket -i \
          -X PUT 'http://localhost/boot-source'   \
          -H 'Accept: application/json'           \
          -H 'Content-Type: application/json'     \
          -d "{
                \"kernel_image_path\": \"${kernel_path}\",
                \"boot_args\": \"keep_bootcon console=ttyS0 reboot=k panic=1 pci=off\"
           }"
    else
        echo "Cannot run firecracker on $arch architecture!"
        exit 1
    fi

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

-   \"Set the guest rootfs\"

<!-- -->

    rootfs_path=$(pwd)"/hello-rootfs.ext4"
    curl --unix-socket /tmp/firecracker.socket -i \
      -X PUT 'http://localhost/drives/rootfs' \
      -H 'Accept: application/json'           \
      -H 'Content-Type: application/json'     \
      -d "{
            \"drive_id\": \"rootfs\",
            \"path_on_host\": \"${rootfs_path}\",
            \"is_root_device\": true,
            \"is_read_only\": false
       }"

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

-   \"Start the guest machine\"

<!-- -->

    curl --unix-socket /tmp/firecracker.socket -i \
      -X PUT 'http://localhost/actions'       \
      -H  'Accept: application/json'          \
      -H  'Content-Type: application/json'    \
      -d '{
          "action_type": "InstanceStart"
       }'

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

> **side note**: seeing \'curl\' here reminded me that that\'s something that needs to be downloaded & instantiated on our default Docker images (Rust version specifically)

From here, they state, \"Going back to your first shell, you should now see a serial TTY prompting you to log into the guest machine\" (guessing that this will be the case for us too - basic Ubuntu image should be more than necessary)

Also, \"*When you\'re done, issuing a reboot command inside the guest will actually shutdown Firecracker gracefully. This is due to the fact that Firecracker doesn\'t implement guest power management.*\"

They also include this one note: \"*The default microVM will have 1 vCPU and 128 MiB RAM.*\" ; this obviously isn\'t enough for what we want to do - but that\'s no problem as this can be configured w relative ease.

They state, \"**If you wish to customize that (say, 2 vCPUs and 1024MiB RAM), you can do so before issuing the InstanceStart call, via this API command**\" (that\'s annoying that we need to use API commands here for that, but that\'s cool):\

    curl --unix-socket /tmp/firecracker.socket -i  \
      -X PUT 'http://localhost/machine-config' \
      -H 'Accept: application/json'            \
      -H 'Content-Type: application/json'      \
      -d '{
          "vcpu_count": 2,
          "mem_size_mib": 1024,
          "ht_enabled": false
      }'

![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkVudGVyIGZ1bGxzY3JlZW4gbW9kZTwvdGl0bGU+CiAgICA8cGF0aCBkPSJNMTYgM2g2djZoLTJWNWgtNFYzek0yIDNoNnYySDR2NEgyVjN6bTE4IDE2di00aDJ2NmgtNnYtMmg0ek00IDE5aDR2Mkgydi02aDJ2NHoiIC8+Cjwvc3ZnPg==)
![](data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgMjQgMjQiIGhlaWdodD0iMjBweCIgd2lkdGg9IjIwcHgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPkV4aXQgZnVsbHNjcmVlbiBtb2RlPC90aXRsZT4KICAgIDxwYXRoIGQ9Ik0xOCA3aDR2MmgtNlYzaDJ2NHpNOCA5SDJWN2g0VjNoMnY2em0xMCA4djRoLTJ2LTZoNnYyaC00ek04IDE1djZINnYtNEgydi0yaDZ6IiAvPgo8L3N2Zz4=)

### [](#configuring-the-microvm-without-sending-api-requests){#configuring-the-microvm-without-sending-api-requests} Configuring the microVM Without Sending API Requests

> *they must have read my mind!*

\"*If you\'d like to boot up a guest machine without using the API socket, you can do that by passing the parameter \--config-file to the Firecracker process. The command for starting Firecracker with this option will look like this:*\" `./firecracker --api-sock /tmp/firecracker.socket --config-file <path_to_the_configuration_file>`

They claim that the `path_to_configuration_file`, \"should represent the path to a file that contains a JSON which stores the entire configuration for all of the microVM\'s resources\" (okay this is fair enough).

Also, they stipulate, \"**The JSON must contain the configuration for the guest kernel and rootfs, as these are mandatory, but all of the other resources are optional, so it\'s your choice if you want to configure them or not. Because using this configuration method will also start the microVM, you need to specify all desired pre-boot configurable resources in that JSON.**\"

**File Names for the Pre-Boot Resources** (included within the greater repo here):

1.  `firecracker.yaml` - Names of resources are contained here ; \'file and the names of their fields are the same that are used in API requests\' (cool)

2.  `tests/framework/vm_config.json` (boilerplate config file to guide us - great)

From here: *\"After the machine is booted, you can still use the socket to send API requests for post-boot operations.\"*

### [](#conclusion){#conclusion} Conclusion

Somewhat of a pain in the ass (just looking through the directions); the fact that we\'d have to go grab a uncompressed kernel image + file system image (\`) is kind of a fucking hassle / burden.

Was hoping for a solution more akin to Docker where it can just be spun up real quick & then deployed. But they claim that this \'jailer\' feature (that they keep hyping) will **ensure** (I guess?) that whatever is done within the container will remain within the container (and not escape).

I haven\'t seen anything that sticks out about this project that leads me to believe that it possesses that capability, but I definitely don\'t want to rule it out.

### [](#extra-documentation-information){#extra-documentation-information} Extra Documentation + Information

1.  **OSv Running on \'Firecracker\'** (yay more work though) - [http://blog.osv.io/blog/2019/04/19/making-OSv-run-on-firecraker/](http://blog.osv.io/blog/2019/04/19/making-OSv-run-on-firecraker/){rel="noopener noreferrer"}

2.  **Building OSv Images Using Docker** - [http://blog.osv.io/blog/2015/04/27/docker/](http://blog.osv.io/blog/2015/04/27/docker/){rel="noopener noreferrer"}

3.  **firecracker containerd** (this is something that\'s probably important for the overall mission of what we want to accomplish here) - [https://github.com/firecracker-microvm/firecracker-containerd](https://github.com/firecracker-microvm/firecracker-containerd){rel="noopener noreferrer"}

### [](#firecracker-containerd){#firecracker-containerd} Firecracker Containerd

**Description** - \"`firecracker-containerd` enables `containerd` to manage containers as Firecracker microVMs\*\"

-   \"This repository enables the use of a container runtime, `containerd`, to manage Firecracker microVMs. Like traditional containers, Firecracker microVMs offer fast start-up and shut-down and minimal overhead. Unlike traditional containers, however, they can provide an additional layer of isolation via the KVM hypervisor.\"

**They Also Identify Potential Use-Cases in the Repo Such as**

1.  \"*Sandbox a partially or fully untrusted third party container in its own microVM. This would reduce the likelihood of leaking secrets via the third party container, for example.*\"

2.  \"*Bin-pack disparate container workloads on the same host, while maintaining a high level of isolation between containers. Because the overhead of Firecracker is low, the achievable container density per host should be comparable to running containers using kernel-based container runtimes, without the isolation compromise of such solutions. Multi-tenant hosts would particularly benefit from this use case.*\"

Really interesting feature of this repo here is: \"*A root file filesystem image builder that constructs a firecracker microVM root filesystem containing `runc` and the `firecracker-containerd` agent.*\" (that could save a lot of time on that whole filesystem image thing that they were mentioning prior)

**Additional Links of Importance**

1.  **Getting Started Guide** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/getting-started.md](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/getting-started.md){rel="noopener noreferrer"}

2.  **Quickstart Guide** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/quickstart.md](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/quickstart.md){rel="noopener noreferrer"}

3.  **A Root Filesystem Image Builder** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/tools/image-builder](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/tools/image-builder){rel="noopener noreferrer"}

4.  **Runtime Linking Containerd** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/runtime](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/runtime){rel="noopener noreferrer"}

**Documentation All Located Here** - [https://github.com/firecracker-microvm/firecracker-containerd/tree/main/docs](https://github.com/firecracker-microvm/firecracker-containerd/tree/main/docs){rel="noopener noreferrer"} (definitely fucking needed because there\'s a lot here to wrap one\'s head around)

-   **Design Approaches Doc** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/design-approaches.md](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/design-approaches.md){rel="noopener noreferrer"}

-   **Shim Architecture** - [https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/shim-design.md](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/shim-design.md){rel="noopener noreferrer"}

-   **Launching 4k VMs Using Firecracker** - [https://github.com/firecracker-microvm/firecracker-demo](https://github.com/firecracker-microvm/firecracker-demo){rel="noopener noreferrer"}

-   `firectl` (CLI options for manipulating this tool from terminal ; this is important as well) - [https://github.com/firecracker-microvm/firectl](https://github.com/firecracker-microvm/firectl){rel="noopener noreferrer"} \[damn, there\'s a lot that came with this here!\]
