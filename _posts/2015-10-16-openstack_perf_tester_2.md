---
layout: post
title: openstack性能测试器(2):消息调用过程oslo.messaging
date: 2015-10-16 00:00:00
categories:
- code
tags: 
- openstack
- perfmance
- AMQP
mathjax: true
description: 
---

# openstack简介
The OpenStack project is an open source cloud computing platform that supports all types of cloud environments. The project aims for simple implementation, massive scalability, and a rich set of features. Cloud computing experts from around the world contribute to the project.
<!--more-->

OpenStack provides an Infrastructure-as-a-Service (IaaS) solution through a variety of complemental services. Each service offers an application programming interface (API) that facilitates this integration.

# openstack组件
Service|Project name|Description
------|----------|--------
Dashboard|Horizon|Provides a web-based self-service portal to interact with underlying OpenStack services, such as launching an instance, assigning IP addresses and configuring access controls.
Compute|Nova|Manages the lifecycle of compute instances in an OpenStack environment. Responsibilities include spawning, scheduling and decommissioning of virtual machines on demand.
Networking|Neutron|Enables Network-Connectivity-as-a-Service for other OpenStack services, such as OpenStack Compute. Provides an API for users to define networks and the attachments into them. Has a pluggable architecture that supports many popular networking vendors and technologies.
||Storage
Object Storage|Swift|Stores and retrieves arbitrary unstructured data objects via a RESTful, HTTP based API. It is highly fault tolerant with its data replication and scale-out architecture. Its implementation is not like a file server with mountable directories. In this case, it writes objects and files to multiple drives, ensuring the data is replicated across a server cluster.
Block Storage|Cinder|Provides persistent block storage to running instances. Its pluggable driver architecture facilitates the creation and management of block storage devices.
||Shared services
Identity service|Keystone|Provides an authentication and authorization service for other OpenStack services. Provides a catalog of endpoints for all OpenStack services.
Image service|Glance|Stores and retrieves virtual machine disk images. OpenStack Compute makes use of this during instance provisioning.
Telemetry|Ceilometer|Monitors and meters the OpenStack cloud for billing, benchmarking, scalability, and statistical purposes.
||Higher-level services
Orchestration|Heat|Orchestrates multiple composite cloud applications by using either the native HOT template format or the AWS CloudFormation template format, through both an OpenStack-native REST API and a CloudFormation-compatible Query API.

## oslo.messaging
消息组件，其中的/_drivers/impl_rabbit.py就是rabbitMQ的具体实现。我们要用的AMQP就是靠它来完成。

# 一个call的前世今生
## neutron/common/rpc.py init
```
common/rpc->oslo_messaging/transport:get_transport
oslo_messaging/transport->stevedore:DriverManager
stevedore->impl_rabbit:get impl_rabbit
impl_rabbit-->stevedore:return impl_rabbit
stevedore-->oslo_messaging/transport:return driver
oslo_messaging/transport-->common/rpc:return transport
```


![](https://github.com/CodeJuan/codejuan.github.io/blob/master/images/blog/amqp/1.png)


## neutron/plugins/ml2/drivers/openvswitch/agent/ovs_neutron_agent.py
```
OVSNeutronAgent->OVSNeutronAgent: __init__
OVSNeutronAgent->OVSNeutronAgent: setup_rpc
setup_rpc->OVSPluginApi: plugin_rpc = OVSPluginApi
OVSPluginApi->PluginApi: __init__
PluginApi->oslo_messaging/target:tartget::__init__
oslo_messaging/target-->PluginApi: return target
PluginApi->common/rpc: get_client
common/rpc->oslo_messaging/rpc/client:construct RPCClient
oslo_messaging/rpc/client-->common/rpc: return RPCClient
common/rpc-->PluginApi: return client
```

![](https://github.com/CodeJuan/codejuan.github.io/blob/master/images/blog/amqp/2.png)


## call
```
OVSNeutronAgent->OVSPluginApi.plugin_rpc:tunnel_sync
OVSPluginApi.plugin_rpc->PluginApi:tunnel_sync
PluginApi->oslo_messaging/rpc/client.RPCClient:prepare
oslo_messaging/rpc/client.RPCClient->oslo_messaging/rpc/client._CallContext:_prepare
oslo_messaging/rpc/client._CallContext-->oslo_messaging/rpc/client.RPCClient:return callContext
oslo_messaging/rpc/client.RPCClient-->PluginApi:return callContext
PluginApi->oslo_messaging/rpc/client._CallContext: call
oslo_messaging/rpc/client._CallContext->oslo_messaging/rpc/client._CallContext:make message
oslo_messaging/rpc/client._CallContext->transport:send
transport->AMQPDriverBase:_send
AMQPDriverBase->oslo_messaging/_drivers/amqp:get connection
note right of oslo_messaging/_drivers/amqp: get connection from pool
oslo_messaging/_drivers/amqp-->driverbase:return connection
AMQPDriverBase->AMQPDriverBase: get exchange
AMQPDriverBase->ReplyWaiter: _get_reply_q
AMQPDriverBase->impl_rabbit:connection.topic_send
impl_rabbit->impl_rabbit: connection._ensure_publishing
AMQPDriverBase->ReplyWaiter: wait
ReplyWaiter-->AMQPDriverBase: return result
AMQPDriverBase-->transport: return result
transport-->oslo_messaging/rpc/client._CallContext: return result
oslo_messaging/rpc/client._CallContext-->PluginApi: return
PluginApi-->OVSPluginApi.plugin_rpc: return
```

![](https://github.com/CodeJuan/codejuan.github.io/blob/master/images/blog/amqp/3.png)



----------------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

