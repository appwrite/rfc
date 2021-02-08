# Deployment to Digitalocean <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 08-02-2021
- Target Date: N/A
- Appwrite Issue: N/A

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

In an effort ease deployment, we want to support Digitalocean Marketplace one click deployment. After this, anyone who wants to deploy Appwrite in Digitalocean can quickly provision a virtual server in their account with Appwrite 1-click installation from marketplace and proceed with Appwrite setup which will automatically run once they log in to the server.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->
Right now, if you want to deploy Appwrite to Digitalocean platform, you need to provision a server and install things like docker, docker-compose or you can provision a server with docker pre installed. And then manually copy the setup script from Appwrite and run it to setup and run Appwrite. With this, users don't have to do any manual steps.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

We are trying to make deployment of Appwrite as easy as possible even for a novice user.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

The system will not change in any way. This whole process is based on Digitalocean marketplace, where we create an image with Appwrite setup scripts. Which once approved will be available in the Digitalocean marketplace.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

Implementing this requires a separate repository, where we can store scripts and other information required to create an image for digitalocean marketplace.

Making image for DO marketplace, they have support for two automation tools, (Fabric and Packer). Among them, we will be using Packer, as it's the one, that has least user interaction. Everything is automated.

## Create new project
Create a new project based off of template from https://github.com/digitalocean/marketplace-partners. 

## appwrite.json
This will be the main file that packer will use to build the Appwrite image. Base image used will be Ubuntu 18.04 as DO marketplace doesn't yet support Ubuntu 20.04.

```json
{
  "variables": {
    "token": "{{env `DIGITALOCEAN_TOKEN`}}",
    "image_name": "appwrite-snapshot-{{timestamp}}"
  },
  "builders": [
    {
      "type": "digitalocean",
      "api_token": "{{user `token`}}",
      "image": "ubuntu-18-04-x64",
      "region": "nyc3",
      "size": "s-1vcpu-2gb",
      "ssh_username": "root",
      "snapshot_name": "{{user `image_name`}}"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "cloud-init status --wait"
      ]
    },
    {
      "type": "file",
      "source": "files/etc/",
      "destination": "/etc/"
    },
    {
      "type": "file",
      "source": "files/opt/",
      "destination": "/opt/"
    },
    {
      "type": "shell",
      "inline": [
        "apt -qqy update",
        "apt -qqy -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' full-upgrade"
      ]
    },
    {
      "type": "shell",
      "scripts": [
        "scripts/01-setup-appwrite-scripts.sh",
        "scripts/02-docker-install.sh",
        "scripts/90-cleanup.sh",
        "scripts/99-image_check.sh"
      ]
    }
  ]
}
```


## Create Install scripts
1. Create appwrite install scripts inside files/opt/appwrite/install.sh in the final image that contains following code.

```shell
#!/bin/bash

docker run -it --rm \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --volume "$(pwd)"/appwrite:/install/appwrite:rw \
    -e version=0.6.2 \
    appwrite/install

cp -f /etc/skel/.bashrc /root/.bashrc
```

While building this file will be copied to /opt/appwrite/install.sh

## Create build time script
Create following scripts in scripts folder, which will only be used during the build time by packer.

1. Script to setup installation scripts

This script will do trivial things like, make appwrite setup script executable, and add line to /root/.bashrc, that will ensure that the installation script is executed when the user logs in.

```shell
#!/bin/bash

chmod +x /opt/appwrite/install.sh /etc/update-motd.d/99-appwrite-readme

## add install.sh to .bashrc
echo "/opt/appwrite/install.sh" >> /root/.bashrc
```

2. Script to setup docker and docker-compose

This script will install docker and docker-compose which is required to run Appwrite.

```shell
#!/bin/bash

curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

3. Cleanup and Image check scripts
These scripts are provided by DO for marketplace partners. These will cleanup any unnecessary data and validate that it's a valid image for marketplace.
  - https://github.com/digitalocean/marketplace-partners/blob/master/scripts/cleanup.sh
  - https://github.com/digitalocean/marketplace-partners/blob/master/scripts/img_check.sh


## Build snapshot
Finally, we can run
`packer build appwrite.json` to build the snapshot required. Which can be submitted to DO marketplace vendor portal.

You need to set the environment variable `DIGITALOCEAN_TOKEN` in the shell before running the build command.

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is reasonably clear how the contribution would be implemented
- Dependencies on libraries, tools, projects or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables
- Changes to documentation
- Ways to scale the solution

Ensure that you include examples, code-snippets etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal. A
few examples of what this can include are:

- Does this functionality exist in other software and what experience has their
  community had?
- For other teams: What lessons can we learn from what other communities have
  done here?
- Papers: Are there any published papers or great posts that discuss this? If
  you have some relevant papers to refer to, this can serve as a more detailed
  theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other software.

Write your answer below.
-->

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

With these scripts setup, every new release version can have a DO marketplace app within minutes.