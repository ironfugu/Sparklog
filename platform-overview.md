# NodePrime Platform Overivew

![Figure 1. NodePrime platform architecture](https://github.com/ironfugu/docs/blob/master/images/platform-architect.png)

NodePrime® is a hyperscale datacenter management platform that provides multi-vendor hardware control, high-speed scalable timeseries data collection and powerful interfaces for datacenter operators and developers. Designed to manage environments ranging from the smallest test lab to a hundred-thousand node datacenter, the platform centers around flexible data models constructed on the fly for any data type - metrics, logs, blob data and key-value store. Components written in Go power NodePrime technology to discover inventory, alert on system log events and track performance metrics and hardware changes over time. Data is written as fast as it is delivered in a format that easily integrates with other databases, languages and tools.


The NodePrime® platform consist of: 

  1. [DATAHUB](#DATAHUB)
  2. [DIRECTIVE](#DIRECTIVE)
  3. [VDC](#VDC)



All of the NodePrime platforms are buit to work seamlessly together.  


![Figure 2. NodePrime platform in the datacenter ](https://github.com/ironfugu/docs/blob/master/images/platform-overview.png)


***
<div id='DATAHUB'/>
## DATAHUB

NodePrime **DataHub** is composed of three components that work together to provide distributed storage, data collection and analytics capabilities. 

#### Sparklog

![Figure 1. NodePrime DataHub (Sparklog) ](https://github.com/ironfugu/docs/blob/master/images/datahub-io.png)

*Sparklog* is an ultra-high performance, distributed datastore capable of ingesting massive amounts of data (over 100 million metrics per second on a commodity x86 server).  Data ingest is prioritized over query optimization.  Query optimization can be optionally enabled as needed on different types of data.

![Figure 2. Sparklog data ingestion  ](https://github.com/ironfugu/docs/blob/master/images/datahub-different-data.png)

*Sparklog* moves beyond the *all-or-nothing* view of data.  Unlike traditional systems which treat all kinds of data the same way, *Sparklog* considers different elemental data types encountered in data centers: Blobs (e.g. inventory), Logs (e.g. syslog), Metrics (e.g. performance data) and Key-Value pairs (e.g. user and configuration information). Different kinds of data are ingested, stored and processed specifically per type, because some data types need less processing overhead than others. For example, metrics incur minimal overhead, while Key-Value pairs may incur more overhead to provide fully transactional I/O. These considerations are key to Sparklog's design.

In contrast, other systems such as Hbase, Cassandra, Oracle, PostgreSQL, MySQL, MongoDB, and most of the time-series database alternatives incur the same overhead for all types of data. These systems treat data ingestion for all data as if there was only one type, so the overhead of indexing, replication and record locking is a constant penalty.  Telemetry data, metric data, and most logs do not necessarily need the additional overhead. For example, while metric data may need to be queried directly periodically, it is more valuable after further filtering, digesting and analysis has been done.  Bulk metric data from sensor systems might be wrong, useless or never referenced at all.  To realize any value from the vast quantities of metric data, it is necessary to process them first in large amounts. Sparklog prioritizes ingest over query optimization.

Sparklog provides flexible data models, created on the fly, which gives application developers and data scientists options and keeps system administration easy for DevOps. Other distributed solutions end up forcing all types of data over a single-model time-series database on top of HBase, backed by a HDFS cluster, replicated multiple times and perhaps moved into a relational database. These systems often rely on distributed key-value stores that replicate to multipe nodes, fighting for concensus, and frequently reveal the limitations of the centralized service registry model. These approaches are not always the best way.  *Sparklog* treats different data types differently.

![Figure 3. Sparklog views of the data as time series ](https://github.com/ironfugu/docs/blob/master/images/datahub-inventory-change.png)

*Sparklog* treats *all* data as timeseries, even key-value and blob types. *Sparklog*'s view is that *Data is incomplete without meta-data*.  One of the most important meta-data is *time*.  The question, *"What is the value of key A?"*, is incomplete and non-sensical, since by the time the query is received or answered, the value of A might have evolved.  Better questions are *"What was the value of key A over a period of time? What was it at a certain point in time?"*  This view of *data and time* as an integral unit of information avoids further complexities arising out of attempts to achieve consensus in distributed environments. Sparklog explicitly exposes API that includes *time*, instead of hiding *time*, as seen in database implementations that attempt to reconcile the loss of the time parameter from their API by implementing MVCC to realize imperfect transactions and consistency.

*Sparklog* assumes that *all clocks from all sources are out of sync*.  It is unreasonable to assume (or even require) that NTP, PTP or other mechanisms are deployed uniformly throughout all nodes in the cluster at all times.  It is common to find machines in the world's largest datacenters that don't even know in which time zone they reside!  Given this reality, *Sparklog* aims to collect multiple *relative* timestamps as data gets collected, transferred and stored. By tracking multiple sources of timestamps it is possible to detect time skew, propagation delays and other jitters in the cluster.  The timestamps can then be normalized for the final analysis.  The receiving program's point of view of time, at the data sink, is weighted differently than the source's, which may be unaware of its own time drift, or indeed its time zone.  This is related to, yet different, from the widely used vector clocks or logical clocks. *Sparklog* timestamp strategy is based on accumulation of multiple sources of time data over a period of time.

*Sparklog* is designed to deploy as a self-contained, statically linked binary, implemented in *Go Language*.  It is very compact and can be deployed with zero external dependency.

*Sparklog* can be used as a datastore server, client, or even as peer-to-peer storage engine.

*Sparklog* is designed to enable both the streaming of real-time data and asynchronous replication of data without requiring external packages. When streaming data, *Sparklog* works as a component of UNIX-style pipeline, allowing for flexible fan-in and fan-out patterns to ingest, import, process and export data.

*Sparklog* is built to work with existing data sources such as OpenTSDB input, graphite, nagios, logstash, collectd, etc.  It is also designed to work with different data backends, including files, networked file systems, HDFS, RethinkDB, ElasticSearch, Cassandra, etc.


#### PrimeView

*Primeview* is NodePrime's dashboard, unifying many of NodePrime's high level features under one pane of glass.  It provides datacenter operators with an integrated solution for deploying and managing datacenter hardware. Various analytics modules allow operators to perform compliance and forensic tasks. 

![Figure 4.  Inventory dashboard ](https://github.com/ironfugu/docs/blob/master/images/dashboard.png) 

The default *dashboard* shows status of all NodePrime services.  There are task oriented work flow designed for typical datacenter use cases.  

![Figure 5. Node detail view](https://github.com/ironfugu/docs/blob/master/images/detail-inventory-time.png)

#### Agents

*Sparklog* supports many different kinds of agents which are data collectors. Collecting and sending data is as easy as writing a line of text over the network. The format of data is deliberately made simple, so that even very resource constrained devices can be programmed to send out metrics.

![Figure 6. NodePrime Agent collector configuration](https://github.com/ironfugu/docs/blob/master/images/agent-collectors.png)

NodePrime provides a set of user-extensible reference agents and examples.


***

<div id='DIRECTIVE'/>
## DIRECTIVE

![Figure 7. NodePrime Directive dashboard ](https://github.com/ironfugu/docs/blob/master/images/directive-dashboard.png)

*NodePrime Directive* is a network-level solution for discovering, deploying and managing hundreds and thousands of servers in a datacenter environment. Able to fix network boot problems and deploy in-band firmware upgrades on demand, Directive integrates seamlessly with DataHub to profile your network by collecting inventory and network metrics for later analytics.

The *Directive* incorporates standalone DHCP, TFTP, HTTP and DNS services to support network booting and post-boot management without interfering with existing network infrastructure. It intelligently scans a network segment and reports what it finds to DataHub to ease the task of OS and Agent deployment. A rich API and CLI gives operators and developers plenty of customization options.

#### Scanning

*Directive* can scan the existing networks using many different active and passive strategies at various levels of protocols to discover machines and types of machines.

Some of the protocols used are: ICMP, ARP, LLDP, SNMP, IPMI

#### Integrated Provisioning

*Directive* integrates a set of protocols to work together in deployment scenarios. These protocols, DHCP, TFTP, HTTP and DNS are implemented in a single zero-dependency binary which require almost no configuration.  The tight integration allows for less errors, tedious editing and setup of multiple daemons, and better information sharing between protocols.  For example, DHCP lease information can be used by DNS server within the *Directive* to serve the set of machines provisioned by the *Directive* the DNS name services for the very same set of machines, before proxying out the main DNS server.  The DHCP requests can also be used to trigger instantiation of a virtual machine which leverages the existing mature protocols for cluster orchestration.  Furthermore, the *Directive* implementation of these protocols are carefully constructed to avoid most of the common pitfalls present in other implementations.  For example, most DHCP servers will not check to make sure there are other DHCP servers present in the subnet.  Frequently this creates massive confusion, when multiple DHCP servers lease out addresses. The *Directive* implementation first detects such conditions and avoid conflicts.


#### Deployment

*Directive* has a set of APIs and modules to allow PXE booting clients, zero-touch provisioning of bare metal servers compatible with KickStart, YaST and others.   *Directive* also allows updating low-level firmware for motherboards, different perpherals and devices.

***

<div id='VDC'/>
## VDC

True immutability only exists when the OS, network topology and application deployment strategy are defined in data, as a blueprint, which can be repeated predictably every time. NodePrime Virtual Data Center enables developers and datacenter operators to create an environment for rapid simulation, examining results, changing the parameters of the runtime environment _in blueprint textual representation, like JSON_ and then repeat for comparison.

![Figure 8. NodePrime Vitual Datacenter  ](https://github.com/ironfugu/docs/blob/master/images/vdc-overview.png)

#### DevDC

*DevDC* allows CLI and HTTP API which can leverage the _blueprint_ definition of a Data Center to create specific contents of the Virtual Data Center, which are virtual network switches, virtual network interfaces, configuration of the switches and interfaces, network routing and switching flows, virtual machine instances, container instances, virtual BMC and IPMI simulator, kernel versions and images, disk contents and filesystem images, command line tools and libraries in the filesystem, booting strategies, application setup and initialization as well as provisioning methods for uninitialized virtual machines via PXE booting.

*DevDC* includes tools to build an entirely reproducible, *immutable*, instances of complete Data Center simuation.  *DevDC* tools and workflow include both very low level and high level customizations that can be prescribed in the _blueprint_.  Low-level details can be the specific devices and firmware emulation, features of the custom kernel which is built from scratch, contents of the root filesystem which is built from scratch, etc.  High-level details can be the networking flows, application initialization and configuration.  This allow for the real *full-stack* approach of conceiving and constructing  applications that can be repeatedly tested and deployed.

#### SimDC

*SimDC* is an example UI for the Virtual Data Center.  It allows loading the JSON blueprint of a Data Center and compiles it into a running simulation, which is then visualized and rendered inside a browser.

![Figure 9. NodePrime simDC Use Cases  ](https://github.com/ironfugu/docs/blob/master/images/simDC-usecase.png)


