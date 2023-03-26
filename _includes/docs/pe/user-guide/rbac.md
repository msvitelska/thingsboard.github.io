* TOC
{:toc}
## Overview

Identity access and management (IAM) is essential for today’s business security strategy.
Many IAM systems use a method known as role-based access control (RBAC) to assign permissions for who can do what within specified IT resources.

Role-Based Access Control (RBAC) allows to create and grant advanced access by assigning a set of permissions.
RBAC roles refer to the levels of access that users may have. Access to resources can be limited to specific operations, such as viewing, creating, writing, or deleting data. Similarly, you can restrict access to sensitive information, increasing business security.

## ThingsBoard CE vs PE security features comparison

ThingsBoard Community Edition (TB CE) supports a straight-forward security model with three main roles: System administrator, Tenant administrator, and Customer user. 
A system administrator is able to manage tenants, while a tenant administrator manages devices, dashboards, customers, and other entities that belong to a particular tenant.
Customer user is able to view dashboards and control devices that are assigned to a specific customer.
TB CE functionality is sufficient for many simple use cases, especially building real-time [end-user dashboards](/docs/{{docsPrefix}}user-guide/dashboards/).
 
ThingsBoard Professional Edition (TB PE) brings much more flexibility in terms of user, customer, and role management. 
It is designed to cover use cases for businesses and enterprises with multiple user groups with different permissions but may interact with the same devices and assets. 

TB PE security model was significantly improved to enable new security features and to support advanced RBAC for IoT applications. For example:

  - ability to create a hierarchy of customers with multiple levels of sub-customers, independent users, and devices; 
  - ability to create roles with a flexible set of permissions;
  - ability to assign roles to exact user groups;
  - ability to grant specific permissions to particular user groups over specific device groups.

This document covers features that are exclusive to TB PE. We will start with a glossary and provide step-by-step examples of configuring the most popular use cases.

## Glossary
 
**Tenant**

A Tenant is a separate business entity: an individual or an organization that owns or produces devices and assets. A tenant can have multiple tenant administrator users and millions of customers.

**Entity**

An Entity can be a device, asset, user, dashboard, entity view, etc. Any entity is managed by ThingsBoard. See [entities and relations](/docs/{{docsPrefix}}user-guide/entities-and-relations/) guide for more details.

**Entity Group (EG)** 

Entity Groups are groups of entities of the same type, for example, Device Group or Asset Group. A single entity can simultaneously belong to multiple entity groups. 
For example, a thermostat device might belong to the group "Thermostats", which contains all devices, and a more specific group like "Thermostats with FW v1.2.3". 

**Customer**

A Customer can be a separate business entity: an individual or an organization that purchases or uses tenant devices and/or assets. 
Customer can also be a division within the Tenant organization. 
Customer can have multiple users, inner customers, and millions of devices and/or assets.

**Customer Group (CG)**

The Customer group is also an EG. It has the same features as regular EG, but we have a separate term for CG to be able to easily distinct CGs and all other EGs.

**User**

Users are able to login to the ThingsBoard web interface, execute REST API calls, access devices, and assets if it's allowed. The User is also an Entity in ThingsBoard.

**User Group (UG)**

A User group is also an EG. It has the same features as regular EG, but we have a separate term for UG to distinguish UGs and all other EGs easily.

**Owner**

Each EG belongs to one owner. This can be either Tenant or Customer. 
Also, each Customer has only one owner. If the Customer Owner is a Tenant, it means that this is a top-level Customer.
If the Customer owner is another Customer, this is a sub-customer. There might be multiple levels of Customers in ThingsBoard.

**Resource**

Anything that has secure APIs or represents a ThingsBoard Entity is a resource. 
Examples of Entities are listed in the Entity definition above. Groups of entities are also resources, for example, Device Group, Asset Group, Dashboard Group. 
Additional resources are white-labeling, audit logs, and admin settings. 

Resource types will be described in more detail below in this article.  

**Operation**

Operations represent actions that you might perform over Resources. 
There are generic actions like "create", "read", "write", "delete", "add to group", "remove from group". There are also specific actions like "read/write credentials".

**Role**
 
