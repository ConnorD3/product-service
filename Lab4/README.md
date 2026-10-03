# CST8913 Lab 4: Cloud Migration Discovery and Assessment

**Student Name**: Connor Dickson    
**Student ID**: 041085826
**Lab**: Lab 4 
**Course**: CST8913_011 
**Date**: October 3rd, 2026

---

## Digital Estate understanding

The digital estate of an organization is the collective make-up of all of its digital assets. Physical and virtual infrastructure, applications, data, etc. are all part of a digital
estate. Its necessary to understand the digital estate that is to be migrated before planning begins so that the environment that the assets are going to be moved to can be designed to accuratetely emulate the organizations current setup as closely as possible. There are also risks that exist during a migration if adequate discovery and assessments are not conducted. One would be that you would be unable to replicate a functional environment to migrate the estate to. The example infrastructure has not been fully discovered, with the current understanding of the architecture, there would be no way of knowing what the necessary amount of memory, processing, or storage would be to host the current workloads. Another risk would be not knowing what the dependencies between services are. As we likely only know some of the depenendcies of the example organization's estate, we cannot migrate without the risk of breaking services and running into prolonged outages as a result. One last risk is that we would need adequate backups to restore the system if there were any data loss or critical failures during and after migration. The example organization has not provided a backup schedule or backup procedures, which could cause major problems in the event of data loss during a migration.

---

## Discovery and Assessment Approach
The system workloads first need to be identified. It is recommended that Azure migrate appliance is deployed on a server to collect necessary configuration and performance data of the active servers. Azure migrate appliance will run its Discovery and Assessment tool to discover the organization's VMware environment. The configuration and performance data that will be collected includes metadata about the server performance, the current running applications, server dependencies, SQL server instances and databases, etc. The workloads that need to be investigated are PORTAL-01, DATA-01, IDENTITY-01, and SHARE-01.
The additional information that is needed from the organization to complete the assessment are the total cost of ownership of the current digital estate, specify when maintenance can occur outside of business hours, specify the acceptible amount of downtime, what the acceptible amount of data loss is, and provide a backup schedule and procedure.


---

## Dependency Analysis

The confirmed dependencies:
- PORTAL-01 is dependant on ORDER-01 as customer information is input through the portal and sent to the shipment generator.
- ORDER-01 is dependant on DATA-01 as it send generated shipping documents to be stored by that server.

The assumed dependencies:
- PORTAL-01 could be dependent on DATA-01 to recieve product information for displaying on the user interface.
- IDENTITY-01 likely has PORTAL-01 as a dependency as it needs to authenticate employees who are using the portal.
- SHARE-01 is likely dependant on PORTAL for clients to access shared files through authentication.

Using the dependancy analysis option with Azure migrate will identify the processes of applications and their TCP connections to validate what servers are actively communicating and what processes are relying on the information tranferred through those connections.

Dependancy analysis reduces the risk of outages. By not identifying the dependancies, there is a risk of total breakdown of inter-service communication and loss of functionality.

A logical migration group would be ORDER-01 and DATA-01, and also PORTAL-01 as theyre directly dependant upon each other for the core functionality of the business.

![Diagram](./dependencyDiagram)

---

## Azure Readiness and Sustainability

|Server|Readiness Catagory|Justification and Missing information|Remediation or Next Step|
|---|---|---|---|
|PORTAL-01|Insufficient Information|Missing data on assumed dependency with SHARE-01|Dependency analysis will confirm correctness of assumption| 
|ORDER-01|Conditionally Ready|OS version and dependencies are known|Assess application for pending updates|
|DATA-01|Conditionally Ready|Dependencies are known but SQL server version is not|Determine SQL server version through discovery|
|IDENTITY-01|Insufficient Information|Dependencies not identified, no DNS information, no Backup plan for sensitive data loss|Create backup schedule, discover DNS congurations and domain controller redundancy, discover dependencies, consider replacement to handle authentication if migration is found too risky|
|SHARE-01|Insufficient Information|Missing confirmation of dependancy to IDENTITY-01, access methods unknown|Discovery will confirm assumption and identify access methods|

---

## Sizing and Cost Considerations

Azure Migrate supports VM sizing and cost estimation based off workload and performance data collected during discovery. Azure will give extimations in the form of as-is and performance based sizing. As-is sizing is based on the discovered on-premises infrastructure, outlining the cost of allocating enough resources to create an equivalent environment to the current one. Performance based sizing recommends an allocated amount of resources based on performance metrics. To be able to recommend the appropriate sizing for the new infrastructure: CPU and memory utilization, input and output speed along with throughput need to be identified. Allocated compute size, storage capacity, and network scale need to be identified as well. 

Some factors that may affect estimated operating cost:
1. The size of the workload in comparison to the organization's current infrastructure capabilities could create a sizeable difference between the cost of paying for As-is sizing versus paying based on performance.
2. The business traffic based on seasonality may affect the performance measurments, as during certain times the identified performance may not accurately reflect the needs of the business.
3. Unmodernized and inefficient software may cause the use of unnecessary resources, casuing  higher performance based cost than what could be achieved with a modernization or refactoring effort.

