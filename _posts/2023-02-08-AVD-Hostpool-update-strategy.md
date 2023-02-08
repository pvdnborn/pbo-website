---
layout: post
title:  "Azure Virtual Desktop Host pool update strategy via a Blue-Green deployment."
date:   2023-02-08 17:00:00 +0100
categories: Terraform, Azure Virtual Desktop, AVD, Release strategy, blue, green, blue-green
---
Recently, I have been working on Infrastructure as Code and Image as Code projects. During these projects, it has become clear to me that having a good release strategy can have many benefits. I have some practical experience with this and would like to share my thoughts of an image release approach in the cloud with you. 

## On-premises versus Cloud Single Image Management
Over the last few years, many non-persistent single image management methods exist within End User Computing, such as Citrix Provisioning Services, Citrix Machine Creation Services, VMware linked clones, VMware instant clones, etc. These methods are handy for managing the underlying disk image of VDI's or Session Hosts from a central point. Single image management is a topic that has been active within EUC for years, and everyone knows the benefits of this approach.

Looking at how traditional single-image management methods release their images, the existing resources (virtual or physical machines) are provisioned with a new disk image. Or, in the case of a rollback, the existing resources are provisioned with the previous disk image.

With the advent of the public cloud, we see that existing single image methods, as we know them from on-premises, are being developed further for the cloud. There is nothing wrong with this because these techniques are often already known within the organization. On the other hand, the non-persistent character can be a definite plus in some cases.

With the arrival of the public cloud, in addition to the traditional single image management methods, new possibilities also arise for releasing disk images to virtual machines in the cloud. New opportunities arise from the elasticity of the cloud, creating new possibilities.

Often, I hear the complaint that no single image management method is available for Azure Virtual Desktop. The Azure platform offers excellent opportunities to apply a good release strategy. The following paragraphs explain how I use a good image release strategy for Azure Virtual Desktop host pools.

## Why a good image release strategy is necessary
To stay in control after releasing new images, a good image release strategy is necessary. Managing pets, where all session hosts are updated individually, is:
• Prone to errors; session hosts can deviate due to inconsistency or human error
• Downtime; a maintenance window must be agreed upfront to perform the updates.
• Recovery time; recovery from a faulty update can take a long time.

To tackle these problems, I often use a blue-green deployment strategy. This strategy comes from software development world, where the production environment is referred to as (blue) and the standby environment as (green). The updates are then installed and tested on the green environment. Once the updates are found to be good, the switch is made from the blue environment to the green environment so that the green environment becomes the new production environment. This method can also be applied to Azure Virtual Desktop Host pools, Citrix Machine Catalogs, etc.

## Green and blue host pools
A blue-green deployment can be applied easily within an Azure Virtual Desktop environment as follows:

![InitialBlueGreenAVDSetup]({{site.baseuirl}}/assets/img/Posts/2023-02-08-bluegreenstrategy/01-InitialBlueGreenAVDSetup.jpg)

A blue-green AVD environment can be created by provisioning two host pools. The first host pool, blue, is the production environment, while the second host pool, green, is the standby environment. When updating an image, the new image is applied to the green (standby) host pool. If the image is good, the switch is made from the blue environment to the green environment so that the green environment becomes the new production environment. The blue host pool is updated with the new image at the next image release, ready for the next release cycle. This image release process can be fully automated by using Azure DevOps, for example.

With Azure Virtual Desktop, it is easy to define which environment is the production environment. In the example above, the blue environment is the production environment since the blue application group is associated with the workspace. The association between the green application group and the workspace is not available. In this example, end-users will only see blue resources in the Remote Desktop Client, making the blue environment the production environment. The green environment is the standby environment and is not associated with the workspace. The VMs of the standby environment can be shut down to save costs.

### Updating the Image in the Green Environment
A new image for the green (standby) environment can be released as follows:

![UpdateBlueAVDHostpool]({{site.baseuirl}}/assets/img/Posts/2023-02-08-bluegreenstrategy/02-UpdateBlueAVDHostpool.jpg)

Process:
    1. Place the updated image as a new image version in the Azure Compute Gallery
    2. Reprovision all machines in the green host pool with the most recent image from the Azure Compute Gallery
    3. Create an association between the green application group and the workspace
    4. Break the association between the blue application group and the workspace

This way, the green environment becomes the production environment, and the blue environment becomes the standby environment. In case of issues arising from updates in the green environment, it can easily be rolled back to the blue environment by establishing an association between the blue application group and the workspace and breaking it between the green application group and workspace.

## Updating the Image in the Blue Environment
The process for updating the following image is the same, only now we update the blue (standby) environment as follows:

![UpdateBlueAVDHostpool]({{site.baseuirl}}/assets/img/Posts/2023-02-08-bluegreenstrategy/02-UpdateBlueAVDHostpool.jpg)

Process:
    1. Place the updated image as a new image version in the Azure Compute Gallery
    2. Reprovision all machines in the blue host pool with the most recent image from the Azure Compute Gallery
    3. Create an association between the blue application group and the workspace
    4. Break the association between the green application group and the workspace

This cycle can be repeated indefinitely by alternating blue and green as the production environment.

## Quick Release of Security Patches vs. Extensive Image Testing
In recent years, it has become very common to test images before releasing them to production to ensure that all applications are still functional. Although testing can be done in many ways, such as through automatic regression testing, I still see testing often performed manually. There is nothing wrong with testing, but in some cases, it delays the release of a new image.

Due to the rise of cybercrime, such as hacking and ransomware, it is crucial to implement security patches as soon as possible. Patching includes operating systems, web browsers, and base applications (PDF, 7zip, VSCode, etc.) security patches. In an Azure Virtual Desktop, end-users may click on links that they shouldn't, making these systems extra vulnerable to exploitation of known vulnerabilities.

The rollback time to the previous image is minimal using a blue-green deployment strategy. The association between the application groups (blue/green) and the workspace must be reversed. Because the rollback process is so simple, I always advise my clients to immediately implement security patches without extensive testing. The downtime and impact on the organization for rolling back to the standby environment are lower than hackers taking advantage of exploits on known vulnerabilities.

On the other hand, some upgrades (e.g., from AutoCAD 2021 to AutoCAD 2023) must be extensively tested in a testing environment. Therefore, I always follow two release strategies:
• Security updates: implement as quickly as possible, relying on rapid rollback
• Major application updates: extensively test updates in a test setup

## Automation and Terraform Workspaces
As you may already know, I am a huge fan of automation and try to do as little manual work as possible. To release images for Azure Virtual Desktop using a blue-green release method, I use [Terraform Workspaces](https://developer.hashicorp.com/terraform/cli/workspaces). With Terraform workspaces, the same Azure Virtual Desktop desired state configuration can easily be applied independently to the blue or green environment. I will write about this in more detail in my next blog.