A Role contains a list of Resources and a list of allowed Operations for each of those resources. There are two Role types: Generic and Group.
There is a special resource "All" which is a shortcut to all available resource types. 
There is also a special operation "All" which is a shortcut to all possible operations.  
We will explain the differences between them later in this article.   
 
**Group Permissions Entity (GPE)**

GPE is basically a mapping between UG, Role, and optional EG. See "Generic roles" and "Group roles" for more details.

## Customer hierarchy

ThingsBoard supports the "recursive" customer hierarchy with an unlimited number of sub-customers. 
The root-level Owner is Tenant. Each Owner may have multiple Entity Groups (EGs), User Groups (UGs), and Customer Groups (CGs).

{% capture difference %}
**Note:** Each Entity has only one owner. However, Entities can belong to multiple EGs that belong to the same owner.
{% endcapture %}
{% include templates/info-banner.md content=difference %}

Since CGs can contain multiple Customers, each Customer can also own his EGs, UGs, and CGs (i.e. sub-customer groups). 
See the diagram below for a visual representation of relations between those entities. 
 
![image](/images/user-guide/security/customer-hierarchy-diagram.svg)

## Roles

Role maps Resource type to a list of allowed Operations. There are two Role types: Generic and Group. Each role type has its own  permissions. 

Please check out resource types and corresponding operations listed below.  

**Available Resources list**:

- API Usage State

API Usage layout is in the main menu of the ThingsBoard platform. API Usage shows full statistics on the platform.

There is an option to create a role with "all", "read", "read telemetry*" operations.