The provided inventory is insufficient for producing reliable estimates as business traffic, operating hours, and metrics that can be revealed during discovery like compute utilization and storage capacity have not been provided yet.

---

## Migration Prioritization and Plan

1. Preparation
    - The Azure virtual network needs to be created to reflect the organization's current network and connections between servers.
    - To maintain availability of Identity services and DNS, a replacement routing and authentication service will be integrated into the Azure environment to ensure as little downtime as possible while migrating services critical to operations.
    - Security and access controls that reflect the source infrastructure must be implemented in the new environment so it is already in place once migration has commenced.
    - Backup and recovery schedule and procedure must be prepared. These must also be tested to ensure functionality.
    - Any blocks to migration found during discovery must be remediated to ensure that dependencies and compatibility issues do not cause outages.
    - Maintainance windows, acceptable downtimes, and acceptable amounts of data loss need to be outlined so that parameters can be agreed upon.

2. First Workload and Pilot
    The first workload that will be migrated is a grouping of servers made up of PORTAL-01, ORDER-01, and DATA-01. These are the business critical services that can most likely be migrated without the inclusion of the internal services. To pilot this migration it is recommended that a these services are deployed in a test environment isolated from the current production environment. Azure app testing service would be employed to simulate regular workloads on the test infrastructure and ensure that the migration occurs under near real world traffic. 

3. Migration Order
    The second workload to migrate should be the SHARE-01 file server. This has business related shared files that are no doubt necessary for operations and is likely dependant on the IDENTITY-01 service, but can be sent separate from the order processing functions of the business as it is not dependent on them and can remain operational on the source infrastructure without them. The replacement authentication service can be utilized once the file server has been migrated. IDENTITY-01 can be migrated last, as it is not dependent on the other services and authentication is being already by the replica service, though it is also possible that the IDENTITY be decommissioned and the replacement service left to take over its role. 
   - The first step in the migration order will be to move the core services of order processing and the database, as these are the most necessary parts and need to be migrated and operational first and due its mission critical status.
   - Secondly the shipping document generation needs to be migrated as it is fully reliant on the database for correct functionality.
   - The next service to migrate would be the PORTAL-01 employee access as it is necessary for employees to use for interacting with the order processing elements and is directly dependant on the shipping document generation
   - Authentication and file access would be migrated last as they are less dependant on the others and can continue to operate on-premises independently.

4. Validation
 To confirm that the migrated services are functioning correctly, the following tests will be conducted:
 - Employee sign-in will be tested by entering a series of correct and incorrect credenials into the portal to ansure that authentication is functioning correctly.
 - The availability of the portal will be tested with azure application insights to ensure availability.
 - Testing order and retrieval via the portal, enter various http method requests to ensure correctness of responses.
 - Ensure database connectivity and integrity by testing through various queries.
 - Employ azure monitor to assist in viewing performance post migration.

 5. Rollback
 A condition that would trigger a rollback would be a loss of data outside the limits agreed upon with the organization. Access to the previous environment would be restored by utilizing a physical or virual backup image of the system created as part of the backup schedule. The backup will be restored after source environment and the restored service reimplemented in place of the old one. The data written after the cutover should be disposed of in order to prevent it from being duplicated on a second migration attempt.

---

## References

*Application insights availability tests - azure monitor*. (2026a, September 28). Microsoft Learn. https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability

*Dependency analysis.* (2025a, June 6). Microsoft Learn. https://learn.microsoft.com/en-us/azure/migrate/concepts-dependency-visualization?view=migrate

*Discover installed software inventory, web apps, and database instances*. (2026, August 28). Azure Docs. https://docs.azure.cn/en-us/migrate/how-to-discover-applications?view=migrate-classic

*Insights in Azure Migrate (preview)*. (2025b, September 19). Microsoft Learn. https://learn.microsoft.com/en-us/azure/migrate/insights-overview?view=migrate

*Resolve migration errors - azure devops*. (2026b, August 3). Microsoft Learn. https://learn.microsoft.com/en-us/azure/devops/migrate/migration-troubleshooting?view=azure-devops

*Review azure VM assessment*. (2025c, April 17). Microsoft Learn. https://learn.microsoft.com/en-us/azure/migrate/review-assessment?view=migrate

*Tutorial: Discover servers running in a VMware environment with Azure Migrate*. (n.d.). Microsoft Learn. https://learn.microsoft.com/en-us/azure/migrate/tutorial-discover-vmware?view=migrate

*What is Azure App Testing?*. (2025d, August 7). Microsoft Learn. https://learn.microsoft.com/en-us/azure/app-testing/overview-what-is-azure-app-testing

*What is VMware vCenter?*. (n.d.). OVHcloud. https://www.ovhcloud.com/en-ca/learn/what-is-vcenter/
