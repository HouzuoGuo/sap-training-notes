# sap-training-notes
Notes were taken from SAP software usage training that took place in early 2015. 

# The notes
SAP : presentation server + app server + database. 
2-tier setup: app server + db on one host
3-tier setup: app server and db run on different hosts

Scaling by having multiple app server.

SAP ERP has 80k tables

“SAP industry solutions” for industries – med care/mining/financial,etc. Each solution runs their own tables, but all in one database.

Installing ERP gets you all industry solutions, all tables are defined, but not all are used.

One example: 600 GB DB, generic tables occupy lot of space.

Program app code is actually stored in DB. Compiled code stored in another table. OS/DB independent. Special SQL is used “ABAP Sql.
OpenSQL/EndSQL for db-vendor-specific code.

Customer may program SAP code via “Modification Assistant” - a GUI program. Used by support engineering too. Customer may even create new programs – under a naming convention “z” “y” namespace reserved for customer.

SAPGUI are executables, installable.
Some applications ship web UI.
Uis talk to app servers – executables (“sap kernel”) installable, layers?
App server has - ABAP intepreter; DBSL – libraries for translating ABAP code into DB-vendor SQL dialect. 
In total, the app servers are about 300 MB. App servers are DB/OS specific. All other business logic programs are not db/os specific.

HANA is written in ABAP, uses an additional app server component, with HANA db components.

Service.sap.com authentication uses “S user”. Download SAP softwares.
.com/notes is the knowledge base, usage tips, docs.
.com/swdc is software download center
.com/patches also for software download.

DB server, another term is “Central Instance” , runs app server, in 2-tier setup. Typically 32GB-128GB ram, 4-8 CPU sockets.

SAP benchmark offers advice of how-many-users-can-run-on-this-system. E.g. SD (sales distribution) benchmark. It involves CPU-utilization goal, response time goal. Benchmark result is “number of SD concurrent users”, plus a KPI called “SAPS”.

In 3-tier setup:
Application servers can scale vertically and horizontally. Load-balancing is provided.
Central processes: Message server process handles load balancing. MQ process handles transaction locking, across app servers. The central processes are single point of failure.

SAP GUI runs natively on Windows. Other options exist: java GUI or web GUI.

ABAP server is responsible for that SAP's own lang.
Java app server is an infrastructure for deploying and running Java apps in NetWeaver. Acquired from a Bulgarian company. Only used for presentation logic, not for business logic.

RPC at SAP is called RFC (function).

ABAP dev environment comes with every SAP implementation, fully integrated.

SAP GUI form is called DYNPRO (dynamic program?). Webdynpro can do Java.

SAP processes look like:
disp+work.exe
And: pgrep dw
Dispatcher is the parent process, it looks at SAP profiles (configurations), which tells how many worker processes to start.
/usr/sap/<sid>/sys/profile is the directory of config files.

SAP GUI connects to dispatcher via hostname:port combination.
Dispatcher dispatches requests to an idle worker process.
A transaction (initiated by user) occupies a worker process for a period of time, and is vacated after transaction is completed. The scheduling mechanism works like OS scheudler.
Number of available worker processes determines max number of concurrent users. Usually an app server is able to handle several hundreds of users at once.

Types of work processor: D – dialogue, V – verbucher (update), E – enqueue (lock management), B – batch (long-running process, avoid blocking D workers), M (controller of processes), S (spool – management of batch process output).

V work processor handles async updates, for better UI responsiveness.

G Gateway is a special type of worker, handling remote connection, remote function call.

The central app + db instance is called DVEBMGS, additional dialogue instances enable horizontal scaling – these instances may run more than dialogue work processors.

The central database has an SID (SAP system ID), 3 upper case char/number. OS user is automatically created <SID>adm. OS group is sapsys. The user belongs to additional DB group(s).

Oracle user, for example, is owned by ora<SID>, group is dba.

SAP uses a single user with full authority to connect to DB, and the user owns all DB tables. The DB owner name is sap<SID>.

Environment profiles are generated during installation, file names are: sapenv_<hostname>.sh/csh

SAP has built-in authorization system to control DB access of application users. Authorities of these users are not mapped to DB permission model.

Java app uses its own DB, with several hundreds of tables and several hundreds MB of data. Java app server uses “ICM” for dispatching, each Java instance runs several server processes – they are fewer than worker processes.

SAP ERP is implemented in ABAP, no Java parts.
CRM has both ABAP + Java parts.
Supplier relationship management has both ABAP + Java parts.
Supply chain management has both ABAP + Java parts.
Enterprise Portal is implemented entirely in Java.
Business warehouse is ABAP entirely for business logics, Java for portal.
Solution manager uses both ABAP and Java.

Customers wish to install SAP in modules, in order to reduce QA effort of z-program when upgrading.

SAP recommends running ABAP and Java on separate hosts.

Instance number becomes part of port number.
In an HA setup, SCS<ID> instance runs M,E processes, “sap central services”.

SAP runs on Solaris, AIX, HPUX, Windows, and enterprise Linux.

