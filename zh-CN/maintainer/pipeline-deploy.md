# Pipeline Deployment

## 1 Introduction

This document is intended as a reference for project maintainers or users who need to perform secondary development (i.e., deploy a complete Harmonybrew project on their own). Regular users do not need to read this.

This project’s CI system is built on Huawei Cloud infrastructure; therefore, all resource operations and configurations described in this document are based on Huawei Cloud. Developers with strong hands-on skills can refer to the underlying logic to migrate the system to other cloud providers on their own.

The following section explains the pipeline deployment process using a custom development scenario as an example.

## 2 Prerequisites

### 2.1 ECS

You will need to prepare several servers, each with distinct roles.

Here, we assume we have prepared 6 servers, each with a unique name and assigned role

| Hostname         | Role                                                 |
| -------------- | ---------------------------------------------- ------ |
| image-builder  | Dedicated to running `auto_images.sh`, responsible for building the `ci-runner` image |
| webhook-filter | Dedicated to running `webhook_filter.py`, responsible for filtering webhook messages  |
| ci-runner-1    | All other pipeline scripts run on these machines                       |
| ci-runner-2    | All other pipeline scripts run on these machines                       |
| ci-runner-3    | All other pipeline scripts run on these machines                       |
| ci-runner-4    | All other pipeline scripts run on these machines                       |

These servers must meet the following requirements:

1. They must be ARM servers, not x86 servers
2. The region must be in Hong Kong or overseas (during the CI execution process, source code and development tools need to be downloaded from GitHub, and servers in mainland China cannot reliably access GitHub)
3. JDK must be installed, version 8 or higher
4. Docker must be installed

### 2.2 Domain Name

Prepare your own domain name, and it is recommended to complete ICP filing. This domain will later be used as the CDN acceleration domain.

Since Huawei Cloud no longer sells domain names, you will need to purchase one from another cloud provider or a specialized domain registrar.

Once you have your domain name, you will also need to obtain an SSL certificate for it. If you are an individual developer looking to save costs, we recommend using the free certificates provided by [Let's Encrypt](https://letsencrypt.org/).

### 2.3 OBS and CDN

Go to the OBS console and create an OBS bucket to store your software packages; the name and region can be chosen freely.

Go to the CDN console, configure your domain as the CDN acceleration domain, and set the CDN origin URL to point to your OBS bucket.

Selecting the CDN service coverage:

- If you have completed ICP filing, you can select “Mainland China” or “Global.”
- If you have not completed ICP filing, you can only select “Outside Mainland China.”

After configuring the CDN accelerated domain, deploy your SSL certificate to the CDN as well to enable HTTPS links later.

### 2.4 SWR

You need to create an organization in the SWR console. The Docker images built by the pipeline will be uploaded to this organization.

### 2.5 Code Repository

1. Create a new organization on AtomGit; the name and region can be arbitrary.
2. Move all repositories under the Harmonybrew organization to
