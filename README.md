DNS as a Service
================

DNSaaS refers to the ability to update DNS records on demand though a web API. The standard implementation to programmable DNS services today is an authenticated JSON API with role based access control. Two popular open source offerings with such API built in are [PowerDNS](https://www.powerdns.com/) and [Atomia DNS](http://atomiadns.com/). For cloud providers running OpenStack, the official DNSaaS offering for OpenStack is [Designate](https://docs.openstack.org/designate/latest/), which provides this interface by default and interfaces with many popular DNS servers, such as BIND9 that do not, on their own, provide API access.

Advantages of DNSaaS
--------------------

DNSaaS opens up a wealth of functionality for end users, at the very minimum, enabling them to achieve a quality of service that exceed that of the underlying infrastucture on which they run. It also enables users to leverage best practices promoted in modern application architectures, which lead to advantages for both the user and the cloud provider. A few of these advantages relevant to academic cloud providers are:

-	The rise in popularity of containerized applications creates a need for first-class service discovery. DNS is a common service discovery solution for small to medium sized applications, which most academic users are most likely to operate.
-	Enables the utilization of advanced service-to-sevice based trust relationships to be established in highly dynamic environments.
-	Enables proper configuration of mail clients which are increasingly blocked by major service providers when attempting to use the default Postfix/Sendmail toolchain.
-	Allows meaningful hostnames to be create which make it possible for new applications to achieve better word of mouth advertising to grow their user base.
-	The ability to generate valid self-service SSL certs from Let's Encrypt. (More on this in a bit)

The Need for DNSaaS or, SSL and the World of Tomorrow
-----------------------------------------------------

Within the context of an Academic cloud environment, users need the ability to enable secure communications between their applications, infrastructure, and third-party services. Some of the driving factors behind this are:

-	Science is increasingly being pushed to the web and browsers are becoming increasingly hostile to non-SSL traffic.
-	The future of the web is HTTP/2 and HTTP/2 is already upon us. As the majority of the web moves over to use this new protocol, all traffic will have to be encrypted via SSL.
-	Research applications, science gateways, and scripted "glue" codes often send and receive sensitive information that could easily be read without encrypted communication.
-	Science is becoming increasingly distributed. Academic clouds are multi-tenant shared hosting environments. As more science moves to the cloud, more research will be done on shared hosting. Without valid host validation and SSL certs, information can be leaked within a host, thus compromising the integrity of the research being performed.
-	The use of SSL reduces commonly known attacks.

While the use of self-signed certificates has long been the go-to for researchers and developers in non-production environments, this is becoming less attractive a solution for several reasons:

-	The use of self-signed certs is often a non-starter when interacting with modern browsers in managed environments such as universities, government labs, and enterprise organizations. Browsers simply will not load content from untrusted sites and users are unable to bypass their browser security.
-	Applications are leveraging more an more third-party content, either from formal web services, or external data sources. The use of self-signed SSL certificates makes the consumption of such content a greater risk. Thus both content providers and consumers suffer from untrusted certificate usage.
-	Continually generating an managing new self-signed certificates requires continually issuing new certificate exceptions by everyone consuming the services. Thus, in highly dynamic environments, the common solution becomes disabling host and certificate validation rather than adding a self-signed CA to the list of trusted CAs maintained by the browser, application, service, or server.

Contrasting Approaches to SSL Deployment
----------------------------------------

In reality, the problem of obtaining and deploying a valid SSL cert is a solved problem. If we are completely honest, it has been for a couple decades now. What we're talking about isn't enabling the possibility of pervasive SSL in academic clouds, it is enabling the capability of a user experience sufficiently robust, integrated, and approachable that there is little to no way to justify not using it. In the commercial space, this is largely the case.

### Commercial Clouds

In the commercial space SSL certificate generation is essentially turnkey solution with dozens of working keys. Services for automated deployment of existing certs, managed hosting solutions with SSL management built in, VM with valid certs injected at startup, REST API to dynamically generated SSL certificates, etc. The list goes on and on. None of this would be possible, however, without either the user or the cloud provider to dynamically manage DNS in such a way that they could ensure that the certificate matched the host for which it was created. Thus, programmable DNS goes hand in hand with broad use of trusted SSL certificates in dynamic cloud environments.

> Please note that we are not saying that commercial cloud providers are the ones who do or should provide either programmable DNS and SSL certs. We are saying that, in the commercial space, where the use of SSL is a fundamental requirement to doing business online, the two services are not mutually exclusive. Pervasive use of SSL comes with the ability to programmatically manage DNS and keep certificates up to date.

### Academic Clouds

In contrast to the commercial space, obtaining and using valid SSL certs in the academic space has, historically, been more of a challenge. The process would resemble the following:

1.	Spin up a VM, obtain the appropriate firewall exceptions, and verify its operation.
2.	Obtained a domain name. Usually one of the project leads would do this at the start of a project. They might buy it outright, put in an order though their purchasing department, request a subdomain off their department/university domain, etc.
3.	Wait for the domain to be issued.  
4.	Request a SSL certificate for the domain and any necessary subdomains. Again, this might be accomplished by purchasing them for a fee, requesting from their university or parent organization, or actually managing their own static CA and establishing the necessary trust with all consuming parties.
5.	Wait for a response from the issuing CA organization.
6.	Optionally prove that they actually own the host and ip. The may involve making a change to the whois record, installing a web server and challenge file on the VM, or adding another DNS entry...in which case, goto #2.

For all the human-in-the-loop interaction, this approach has the advantages of allowing a great degree of flexibility over the source, duration, and coverage of the certificates obtained. However, that flexibility comes at a cost. That cost is paid in:

-	**Time lost:** waiting for a human to issue the certs,
-	**Effort spent:** installing, monitoring, renewing, and rotating certificates every 1-2 years, and
-	**Ongoing money spent:** by the project lead or their parent organization to lease a domain name and purchase limited SSL certs.

In the mean time, while the various forms, purchase orders, emails, and signatures were processed, the developers went ahead with their work using, at best, self-signed certs, but more realistically, no certs at all. In academic research environments, the kind of information one would want to protect in a development environment is not significantly different than a production environment. They are accessing data, systems, and accounts in development just like in production. In fact, it is likely that those infrastructure components are common regardless of their deployment environment.

The big takeaway from the current academic state of the practice in this area is that, unsurprisingly, the human-in-the-loop is the bottleneck. If we look only at the growth in science gateway development and utilization in the last 3 years, it becomes glaringly obvious that not finding a better solution to the problem could become of the biggest problems to all academic resource providers, cloud-based or not.

Let's Encrypt
-------------

In theory, this should no longer be an issue. With the launch of [Let's Encrypt](https://letsencrypt.org/) in 2006, anyone can obtain trusted, top level SSL certificates for their domains and subdomains free of charge. Every major web server supports them natively or via plugin. The Let's Encrypt community provides tooling to obtain, manage, and refresh certificates manually, via REST API calls, and on demand as plugins to most common web and application servers. According to the [Let's Encrypt Wikipedia page](https://en.wikipedia.org/wiki/Let%27s_Encrypt#Certificates_issued), as of June of 2017, they have issued over 100 million certificates.

The primary drawback to the use of Let's Encrypt as the de facto SSL provider for academic cloud utilization is the lack of programmable DNS by academic cloud providers. Without such capability, users must still obtain their own hostname and manually update their DNS entries every time they spin up or down a new host. While tedious, this becomes more problematic when propagation time is taken into consideration. When building out infrastructure with modern, distributed footprints, the solution to delays caused by DNS propagation is often disabling SSL on all "backend" components to avoid invalid certificate errors creeping up while DNS finishes propagating far enough to reflect the new records to the CA servers issuing new certs.

Regulation, policy, and documentation are not going to convince developers to stop and wait on DNS every time they need to spin up or down a host. Scale their infrastructure, or migrate a workload. Cloud providers can and should meet their heaviest users in the middle by providing tools to make the right thing to do, the easy thing to do. We recommend that cloud providers begin providing programmable, API-driven, self-service DNS management (DNSaaS) as soon as possible.

Recommendations for Academic Cloud Resource Providers
-----------------------------------------------------

The existing DNSaaS offerings in OpenStack and the open source community provide the majority of the functionality cloud providers should make available to their users. Some additional work may be needed to address specific use cases and security concerts in an academic environment. In the remainder of this section we introduce some terminology to help us shape the requirements that follow.

### Terminology

We define an "**Account**" as a unique identity within the namespace of a given cloud provider's identity provider. Accounts are eternally unique regardless of the existence of federation and locality.

We define a "**Project**" as a organizational resource to which multiple accounts may be associated.

We define an "**Allocation**" as the organizational resource representing a utilization constraint for one or more cloud resources by a given project. Allocations may be extended, renewed, ore expired, but never reused across multiple projects.

Recommendations for Academic Cloud Resource Providers
-----------------------------------------------------

Based on the preceding motivation an use cases, we make the following recommendations.

### Subdomain Namespacing

Initially, externally addressable DNS entries should be created and maintained for each project and account. Namespacing should be done such that any cloud resource can be referenced at the regional and load-balanced at the resource levels. Each project should have a unique hash sufficiently anonymized to avoid conflict with account identifiers. Examples for the Jetstream cloud might be:

```
# regional domains
<hash>.iu.jetstream.cloud
<hash>.tacc.jetstream.cloud

# cross-cloud domains
<hash>.us.jetstream.cloud  
```

Account records should exist as subdomains of their associated project domains. It is recommended that account handles be used in leu of numeric or random identifiers.

```
# regional domains
<account>.<hash>.iu.jetstream.cloud
<account>.<hash>.tacc.jetstream.cloud

# cross-cloud domains
<account>.<hash>.us.jetstream.cloud  
```

### Dynamic resource records

New resources provisioned within a cloud provider should have a unique, externally addressable, domain name assigned to them. For example,

```
<resource>.nova.iu.jetstream-cloud.org
<resource>.trove.iu.jetstream-cloud.org
```

At the same time, subdomain records should be created for the project and creating account. This allows for predictable automation and addressing based on hostname rather than floating ip. The corresponding records within project namespaces would be:

```
# regional domains
<resource>.<hash>.iu.jetstream.cloud
<resource>.<hash>.tacc.jetstream.cloud

# cross-cloud domains
<resource>.<hash>.us.jetstream.cloud  
```

The resource records within the account namespace would be:

```
# regional domains
<resource>.<account>.<hash>.iu.jetstream.cloud
<resource>.<account>.<hash>.tacc.jetstream.cloud

# cross-cloud domains
<resource>.<account>.<hash>.us.jetstream.cloud  
```

### Self-service DNS management

The resource records within the cloud provider namespace should be managed by the cloud provider and inaccessible to end users. Resource records written within the namespace of a project, or account should allow the user to create and manage subdomains, treating each entry as if it were valid top-level domain name. Thus, users would have the ability to define further, arbitrary subdomains for a server they provisioned. One example of how this might look in practice is the following:

```
# primary dns for prod
         jupyter.<resource_1>.<hash>.iu.jetstream.cloud
<user_1>.jupyter.<resource_1>.<hash>.iu.jetstream.cloud
<user_2>.jupyter.<resource_1>.<hash>.iu.jetstream.cloud
<user_3>.jupyter.<resource_1>.<hash>.iu.jetstream.cloud
<user_4>.jupyter.<resource_1>.<hash>.iu.jetstream.cloud
      db.jupyter.<resource_1>.<hash>.iu.jetstream.cloud

# A instance     
         jupyter-a.<resource_1>.<hash>.iu.jetstream.cloud
<user_1>.jupyter-a.<resource_1>.<hash>.iu.jetstream.cloud
<user_2>.jupyter-a.<resource_1>.<hash>.iu.jetstream.cloud
<user_3>.jupyter-a.<resource_1>.<hash>.iu.jetstream.cloud
<user_4>.jupyter-a.<resource_1>.<hash>.iu.jetstream.cloud


# B instance    
         jupyter-b.<resource_1>.<hash>.iu.jetstream.cloud
<user_1>.jupyter-b.<resource_1>.<hash>.iu.jetstream.cloud
<user_2>.jupyter-b.<resource_1>.<hash>.iu.jetstream.cloud
<user_3>.jupyter-b.<resource_1>.<hash>.iu.jetstream.cloud
<user_4>.jupyter-b.<resource_1>.<hash>.iu.jetstream.cloud
```

This is a single hot deployment with two instances of a jupyter hub running in an A/B setup. In this configuration, the user can spin up the vm, create the known user subdomains (wildcard domain entry), repeat for the A an B instances, then request valid SSL certs for all of these subdomains on first request to the server using Let's Encrypt.

```
proxy.<resource_1>.<hash>.iu.jetstream.cloud
app.<resource_2>.<hash>.us.jetstream.cloud
blog.<resource_2>.<hash>.us.jetstream.cloud
api.<resource_3>.<hash>.tacc.jetstream.cloud
queue.<resource_3>.<hash>.tacc.jetstream.cloud
relay.<resource_4>.<hash>.tacc.jetstream.cloud
<resource_3>.trove.iu.jetstream-cloud.org

app.<hash>.iu.jetstream.cloud
blog.<hash>.iu.jetstream.cloud
```

This deployment is made up of a database managed by Trove and 4 VMs. Two VM are at TACC, one is at IU, and one is elastically deployed at either IU or TACC at TACC. The power here is that two public facing DNS entries, `app` an `blog`, were created at the project level. These records point to a load balancer which pulls valid SSL certs from Let's Encrypt for these two domains. The traffic is forwarded to the appropriately named backend server, however because those servers also have outwardly addressable hotnames, SSL certs are pulled from Let's Encrypt for them as well, thus the proxy can be configured to preserve end-to-end encryption. The backend services are also able to maintain valid SSL communication between each other due to the valid SSL cert, thus the application, in addition to the standard firewall production also ensures completly encrypted communication between all components.

### Access control

DNS management within a project should be controlled by their own set of access controls. These can be implemented at the OAuth scope level or independently to enable per-route access to specific domains. The mechanism to grant and control such access control is left up to the cloud provider.

Concluding Remarks
------------------

DNSaaS is a critical feature in today's academic cloud environment. This document presented a set of recommendations for cloud providers to meet the current and future needs of their users while improving their own security and reducing their attack surface. We recommend these recommendations be implemented as soon as possible by all academic cloud providers.
