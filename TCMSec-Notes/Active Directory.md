## What?

Directory service - just a big phone book
Stores info on objects (Users, Comps, Printers, etc)
Uses Kerberos tickets for Auth:
- RADIUS
- LDAP

Not just for windows, can use for Linux or firewalls

Most attacks are abuse of features, trusts and components

## Physical Components

### Data Store

- AD DS
- Contains the database files and processes for storing 'n' managing directory info
- Contains **Ntds.dit**
- Stored by default in the *%SystemRoot%*\NTDS folder on all DCs
- Only accessible through DC processes and protocols

### Domain Controllers

- Head Honcho
- Hosts a copy of the AD DS (directory store)
- Provides AuthN and AuthZ
- Allows admin access to manage user accounts and net resources
- Replicates updates to other DCs in the domain and forest
### Global Catalog Server

### Read Only Domain Controller

## Logical Components

### Schema
- Defines all object types in directory
- Class objects have attribute objects attached
### Domain
- Domains used to group and manage objects in an org
- An admin boundary for applying policies to groups of objects
- Replication boundary for replicating between DCs
- AuthN and AuthZ boundary to limit scope of access to resources
### Tree
- Hierarchy of domains
- Share a contiguous namespace with parent domain
- Can have additional child domains
- By default - two-way transitive trust between domains
### Forest
- Share common schema
- share common config partition
- share common global catalog for  searching
- enables trusts between all domains in forest
- share Enterprise Admins and Schema Admins groups
### Organizational Units (OUs)
- OUs are AD containers that contain users, groups, computers and other OUs
- Represent your org hierarchically and logically
- Manage a collection of objects in a consistent manner
- Delegate perms to administer groups of objects
- apply policies at OU levels
### Trust
- Direction: Domain trusts another domain
- Transitive: Extended beyond just 2 domains (x == Y == Z)
- Trusts within forest are transitive across domains in forest
### Objects

| OBJECT | DESCRIPTION |
| ---- | ---- |
| User | - Enables network resource access for a user |
| InetOrgPerson | - Similar to a user account<br>- Used for compatibility with other directory services |
| Contacts | - Used mostly to assign email addresses to external users<br>- Not for enabling network access |
| Groups | - Simplifies admin of access control<br>- Groups of objects |
| Computer | - Enables authN and audit of computer access to resources |
| Printer | - Simplifies locating and connecting to printers |
| Shared Folders | - Enables users to search for shared folders based on properties of folders |

## Kerberos