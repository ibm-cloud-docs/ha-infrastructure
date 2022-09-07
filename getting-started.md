---

copyright:
  years: 2018, 2021
lastupdated: "2021-03-30"

keywords: cloud environment, virtual server, virtual machine, vm, understanding infrastructure, IaaS model, IT ops admin, on-premises, data center

subcollection: ha-infrastructure


---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:script: data-hd-video='script'}
{:video: .video}

# Getting started with High Availability and Resiliency for {{site.data.keyword.cloud_notm}} 
{: #getting-started-tutorial}

Use {{site.data.keyword.cvad_full}} to create a dedicated {{site.data.keyword.cvad_short}} service environment at any of our global data centers. You have the option of creating a Classic solution, VMWare solution, or VPC solution. 
{: shortdesc}

## Before you begin
{: #before-you-begin-getting-started}

After you review this information, you are ready to get started. To begin, you need to complete the following prerequisites.

1. Sign up for an {{site.data.keyword.cloud_notm}} account
2. Create required API keys
3. Create required SSH keys

## Step 1. Sign up for an {{site.data.keyword.cloud_notm}} account
{: #sign-up-IBM-Cloud-account}

To deploy highly available and resilient resources, you need to sign up for an {{site.data.keyword.cloud_notm}} account. Billing information is associated with your {{site.data.keyword.cloud_notm}} account and the cost of the physical and virtual infrastructure and the resulting licenses are charged to your account. For more information about signing up, see [Setting up your {{site.data.keyword.cloud_notm}} account](/docs/account?topic=account-account-getting-started#signup).

## Step 2. Create {{site.data.keyword.cloud_notm}} API keys
{: #create-api-keys}

For Classic deployments, you need to add a classic infrastructure API key to your {{site.data.keyword.cloud_notm}} account. For more information about creating or managing API keys, see [Managing classic infrastructure API keys](/docs/account?topic=account-classic_keys).

For VPC solutions, you need to add an {{site.data.keyword.cloud_notm}} API key to your {{site.data.keyword.cloud_notm}} account. For more information about creating or managing API keys, see [Managing user API keys](/docs/account?topic=account-userapikey&interface=ui).

## Step 3. Create {{site.data.keyword.cloud_notm}} SSH keys
{: #create-ssh-keys}

Create or retrieve your [SSH key name](/docs/ssh-keys?topic=ssh-keys-getting-started-tutorial). The SSH key is used to SSH to the bastion server.

## Next steps
{: #nextsteps1}

To continue, see [High Availability and Resiliency on {{site.data.keyword.cloud_notm}}](/docs/ha-infrastructure?topic=ha-infrastructure-landing-about-ha-dr-backup).

