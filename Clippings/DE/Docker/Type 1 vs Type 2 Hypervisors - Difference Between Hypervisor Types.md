---
title: "Type 1 vs Type 2 Hypervisors - Difference Between Hypervisor Types"
source: "https://aws.amazon.com/compare/the-difference-between-type-1-and-type-2-hypervisors/"
author:
  - "[[Amazon Web Services]]"
published:
created: 2026-04-10
description: "What's the Difference Between Type 1 and Type 2 Hypervisors? How to Use Type 1 and Type 2 Hypervisors with AWS"
tags:
  - "clippings"
---
## Page topics

- [What’s the difference between Type 1 and Type 2 Hypervisors?](#whats-the-difference-between-type-1-and-type-2-hypervisors--axi6re)
- [Why are type 1 and type 2 hypervisors important?](#why-are-type-1-and-type-2-hypervisors-important--axi6re)
- [How they work: type 1 vs. type 2 hypervisors](#how-they-work-type-1-vs-type-2-hypervisors--axi6re)
- [Key differences: type 1 vs. type 2 hypervisors](#key-differences-type-1-vs-type-2-hypervisors--axi6re)
- [When to use: type 1 vs. type 2 hypervisors](#when-to-use-type-1-vs-type-2-hypervisors--axi6re)
- [Summary of differences: type 1 vs. type 2 hypervisors](#summary-of-differences-type-1-vs-type-2-hypervisors--axi6re)
- [How can AWS help with your hypervisor requirements?](#how-can-aws-help-with-your-hypervisor-requirements--axi6re)

## What’s the difference between Type 1 and Type 2 Hypervisors?

Type 1 and type 2 hypervisors are software you use to run one or more virtual machines (VMs) on a single physical machine. A virtual machine is a digital replica of a physical machine. It’s an isolated computing environment that your users experience as completely independent of the underlying hardware. The hypervisor is the technology that makes this possible. It manages and allocates physical resources to VMs and communicates with the underlying hardware in the background.

The type 1 hypervisor sits on top of the bare metal server and has direct access to the hardware resources. Because of this, the type 1 hypervisor is also known as a *bare metal hypervisor*. In contrast, the type 2 hypervisor is an application installed on the host operating system. It’s also known as a *hosted* or *embedded hypervisor*.

[Read about hypervisors »](https://aws.amazon.com/what-is/hypervisor/)

## Why are type 1 and type 2 hypervisors important?

A hypervisor, sometimes called a *virtual machine monitor (VMM)*, creates and coordinates virtual machines (VMs), an essential technology in modern computing infrastructure. A hypervisor is what makes virtualization of computers and servers possible.

Virtualization is technology that you use to create virtual representations of hardware components like server or network resources. The software representation uses the underlying physical resource to operate as if it were a physical component. Similarly, a VM is a software-based instance of a computer, with elements like memory, processing power, storage, and an operating system.

VMs are preferable to using real machines thanks to their portability, scalability, cost, resource optimization, and reconfigurability. A VM requires a hypervisor to run.

[Read about virtualization »](https://aws.amazon.com/what-is/virtualization/)

## How they work: type 1 vs. type 2 hypervisors

The hypervisor is the coordination layer in virtualization technology. It supports multiple virtual machines (VMs) running at once.

![](https://docs.aws.amazon.com/images/whitepapers/latest/security-design-of-aws-nitro-system/images/virtualization-architecture.png)

### Type 1 hypervisor

A type 1 hypervisor, or a bare metal hypervisor, interacts directly with the underlying machine hardware. A bare metal hypervisor is installed directly on the host machine’s physical hardware, not through an operating system. In some cases, a type 1 hypervisor is embedded in the machine’s firmware.

The type 1 hypervisor negotiates directly with server hardware to allocate dedicated resources to VMs. It can also flexibly share resources, depending on various VM requests.

### Type 2 hypervisor

A type 2 hypervisor, or hosted hypervisor, interacts with the underlying host machine hardware through the host machine’s operating system. You install it on the machine, where it runs as an application.

The type 2 hypervisor negotiates with the operating system to obtain underlying system resources. However, the host operating system prioritizes its own functions and applications over the virtual workloads.

## Key differences: type 1 vs. type 2 hypervisors

While type 1 and type 2 hypervisors share the common goal to run and coordinate virtual machines (VMs), they have some significant variations.

### Resource allocation

Type 1 hypervisors directly access underlying machine resources. They can implement their own custom resource allocation strategies to service their VMs.

Type 2 hypervisors negotiate resource allocation with the operating system, which makes the process slower and less efficient.

### Ease of management

Managing a type 1 hypervisor and its VM configuration requires system administrator-level knowledge, as it’s relatively complex.

In contrast, you can install and manage type 2 hypervisors as an application on an operating system. Even nontechnical users can operate them.

### Performance

Type 1 hypervisors offer greater performance to their VMs. This is because they don’t need to negotiate resources with the operating system or travel through the operating system layer. The type 1 hypervisor offers dedicated underlying resources without any negotiation required.

Type 2 hypervisors must only use the resources that the operating system is willing to provide.

### Isolation

Type 1 hypervisors offer a greater degree of isolation for each virtual environment. There’s no shared layer like there is with the operating system for a type 2 hypervisor. This makes virtual machines running on the type 1 hypervisor inherently more secure. However, updating and patching your virtual machine operating systems is a critical security activity.

## When to use: type 1 vs. type 2 hypervisors

Type 1 hypervisors are typically used in data centers, enterprise computing workload situations, web servers, and other primarily fixed-use applications. Cloud computing environments run bare metal hypervisors to offer the most performant virtual machines (VMs) for the underlying physical hardware. Cloud providers also abstract away type 1 hypervisor management and offer VMs as cloud instances you can access through APIs.

Type 2 hypervisors are most often used in desktop and development environments, where workloads are not as resource-intensive or critical to operations. They’re also preferred in cases where users want to simultaneously use two or more operating systems but only have access to one machine.

[Read about cloud instances »](https://aws.amazon.com/what-is/cloud-instances/)

## Summary of differences: type 1 vs. type 2 hypervisors

|  | **Type 1 hypervisor** | **Type 2 hypervisor** |
| --- | --- | --- |
| Also known as | Bare metal hypervisor. | Hosted hypervisor. |
| Runs on | Underlying physical host machine hardware. | Underlying operating system (host OS). |
| Best suited for | Large, resource-intensive, or fixed-use workloads. | Desktop and development environments. |
| Can it negotiate dedicated resources? | Yes. | No. |
| Knowledge required | System administrator-level knowledge. | Basic user knowledge. |
| Examples | VMware ESXi, Microsoft Hyper-V, KVM. | Oracle VM VirtualBox, VMware Workstation, Microsoft Virtual PC. |

## How can AWS help with your hypervisor requirements?

Amazon Web Services (AWS) offers virtualization solutions across an extensive range of infrastructure, including networking, compute, storage, and databases. The cloud is built on virtualization, and we continually optimize, simplify, and diversify our services to suit the needs of all users and organizations.

[AWS Nitro System](https://aws.amazon.com/ec2/nitro/) is a lightweight hypervisor that allows organizations to innovate faster in a secure cloud environment. Traditionally, hypervisors protect the physical hardware and bios and virtualize the CPU, storage, and networking. They also provide a rich set of management capabilities. With the Nitro System, we can break apart those functions. We can offload them to dedicated hardware and software and reduce costs by delivering practically all of the resources of a server to your instances.

With the Nitro System, you benefit from these capabilities:

- Continuously monitor your virtualized resources to prevent unauthorized access
- Achieve enhanced performance with dedicated Nitro Cards, including high-speed networking, high-speed block storage, and I/O acceleration
- Create isolated compute environments to protect personally identifiable information (PII), financial data, and other sensitive information

The Nitro System is the underlying platform for our next generation of cloud instances. You can use [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) to choose from over 600 instances with different processor, storage, networking, operating system, and purchase model configurations. You can also use cloud instances for all types of complex use cases at scale, without worrying about hypervisors.

Get started with hypervisors and instances on AWS by [creating an account](https://portal.aws.amazon.com/billing/signup) today.

## Browse all cloud computing concepts

Browse all cloud computing concepts content here:

Displaying 1-8 (293)[2022-08-08](https://aws.amazon.com/what-is/iaas/?trk=faq_card)

#### What is IaaS (Infrastructure as a Service)?

Infrastructure as a Service (IaaS) is a business model that delivers IT infrastructure like compute, storage, and network resources on a pay-as-you-go basis over the internet. You can use IaaS to request and configure the resources you require to run your applications and IT systems. You are responsible for deploying, maintaining, and supporting your applications, and the IaaS provider is responsible for maintaining the physical infrastructure. Infrastructure as a Service gives you flexibility and control over your IT resources in a cost-effective manner.

Learn more

[View original](https://aws.amazon.com/what-is/iaas/?trk=faq_card)[2022-05-25](https://aws.amazon.com/what-is/machine-translation/?trk=faq_card)

#### What is Machine Translation?

Machine translation is the process of using artificial intelligence to automatically translate text from one language to another without human involvement. Modern machine translation goes beyond simple word-to-word translation to communicate the full meaning of the original language text in the target language. It analyzes all text elements and recognizes how the words influence one another.

Learn more

[View original](https://aws.amazon.com/what-is/machine-translation/?trk=faq_card)[2022-08-12](https://aws.amazon.com/what-is/block-storage/?trk=faq_card)

#### What is Block Storage?

Block storage is technology that controls data storage and storage devices. It takes any data, like a file or database entry, and divides it into blocks of equal sizes. The block storage system then stores the data block on underlying physical storage in a manner that is optimized for fast access and retrieval. Developers prefer block storage for applications that require efficient, fast, and reliable data access. Think of block storage as a more direct pipeline to the data as opposed to file storage which has an extra layer consisting of a file system (NFS, SMB) to process before accessing the data.

Learn more

[View original](https://aws.amazon.com/what-is/block-storage/?trk=faq_card)[2021-09-29](https://aws.amazon.com/relational-database/?trk=faq_card)

#### What is a Relational Database?

A relational database is a collection of data items with pre-defined relationships between them. These items are organized as a set of tables with columns and rows. Tables are used to hold information about the objects to be represented in the database. Each column in a table holds a certain kind of data and a field stores the actual value of an attribute. The rows in the table represent a collection of related values of one object or entity. Each row in a table could be marked with a unique identifier called a primary key, and rows among multiple tables can be made related using foreign keys. This data can be accessed in many different ways without reorganizing the database tables themselves.

Learn more

[View original](https://aws.amazon.com/relational-database/?trk=faq_card)[2023-10-02](https://aws.amazon.com/what-is/advanced-analytics/?trk=faq_card)

#### What is Advanced Analytics?

Advanced analytics is the process of using complex machine learning (ML) and visualization techniques to derive data insights beyond traditional business intelligence. Modern organizations collect vast volumes of data and analyze it to discover hidden patterns and trends. They use the information to improve business process efficiency and customer satisfaction. With advanced analytics, you can take this one step further and use data for future and real-time decision-making. Advanced analytics techniques also derive meaning from unstructured data like social media comments or images. They can help your organization solve complex problems more efficiently. Advancements in cloud computing and data storage have made advanced analytics more affordable and accessible to all organizations.

Learn more

[View original](https://aws.amazon.com/what-is/advanced-analytics/?trk=faq_card)[2023-10-04](https://aws.amazon.com/what-is/gpu/?trk=faq_card)

#### What is a GPU?

A graphics processing unit (GPU) is an electronic circuit that can perform mathematical calculations at high speed. Computing tasks like graphics rendering, machine learning (ML), and video editing require the application of similar mathematical operations on a large dataset. A GPU’s design allows it to perform the same operation on multiple data values in parallel. This increases its processing efficiency for many compute-intensive tasks.

Learn more

[View original](https://aws.amazon.com/what-is/gpu/?trk=faq_card)[2024-08-01](https://aws.amazon.com/what-is/enterprise-ai/?trk=faq_card)

#### What Is Enterprise AI?

Enterprise artificial intelligence (AI) is the adoption of advanced AI technologies within large organizations. Taking AI systems from prototype to production introduces several challenges around scale, performance, data governance, ethics, and regulatory compliance. Enterprise AI includes policies, strategies, infrastructure, and technologies for widespread AI use within a large organization. Even though it requires significant investment and effort, enterprise AI is important for large organizations as AI systems become more mainstream.

Learn more

[View original](https://aws.amazon.com/what-is/enterprise-ai/?trk=faq_card)[2022-05-25](https://aws.amazon.com/what-is/web-hosting/?trk=faq_card)

#### What is Web Hosting?

Web hosting is a service that stores your website or web application and makes it easily accessible across different devices such as desktop, mobile, and tablets. Any web application or website is typically made of many files, such as images, videos, text, and code, that you need to store on special computers called servers. The web hosting service provider maintains, configures, and runs physical servers that you can rent for your files. Website and web application hosting services also provide additional support, such as security, website backup, and website performance, which free up your time so that you can focus on the core functions of your website.

Learn more

[View original](https://aws.amazon.com/what-is/web-hosting/?trk=faq_card)

## Did you find what you were looking for today?

Let us know so we can improve the quality of the content on our pages