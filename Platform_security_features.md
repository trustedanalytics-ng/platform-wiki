#Platform Security Features
## Overview
The Trusted Analytics Platform (TAP) is a big data analytics platform designed with security from the ground up. “Trust” applies to TAP as both: 
- A “trusted platform” that provides strong security mechanisms and simplifies their use, and
- An enabler of “trusted analytics”, preserving the confidentiality and privacy of data while it is analyzed.

The mechanisms of trust embodied in TAP are based on principles of layered defense implemented with proven success in trusted operating systems and cloud security.  

TAP architecture centers around two core components – Cloudera Hadoop (CDH) and Kubernetes (K8s). TAP integrates these two components in a seamless manner. Remaining TAP elements build upon these core components or provide services for them, as follows:

**Amazon Web Services (AWS)** – an Infrastructure as a Service (IaaS) layer needed to operate CDH and K8s. It provides APIs for VM creation, storage, and network machinery. 

**Deployment Automation** – a set of tools and configuration scripts that help deploy the whole system, including end-user applications, in a target environment.

**Cloudera Distribution of Hadoop (CDH)** – a Big Data software stack.

**Kubernetes (K8s)** – platform for developing and hosting cloud applications.

**TAP Integrated and Custom-Built Services** – This layer is comprised of different types of services for users to create applications. These services include a myriad of component types including primitive open-source services, like MySQL or Redis, as well as in-house developed components, like Data Catalog and Model Catelog.

**End-User Applications** – This layer enables application developers to build custom application on top of the TAP stack by leveraging the platform services.

## Security Mechanisms
Platform security includes several well-defined mechanisms to protect applications as well as data:    
- **Authentication**: TAP inherits and extends the authentication mechanisms enabled in CloudFoundry (CF) by integrating them with the authentication mechanisms in Hadoop. While CF uses [OAuth 2.0](http://oauth.net/2/), Hadoop uses [Kerberos](http://web.mit.edu/kerberos/). TAP integrates the two via a custom security gateway, which allows a CF application to authenticate a user via OAuth 2.0 and convert the OAuth token into a service ticket that is accepted by Kerberos. This provides the user with a Single-Sign On (SSO) experience across the data and application layers.    
- **Authorization**: TAP delegates authorization to CF and the individual data components. More details are included below.
- **Isolation**: TAP uses [Warden containers](https://docs.cloudfoundry.org/concepts/architecture/warden.html) in CF to enforce application isolation. TAP enables isolation of data sets by limiting visibility to users on the basis of their parent organization and their scope of access. TAP uses a combination of OAuth tokens and the metadata of a data set (indicating scope of access) to make this determination.   

TAP also addresses the following data and analytics security concerns:
- **Confidentiality**: TAP protects the confidentiality of data by enabling encryption with performance parity, which allows all data at rest and in motion to be encrypted at all times without loss of application performance. TAP users can configure Hadoop transparent data encryption. With transparent encryption, data read from and written the Hadoop file system (HDFS) directories is transparently encrypted and decrypted without requiring changes to user application code. More details can be found at the [Hadoop documentation page](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/TransparentEncryption.html).
- **Privacy**: Privacy-preserving transactions are considered on the roadmap.

## User Management
TAP uses CF’s [User Account and Authentication server (UAA)] (https://docs.cloudfoundry.org/concepts/architecture/uaa.html) to authenticate users and inherits CF’s access control system, in which users are assigned to a specific organization and space and can have various roles in these organizational units.

- Organization: An organization is a development account that an individual or multiple collaborators can own and use. All collaborators access an organization with user accounts. Collaborators in an organization share a resource quota plan, applications, services availability, and custom domains.
- Space: Every application and service is scoped to a space. Each organization contains at least one space. A space provides users with access to a shared location for application development, deployment, and maintenance. Each space role applies only to a particular space.

### Roles and Permissions
A user can have one or more roles. The combination of these roles defines the user’s overall permissions in the organization and within specific spaces in that organization.
The organization-level roles include the Org Manager, Org Auditor, and Billing Manager. The space-level roles include the Space Manager, Space Auditor, and Space Developer. The description of these roles and their associated permissions can be found [here](https://docs.cloudfoundry.org/concepts/roles.html).

### User Registration Process
To register a new user, a system administrator sends an email with an invitation to create a user account and a new organization. The registration process starts by generating a Security Code and storing it in a Security Codes Database (a Redis database). Then the [User Management service] (https://github.com/trustedanalytics/user-management) sends the invitation email with a URL with the Security Code using an external SMTP Service. Invited users use the provided URL to access the registration form.

## Audits and Logs
The TAP platform provides auditing capabilities on multiple levels, as follows:

### Cloud level
The underlying TAP IaaS providers, such as AWS and OpenStack, support auditing and logging capabilities. They allow logging of each change in the underlying infrastructure. More details can be found on their respective sites:
- [AWS auditing] (https://aws.amazon.com/cloudtrail/)
- [OpenStack auditing] (https://www.openstack.org/summit/openstack-summit-atlanta-2014/session-videos/presentation/an-overview-of-cloud-auditing-support-for-openstack)

### Infrastructure level
BOSH, the daemon handling infrastructure changes in the platform, provides a history of all of the changes done to Cloud Foundry components. It shows the detailed changesets made to the platform, along with a debug log of the process. More information is available in the [provided documentation](https://bosh.io/docs/sysadmin-commands.html#tasks).

### Application level
Cloud Foundry allows the operator to access two main streams from an application – events and logs. Events are provided by the platform and include events such as application restarts, while logs are provided by the application itself, reporting on its inner workings. More information is available in the [CF documentation](https://docs.cloudfoundry.org/devguide/deploy-apps/troubleshoot-app-health.html).
Cloudera offers two types of logs – one for the management services and the second for Hadoop services running on it. Documentation for the management part can be found [here](http://www.cloudera.com/documentation/archive/manager/4-x/4-7-1/Cloudera-Manager-Diagnostics-Guide/cmdg_logs.html), while the Hadoop services log is documented [here](http://www.cloudera.com/documentation/archive/manager/4-x/4-6-1/Cloudera-Manager-Diagnostics-Guide/cmdg_view_server_and_agent_logs.html).

Additionally TAP utilizes the [LogSearch](http://www.logsearch.io/) stack to expose the infrastructure and application level logs via [Kibana](https://www.elastic.co/products/kibana); an easy to use web interface that enables browsing, filtering, and charting of logs. Currently, CF logs are exposed within the LogSearch and Kibana integrations. The integration supports SSO to enable users to log into the web interface using their TAP account credentials and enforce their roles access rights.