SAP supports DB: Oracle, DB2, MSSQL, MaxDB (SAP's own DB)
SAP HANA runs on Linux.
SAP ASE is Sybase ASE.

How much does SAP cost? Depends on user type, country, and industry, here is an example (EUR/user):
Plus percentage of user license for type of DB
Plus maintenance subscription
RTDP (runtime data processing) license covers HANA, MaxDB, ASE, costs user license fees. Good for migration.


The attribute (key) “Client” represents a company, business unit, or subsidiary, in an SAP deployment. 

Data dictionary is vendor neutral. Migration is trivial.

“Pool of tables” is a legacy feature, workaround for table limitations. May go away with HANA.

SAP application modules and “kernel” versions are more-less arbitrary. NetWeaver was invented in 2004, and “kernel” version numbers were aligned with its introduction.

SAP_BASIS and SAP_ABA are the foundation modules (plus few other modules?) belong to the “kernel”.

SAP business suite (incl. AllInOne, ByDesign), HANA, NetWeaver, StreamWork run on SLES (incl Cloud).
BW Accelerator used to be SUSE only.
Business By Design is SUSE only.

SAP DB export tool generates Arch/OS/DB neutral output. DB migration prefers using identical SID on source side and target side. Migration may take several weeks. Backup-restore takes 1-2 days depends on size of DB. Post-processing procedure may take several hours.

Certified SAP migration consultant is mandatory for DB-vendor migration, or Platform-vendor migration. Such migration requires special tool, and key, and the certified consultant's SAP user ID.

For DBMS/OS upgrades, SAP offers update installer substitution, do not install vendor's release.

Select * from cvers/svers – module version numbers

SAP kernels uses UTF-16. Non-unicode to unicode migration requires additional memory and disk space – expect up to 100% more disk space consumption. Note that, by enabling DB compression and re-importing DB, the disk usage may even go down.

R3load is the DB export/import tool, but it is almost always not called directly; instead, call sapinst (legacy), swpm (software provisioning tool) instead. Answer questions such as – how many processes to run in parallel? Should the database be split into packages? Do test runs to explore parallelism and determine best course of action. Consider running export and import in parallel.

SAP shipped ABAP code are tested in both non-unicode and unicode environments. Customer needs to analyse their own ABAP programs when migrating, the process can be done using SAP transaction “check a program set for syntax errors in unicode environment”.

Running on Linux, SAP requires kernel parameters to be adjusted to accommodate large memory requirements. SWPM installs SAP applications and creates configuration, users, groups, etc.

HANA is actually “high performance analytical database”. In-memory DB, offering compression, both row and column store. Performance is good enough, makes aggregation tables obsolete.

With HANA, some application code are translated into DB executable code, for efficient memory usage.

HANA deployment may support a single SAP module, or multiple SAP modules at once.

New approach, S/4 HANA, reduces number of DB tables, by combining table fields, and creating “views” for the legacy “helper” tables – e.g. simplified financial data model, simplified logistics data model.

HA400 offers training on efficient use of HANA in ABAP

Migration of data from other DB vendor to HANA requires usage of a new tool called “Software Update Manager with Data Migration Option”.

In HA setup, assign Unique ID numbers to instances.
This is not a requirement on non-HA setup.

SAP supports SUSE, Redhat, Oracle. SAP Linux Lab was founded in 2001.

SAP+Linux can use various HA/Database/Distributions/Hardware solutions, choice is up to the customer.

SAP OSS (support system) connects customer with SAP partners in finding resolution for OS/hardware/SAP product issues. OSS is not catered for individual SLA signed between partner company and customer, however.

Virtualized SAP runs on KVM, Xen, Vmware. LXC and HyperV are not yet supported.

SAP avoids installing binary kernel modules, where possible. Exceptions can be made, given that binary module is certified by SAP, and support agreement is reached – costly. There are currently only 9 binary module vendors.

SOH “Suite On Hana” - use Hana as a substitution of backend database.
S4 is the next generation SAP system. S4HANA uses HANA backend exclusively, and uses simplified schema. S4 means “simplified fourth generation”.

HANA uses replay journal for persistence.

HANA can be deployed on a single server (multi-socket CPU, TB ram), using standby nodes for high availability.
Clustered deployment may involve max. 100+ nodes, each server is multi-socket CPU; data is sharded in this configuration; better performance for aggregation workloads, but slower for joins.
Cloud deployment is available too on AWS.

HANA ships pre-configured appliance, with rigorous performance guarantee.

HANA HA option – replication (active standby?): having two active instances, one primary, the other seconday. Both instances use the same instance ID. In the beginning, primary site supports read/write, and secondary site is replicated from the primary's journal. Durability is tunable – with optional ack from secondary site for each transaction. In case of primary malfunction, both primary role and IP become effective on secondary. AUTOMATIC_REGISTER=false prohibits a previously failed primary to become new secondary, could be useful for admin diagnostics.
Additional requirements:
- Time in sync
- User and host name resolution
- Physical and SAP-specific hostnames
- Careful with geo-distance and communication latency (maintain less than 15 millisecs)
Parameter SID “SAP identifier”, InstanceNumbers (SAP instance number) are identical on both hosts; “PREFER_SITE_TAKEOVER” true if secondary is preferred, false if restart is preferred.

Mandatory – follow the setup guide which belongs to SAPHanaSR package.

HANA resource agent knows replication status via internal mechanisms, only if replication data is all caught up on secondary site, switch over to secondary is allowed.

Sapstartsrv, sapcontrol, “hdb?” for startup.
landscapeHostConfiguration.py generates topology output, site status, failing nodes, etc.
hdbnsutil generates replication, synchronization information.
saphostctrl is the host agent, also used for hostname/nodename manipulation?
hdbsql (previously systemReplicationStatus.py) asks primary site for internal replication status.

A more cost-effective approach is to run additional application workload on secondary DB, in this case customer may prefer to restart primary rather than letting secondary to take over.

HANA HA option – double replication. Primary site synchronizes to secondary site (sync op), secondary site synchronizes with a third site (async op). A single site does not synchronize to two other sites, therefore it is called “chain replication”.

SAPHanaSR on SUSE offers these advantages:
- Reduced complexity – UI wizard for configuration, using SID, IP, instance number.
- Reduced risk - ??? ads

SUSE is still the no.1 platform for HANA deployment, even though Redhat introduced similar offering. 