Learn more about API Usage [here](/docs/{{docsPrefix}}user-guide/ui/aliases/#api-usage-state).


- Alarms

Alarms are the platform messages with a specific severity that appear when alarm rules are not observed.

There is an option to create a role with "all", "create", "read", "write" operations.

Check out Alarms in more detail [here](/docs/{{docsPrefix}}user-guide/alarms/). 


- Asset, Asset Group, and  Asset profile 

Assets are one of the main entities types, namely buildings or structures, united in Asset Groups.
Asset profile contains general settings of the Assets. With these permissions, the user can edit the rule chain or create a new one, as well as choose the existing queue from the list.

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry",  "write", "write attributes", "write telemetry" operations.

To know more about the Assets check the article [here](/docs/{{docsPrefix}}user-guide/assets/).


- Audit logs

Audit logs provide opportunity to track user actions. It is possible to log user actions related to main entities: assets, devices, dashboard, rules, etc.

There is an option to create a role with "all", "read" operations.

Check out the article about Audit logs [here](/docs/{{docsPrefix}}user-guide/audit-log/).


- Blob Entity 

Binary large object entity in the reporting feature, in order to store Dashboard states snapshots of different content types.
Blob Entity information represents an object that contains base information about the blob entity (name, type, content type, etc.).
Access to blob entity can be used for generating reports from the dashboard in .pdf, png, and jpeg formats.

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry", "write", "write attributes", "write telemetry" operations.

- Converter

Data Converters is a part of the Platform Integrations feature. There are Uplink and Downlink data converters. If converter resource is chosen, it is possible to add a role with permissions to both of these converters.

There is an option to create a role with  "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry", "write", "write attributes", "write telemetry" operations.

Check the article about Data Converters [here](/docs/{{docsPrefix}}user-guide/integrations/#data-converters/).


- Customer, Customer Group

A Customer is a separate business entity, individual, or organization that purchases or uses tenant devices and assets.
Tenant administrators can create customer groups and assign the role with specific permissions.
The access to Customer or Customer Group level gives the possibility to seе the data available on the upper level. For example, these permissions could be useful for data aggregation.

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry", "write", "write attributes", "write telemetry" operations.

Check out the information in more detail [here](/docs/{{docsPrefix}}user-guide/ui/customers/).


- Dashboard, Dashboard Group

Dashboard is tha visual display of the specific data. Dashboards display data from many entities: devices, assets, etc. Dashboards can be assigned to Customers.
You can create several dashboards and unite them to Dashboard Group, for each entity you can create a role with permissions.

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry", "write", "write attributes", "write telemetry" operations.

Find out more information about the Dashboards [here](/docs/{{docsPrefix}}user-guide/dashboards/).


- Device, Device Group and Device Profile

Devices are basic IoT entities that can produce telemetry data and handle RPC commands. For example, they can be sensors, actuators, switches and any other gauges.
Devices can be joined into a device group entity. Common settings for multiple devices are indicated in Device Profiles. 

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes", "read telemetry", "write", "write attributes", "write telemetry" operations. 

Find out more details about Devices [here](/docs/{{"pe/"}}user-guide/ui/devices/) and Device profiles [here](/docs/{{"pe/"}}user-guide/device-profiles/).


- Edge, Edge Group

Edge refers to the practice of processing data at or near the source in order to reduce latency, improve bandwidth efficiency, and enable real-time processing of data.
   


- Entity View, Entity View Group

Entity View is the ThingsBoard feature, which limits the degree of exposure of the Device or Asset telemetry and attributes to the Customers.

There is an option to create a role with "all", "change owner", "create", "delete", "read", "read attributes, "read telemetry", "write", "write attributes", "write telemetry" operations. 

Learn more about Entity Views [here](/docs/{{docsPrefix}}user-guide/entity-views/). 


- OTA Package

OTA Package is a heavy weight object that includes main information about the OTA Package and also data. It allows you to upload and distribute over-the-air(OTA) updates to devices. As a tenant administrator, you may upload firmware or software packages to the OTA repository.

There is an option to create a role with  "read", "write", "create", "delete" operations.

Learn more about OTA Package [here](/docs/{{docsPrefix}}user-guide/ota-updates/). 


- Scheduler Event 

Scheduler allows to schedule various types of events with flexible schedule configuration.
Scheduler events page displays current configured scheduler events.

There is an option to create a role with "read", "write", "create", "delete*" operations.

Learn more about Scheduler Event [here](/docs/{{docsPrefix}}user-guide/scheduler/).

- Version Control

ThingsBoard Version Control service provides the ability to export and restore ThingsBoard Entities using Git.

There is an option to create a role with "all","delete", "read", "write" operations.

Learn more about Version Control [here](/docs/{{"pe/"}}user-guide/version-control/).


- White Labeling 

Permissions to white labeling allows to configure your company or product logo and color scheme in a short period of time.

You can create a role with "all", "read", "write" operations.

Learn more about White Labeling [here](/docs/{{"pe/"}}user-guide/white-labeling/).



- Widget type and Widget bundle 

There are five widget types in ThingsBoard, each widget definition represents a specific type of widget.

Widgets are grouped into widget bundles according to their purposes. 

You can create a role with "all", "create","delete", "read", "write" operations.

To learn more about Widgets please [read here](/docs/{{docsPrefix}}user-guide/ui/widget-library/).




### Generic roles

Each Role is related to one or more User Groups. Each User Group has only one Owner. 
With the Generic Role, you grant UG with the same permissions over all entities that belong to the same Owner and all its' Sub-customers recursively.
We use a special "connection" object called Group Permission Entity to make a connection between User Group and Generic Role.
Generic  role consists the set of permissions that are applied to all resources of the Tenant or Customer or Sub-customer.


### Adding a Generic role 

To add a new **Generic Role**, navigate to the Roles tab in the left-hand menu and click the plus icon.

Fill out the opened window from top to bottom. Indicate the role name and type, as well as the necessary permissions.
The Generic Role automatically grants User Group access to the **All** folder.

![image](/images/user-guide/security/add-role-window.png)

Click the pencil icon in the top left corner next to the created entity group name to open the entity group details.

Open Roles tab and click the plus icon to add a new role.

![image](/images/user-guide/security/add-role.png)

In the opened Add group permissions window, select the role type and the name of the created role. 

![image](/images/user-guide/security/add-group-permissions.png)


Let's review the diagram below. 

User Bob will be able to perform any operations over any entity that belongs to either his Tenant A or Customer B as long as any other customers and sub-customers are for the same Tenant.
However, User Alice will be able to perform any operations over any entity that belongs to only her Customer B and all it's sub-customers.
So, Alice and Bob are able to access Device B1, but only Bob is able to access Device A1.        

![image](/images/user-guide/security/generic-role-diagram.svg)

### Group roles

Group Role allows you to map a set of Permissions for a specific User Group to a particular Entity Group.
We use special "connection" object called Group Permission Entity to make a connection between User Group, Entity Group and Group Role.  

### Adding a Group role

To add a new **Group Role**, navigate to the Roles tab in the left-hand menu and click the plus icon.

Fill out the opened window from top to bottom. Make sure to include the role type, name, and necessary permissions.
The Group role grants access to the selected one or several entity groups with specific permissions. 

![image](/images/user-guide/security/add-group-role.png)


In the opened Add group permissions window, select the role type and the name of the created role.

![image](/images/user-guide/security/add-group-permissions1.png)


{% capture difference %}
**IMPORTANT**:
When two or more roles are assigned to a user group, the permissions are merged and combined. As a result, the user has permissions for each role.
{% endcapture %}
{% include templates/info-banner.md content=difference %}

### Deleting the group permissions

After selecting the required role, there is an option to delete permissions directly from the Entity group information, as seen in the image below.

![image](/images/user-guide/security/delete-role.png)

{% capture difference %}
**NOTE**:
All chosen group permissions will be deleted after deletion confirmation, and the related users will no longer have access to the specified resources.
{% endcapture %}
{% include templates/info-banner.md content=difference %}

Likewise, you can delete the permissions from the list by simply clicking the trash icon, as seen in the example below.

![image](/images/user-guide/security/delete-permissions.png)

Let’s review the diagram below.

User Bob belongs to the "Tenant Administrators" group and is able to do any operations with any tenant entities. 
Basically, Bob has a full control over both Device Groups A and B. 
User Alice belongs to "Group A Administrators" and has reading/writing access to all devices in device group A. 
However, Alice will not be able to see or use devices from group B.

![image](/images/user-guide/security/group-role-diagram.svg)      

**Note:** Since Entity Group has only one Owner, you can assign Group Role to any User Group that belongs to the same Owner or any parents of the Owner.

## Examples and How-Tos

See list of configuration examples below for the most popular use cases.

### Smart Buildings: Separate User Groups per Facility

Let's assume your solution manages commercial buildings. 
Your main customer is a Building Manager that wants to monitor HVAC systems, electricity consumption, and other smart devices in the building.  
The Building Manager may want to design and share some dashboards with the end-users - office workers.
Besides, your engineers responsible for the maintenance are interested in supervising the devices state, for example, receive alerts when the battery level forgoes below certain thresholds.

To summarize those requirements in ThingsBoard terms, we should implement the following roles:
 * Supervisors - read-only access to all devices' telemetry in all the buildings, and the ability to create their custom dashboards, but no access to dashboards created by users from different user groups.
 * Facility Managers - allows provisioning new devices for each facility, setup thresholds, manage users, and configure dashboards.
 * End Users - allows having read-only access to the state of the facility where this user belongs to.

Let's configure ThingsBoard to support this use case. The instructions below assume that you have logged in as a Tenant Administrator.

**Supervisors**

We will create a separate User Group named "Supervisors", and a separate Dashboard Group "Supervisor Dashboards". 
Our goal is to allow Supervisors to manage dashboards in the "Supervisor Dashboards" group, but for all other entities in the system, they should have read-only access. 

Let's start by creating a "Supervisor Dashboards" group: 
1. In the Dashboard Groups section click on the plus icon in the top right of the screen.
   
   ![image](/images/user-guide/security/entity-create-example.png)

2. Input the name of your Device Group. In our case it's "Supervisor Dashboards". By selecting the Share entity group, you can optionally share the created entity.
3. Click the "Add" button.

   ![image](/images/user-guide/security/add-entity-example.png)

We should create two roles to implement this use case:

 * "All Entities Read-only" - the **generic role** that will allow access to all entities' data except device credentials. 
1. In the Roles section click on the "+" sign at the bottom right of the screen;
2. Input the name "All Entities Read-only";
3. You should choose "Generic" for the Role type;
4. In the Permissions for Resources choose "All";
5. For the Operations input "Read", "Read Attributes", and "Read Telemetry";
6. Click on the lowest "Add" button, the one without a "+" sign.  

   ![image](/images/user-guide/security/role-example1.png)

 
 * "Entity Group Administrator" - the **group role** that allows all operations for the group. 
1. In the Roles section click on the “+” sign at the bottom right of the screen;
2. Input the name “Entity Group Administrator”;
3. You should choose “Group” for the Role type;
4. In the Permissions for Operations choose “All”;
5. Click on the “Add” button.

   ![image](/images/user-guide/security/entity-group-administrator-example.png) 
  
Now let's assign those roles to the "Supervisors" group. 
1. In the User Groups section click on the "+" sign (Add Entity Group) at the bottom right of the screen;
2. Input the name "Supervisors", then click on the "Add" button. You'll see the created "Supervisors" Group;
3. Click on the created "Supervisors" Group;
4. In the opened menu click on the "Pen" sign at the top of the screen;
5. Choose "Roles" and click on the "+" sign at the right top of the opened menu;
6. Choose "Generic" for a Role Type and "All Entities Read-only" in the Options. Click "Add";
   
   ![image](/images/user-guide/security/add-group-permissions-example.png)

7. Again press the "+" sign at the top of the opened window;
8. This time choose "Group" for a Role Type and "Entity Group Administrator" for a Role;
9. For a Group Owner choose "New Tenant", for a Type choose "Dashboard", and click on the "Supervisor Dashboard" in the Entity Group.

   ![image](/images/user-guide/security/add-group-permissions-example2.png) 


**Facility Managers**

We will create a separate Customer entity for each building or group of buildings. We will add the Facility Manager user account to the default "Customer Administrators" group which is automatically created for each Customer.
Now, as Facility Manager, we can log in, design dashboards, provision devices, and end-users.  
1. In the Customer Hierarchy click on the "+" sign (Add Customer) at the top right of the screen;
   
   ![image](/images/user-guide/security/add-customer.png)

2. Input the Title "Building A" and click "Add";

   ![image](/images/user-guide/security/add-customer.png)

3. At the left top of the screen you, shall see the blue icon "All". Click on it;
4. In the drop-down menu follow the path: Building A --> User Groups --> Customer Administrators;
5. On the right side of the screen should have been opened "Customers Administrators: Users", click on the "+" sign at the top right of the screen;

   ![image](/images/user-guide/security/add-user1.png)

6. Input email address, for instance, we can use _alice@thingsboard.oi_, and click "Add";
7. In the opened window you can see the User Activation Link, click "ok".  


   ![image](/images/user-guide/security/add-user-example.png)

**End Users**

Let's log in as Alice (created in a previous guide), Building A administrator, and create several dashboards.
1. Open the Dashboards page. Click on the "+" icon in the top right corner. Select "Create new dashboard".
2. Input dashboard name. For example, "End User Dashboard". Click "Add" to add the dashboard.
3. Now your dashboard should be listed first, since the table sort dashboards using created time by default.
      
![image](/images/user-guide/security/smart-buildings-building-a-dashboards.png)

Now, let's create a Read-only User. Let's assume we want to assign "End User Dashboard" to him and make sure that this Dashboard will open full screen once the user is logged in. 
So, our read-only user will not have access to the administration panel to the left, since they are still not allowed to perform any server-side API calls, except read-only browsing the data.  
1. Choose Customer User in the User Group section;
2. Click "+" at the top right of the screen;
3. Input email address, for example, we will use bob@thingsboard.io, click "Add";
4. In the opened window you can see the User Activation Link, click "ok";
5. Now, click at the created User;
6. At the right top of the screen you shall see the "Pen" icon, click on it;
7. Check the box "Always full screen" and choose "End User Dashboard" in the Select Dashboard menu.


**Below you can find all the information and instructions about this feature**, or you can see the video tutorial for step-by-step instructions on how to use it.

<br/>
<div id="video">  
    <div id="video_wrapper">
        <iframe src="https://www.youtube.com/embed/xpnYzSiDiJo" frameborder="0" allowfullscreen></iframe>
    </div>
</div> 


## Next steps

{% assign currentGuide = "AdvancedFeatures" %}{% include templates/multi-project-guides-banner.md %}

