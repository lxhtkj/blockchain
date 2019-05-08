POW：【比特币】工作量证明机制，按劳分配  优点：本身机制复杂：挖矿难度自动调整，挖矿奖励逐渐减半；越早挖矿，收益越多；挖矿实现酬劳的相对公平

缺点：耗费电力；发展造成算力集中，与去中心化背道而驰；挖矿奖励越来越少，安全风险增加

 

POS：【点点币】股权证明机制   优点：节能，不依靠算力获得、更去中心化，避免紧缩【用户遗忘导致币越来越少】；缺点：IPO发行导致少数人参与；信用基础不牢固；

DPOS：【EOS】授权股权证明机制  优点：能耗更低，将POS的多节点在线简化为101个节点；相比POS需要用户时时打开电脑，更去中心化；确认交易的速度更快；

缺点：投票积极性不高；处理坏节点困难，给网络安全造成隐患

PBFT

POOL

 

 

核心特性：

·解耦了原子排序环节与其他复杂处理环节，消除了网络处理瓶颈，提高可扩展性；

·解耦交易处理节点的逻辑角色为背书节点（Endorser）、确认节点（Committer），可以根据负载进行灵活部署；

·加强了身份证书管理服务，作为单独的FabricCA项目，提供更多功能；

·支持多通道特性，不同通道之间的数据彼此隔离，提高隔离安全性；

·支持可拔插的架构，包括共识、权限管理、加解密、账本机制等模块，支持多种类型；

·引入系统链码来实现区块链系统的处理，支持可编程和第三方实现。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps40.jpg) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps41.jpg) 

 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps42.jpg) 

Fabric为应用提供了gRPCAPI，以及封装API的SDK供应用调用。应用可以通过SDK访问Fabric网络中的多种资源，包括账本、交易、链码、事件、权限管理等。应用开发者只需要跟这些资源打交道即可，无需关心如何实现。其中，账本是最核心的结构，负责记录应用信息，应用则通过发起交易来向账本中记录数据。交易执行的逻辑通过链码来承载。整个网络运行中发生的事件可以被应用访问，以触发外部流程甚至其他系统。权限管理则负责整个过程中的访问控制。账本和交易进一步地依赖核心的区块链结构、数据库、共识机制等技术；链码则依赖容器、状态机等技术；权限管理利用了已有的PKI体系、数字证书、加解密算法等诸多安全技术。底层由多个节点组成P2P网络，通过gRPC通道进行交互，利用Gossip协议进行同步。层次化结构提高了架构的可扩展和可插拔性，方便开发者以模块为单位进行开发。

 

对于常见的公有区块链，用户只需要将交易通过服务接口直接发送到区块链网络中，网络中的对等节点负责完成所有的共识和处理过程。对于联盟链场景，要更多地考虑权限管理相关的功能需求。比如哪些身份可以向网络中发送交易？哪些交易可以发送到网络中？超级账本Fabric根据交易过程中不同环节的功能，在逻辑上将节点角色解耦为Endorser和Committer，让不同类型节点可以处理不同类型的工作负载。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps43.jpg) 

 

各个组件的主要功能如下：

·客户端(App)：客户端应用使用SDK来跟Fabric网络打交道。首先，客户端从CA获取合法的身份证书来加入网络内的应用通道。发起正式交易前，需要先构造交易提案(Proposal)提交给Endorser进行背书(通过EndorserClient提供的ProcessProposal(ctxcontext.Context，signedProp*pb.SignedProposal)(*pb.ProposalResponse，error)接口)；客户端收集到足够(背书策略决定)的背书支持后可以利用背书构造一个合法的交易请求，发给Orderer进行排序(通过BroadcastClient提供的Send(env*cb.Envelope)error接口)处理。客户端还可以通过事件机制来监听网络中消息，获知交易是否被成功接收。命令行客户端的主要实现代码在peer/chaincode目录下。

·Endorser节点：主要提供ProcessProposal(ctxcontext.Context，signedProp*pb.Signed-Proposal)(*pb.ProposalResponse，error)方法(代码在core/endorser/endorser.go文件)供客户端调用，完成对交易提案的背书(目前主要是签名)处理。收到来自客户端的交易提案后，首先进行合法性和ACL权限检查，检查通过则模拟运行交易，对交易导致的状态变化(以读写集形式记录，包括所读状态的键和版本，所写状态的键值)进行背书并返回结果给客户端。注意网络中可以只有部分节点担任Endorser角色。主要代码在core/endorser目录下。

·Orderer：仅负责排序。为网络中所有合法交易进行全局排序，并将一批排序后的交易组合生成区块结构。Orderer一般不需要跟账本和交易内容直接打交道。主要实现代码在orderer目录下。对外主要提供Broadcast(srvab.AtomicBroadcast_BroadcastServer)error和Deliver(srvab.AtomicBroadcast_DeliverServer)error两个RPC方法(代码在orderer/server.go文件)。

·Committer节点：负责维护区块链和账本结构(包括状态DB、历史DB、索引DB等)。该节点会定期地从Orderer获取排序后的批量交易区块结构，对这些交易进行落盘前的最终检查(包括交易消息结构、签名完整性、是否重复、读写集合版本是否匹配等)。检查通过后执行合法的交易，将结果写入账本，同时构造新的区块，更新区块中BlockMetadata[2](TRANSACTIONS_FILTER)记录交易是否合法等信息。同一个物理节点可以仅作为Committer角色运行，也可以同时担任Endorser和Committer这两种角色。主要实现代码在core/committer目录下；

·CA：负责网络中所有证书的管理(分发、撤销等)，实现标准的PKI架构。主要代码在单独的fabric-ca项目中。CA在签发证书后，自身不参与网络中的交易过程。

 

 

·网络层：面向系统管理人员。实现P2P网络，提供底层构建区块链网络的基本能力，包括代表不同角色的节点和服务。

·共识机制和权限管理：面向联盟和组织的管理人员。基于网络层的连通，实现共识机制和权限管理，提供分布式账本的基础。

·业务层：面向业务应用开发人员。基于分布式账本，支持链码、交易等跟业务相关的功能模块，提供更高一层的应用开发支持。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps44.png) 

 

**交易**

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps45.png) 

 

**区块**

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps46.png) 

 

 

**链码**

用户链码：单独容器，为上层应用提供支持

系统链码：嵌入系统内，提供系统配置、管理支持

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps47.png) 

 

通道

排序服务上划分的彼此隔离的原子广播渠道，有排序服务进行管理

应用通道：用户交易；系统通道：通道管理

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps48.png) 

账本：区块链+数据库结构

数据库结构：state database：记录最新世界状态；History database：各个状态的历史变化记录；Index database：索引数据库，从Hash、编号索引到区块，从ID索引到交易

区块链通过文件系统进行存储。状态数据库支持LevelDB，CouchDB；历史数据库和索引数据库则主要支持LevelDB

/var/hyperledger/production/ledgersData   

账本：chains/chains    索引：chains/index     状态：stateLeveldb     历史：historyLeveldb

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps49.png) 

 

gRPC通信

客户端与Peer节点间通信，客户端、Peer节点与Order节点通信，链码容器与Peer节点之间，多个Peer节点之间

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps50.png) 

 

Envelope结构：

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps51.png) 

 

消息结构SignedPeoposal

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps52.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps53.png) 

 

链码容器和Peer节点之间的操作

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps54.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps55.png) 

 

 

配置系统链码：配置管理系统链码，链外可以调用

JoinChain申请加入链码 ；GetConfigBlock:获取节点在通道上的配置；UpdateConfigBlock：更新通道上的配置；GetChannles:获取节点加入的通道列表。

背书管理系统链码：对链码提案进行签名，返回给客户端；背书策略管理。

生命周期系统链码：链码的生命周期安装、部署、升级、权限管理、获取信息等环节

Install 将用户链码进行打包，放入节点的文件系统

Deploy 链码被部署和实例化，生成链码容器。

Upgrade 对本地文件进行替换升级链码

GetCCInfo 获取链码信息

GetCCData 获取指定链码的完整数据

GetChaincodes 返回通道上所有的链码信息，包括已安装和已实例化

查询系统链码：账本和链的信息查询

GetChainInfo      获取区块信息，包括高度值，当前区块Hash值

GetBlockByNumber 给定高度，返回对应区块数据

 GetBlockByHash   给定Hash，返回对应区块

GetTransactionByID 根据交易ID，返回交易数据

GetBlockByTxID     根据交易ID，返回包含交易的区块的数据

验证系统链码：对从Order节点接受的交易进行写入前的在验证

   验证的过程:对从Order中获取的交易信息的结构进行解析，对交易结构格式校验；检查交易的读集合中元素的版本与本地账本的版本是否一直；

检查是否带有合法的背书信息；

 

**链码操作**

键查询，范围查询，组合查询，只读查询，只读历史查询，

每一个请求都由智能合约读取的读集和写入的写集多版本构造而成

每一条请求都包含提交该请求的节点的签名证书，提交到背书或者排序服务节点

对等节点请求事务的验证通过背书策略来完成

添加块之前检查版本，确保被读取的资产的状态在链代码执行之后没有改变

一个通道的账本包含区块的生成配置策略，访问控制列表  

Core/Chaincode

生成Chaincode镜像，支持对chaincode调用和查询

Package files: [ccproviderimpl]()、[chaincode_support]()、[chaincodeexec]()、[exectransaction]()、[handler]() 

[chaincode_support]()：通过对VMC驱动的调用来支持对chaincode容器的管理，包括启动容器、注册、执行合约

启动

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps56.jpg) 

注册

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps57.jpg) 

执行合约

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps58.jpg) 



![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps59.jpg) 

andler: Chaincode容器启动后，会发送注册消息让peer节点创建一个Handler结构，并进入主循环响应消息。peer侧创建一个状态机来维护对应chaincode各种消息的响应，利用before和after等触发条件

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps60.png) 

 

在shim包中提供了Chaincode与账本结构进行交互的中间层

ChaincodeStub :chaincode中代码通过该结构提供的方法来修改账本状态

Handler：chaincode一侧使用状态机来跟踪shim事件

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps61.jpg) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps62.png) 

 

channel

命令：create、join、fetch、list

调用上述命令时会先调用InitCmdFactory,初始化前需要EndorserClient,OrdererClient

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps63.jpg) 

初始化完成后通过调用Cmd方法

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps64.jpg) 

Create:创建一个新的Channel。根据指定的配置交易文件路径或默认配置，创建Envelope结构，发送给Orderer，并将所指定的创建通道中的初始区块写到本地文件chainID.block。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps65.jpg) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps66.png) 

 

fetch:获取指定通道的初始区块。从Orderer获取指定通道的初始区块，写到本地文件chainID.block。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps67.jpg) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps68.jpg) 

join:让peer加入某个通道。读取本地的block文件，生成一个cscc的JoinChain交易spec序列号,进一步封装为一个ChaincodeInvocationSpec,创建CONFIG类型的Proposal并签名，通过EndorsorClient发送给p

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps69.jpg) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps70.jpg) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps71.jpg) 

list:列出peer所加入的所有通道。生成一个cscc的GetChannels交易序列，进一步封装为一个ChaincodeInvocationSpec,创建ENDORSER_TRANSACTION类型的Proposal，并签名，通过EndorserClient发送给peer。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps72.jpg) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps73.jpg) 

BroadcastClient:*endorser**向**orderer**发送交易的**BroadcastClient**接口和功能实现。*

**智能合约的特点**：

1. 无状态，事件驱动的代码

2. 智能合约可以操作账本状态，账本状态记录着与业务相关重要数据

3. 应用程序通过向区块链网络发送交易来调用智能合约

4. 多个智能合约时，应用程序通过名称、版本号来确定调用哪个智能合约

**使用SDK进行智能合约的一次完整调用过程****:**

I. 从CA获取合法的身份证书

II. 构造合法的交易Proposal后提交Endorser进行背书

III. 收集足够多的背书后，构造合法交易请求发送给order节点进行排序

IV. 监听事件，确保交易已经写入账本

实现码链的条件：

a. 实现Chaincode接口

**type** Chaincode **interface** {

​     Init(stub ChaincodeStubInterface) pb.Response //链码收到实例化或升级的交易时调用

​     Invoke(stub ChaincodeStubInterface) pb.Response//链码收到调用或查询的交易调用

}

b. 链码的必要结构

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps74.jpg) 

SDK与链码交互过程

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps75.png) 

 

响应的chaincodeMessage结构如下

message ChaincodeMessage {

enum Type {

UNDEFINED = 0;

REGISTER = 1;REGISTERED = 2;INIT = 3;READY = 4;TRANSACTION = 5;COMPLETED = 6;ERROR = 7;

GET_STATE = 8;PUT_STATE = 9;DEL_STATE = 10;INVOKE_CHAINCODE = 11;RESPONSE = 13;

GET_STATE_BY_RANGE = 14;GET_QUERY_RESULT = 15;QUERY_STATE_NEXT = 16;QUERY_STATE_CLOSE = 17;

KEEPALIVE = 18;GET_HISTORY_FOR_KEY = 19;

}

Type type = 1; google.protobuf.Timestamp timestamp = 2;bytes payload = 3;

string txid = 4;SignedProposal proposal = 5;ChaincodeEvent chaincode_event = 6;}

 

shim.ChaincodeSubInterface提供的四种API，包括账本交互、交易信息、参数读取、其他，如下：

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps76.png) 

记账节点的账本数据是基于文件系统存储的， 每个链的账本数据存储在不同的目录下。 只有属于

某个链， 才会存在以这个链的通道命名的账本目录。

账本数据存储目录:**fileSystemPath:** /var/hyperledger/production

**配置信息**

配置文件

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps77.png) 

peer从环境变量中读取信息时，是以CORE开头，如peer.id  ->  CORE_PEER_ID

 

配置管理工具

ctyptogen:网络中的组织结构和身份文件

configtxgen:configtx.yaml 生成通道相关配置

configtxlator:对网络中配置进行编，解码，并计算配置更新量

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps78.png) 

Chaincode中CA配置信息

MSP：加密机制，协议，用户身份验证

可以自定义身份，身份认证规则，签名规则和验证规则

**配置MSP需要设置**：在peer和order节点上做配置，启动peer和order节点中的身份验证，并对加入channel中的节点分别签名和验证

1.指定MSP的名称，即MSP的唯一标识

2.身份验证和签名验证需要的参数：

a.一个自签名（X.509）证书列表构成的根

b.把一个表述为中间CAs的X.509证书列表作为提供者的证书验证；这些证书应该由信任根证书中的一个证书进行验证【CAs是可选参数】

c.一个X.509证书列表和授信的根证书集的证书路径作为MSP管理员，这些证书的所有者被授权请求对MSP配置进行更改【如：根CA，中间CAs】

d.这个MSP的有效成员应该包含在X.509证书列表中，这是一个可选参数，如在多个组织中使用相同的根CA和中间CAs，并为组织留一个Organization和Util

e.证书撤销列表集[CRLs certificate revocation lists]中的每一个列表都应列出MSP证书颁发机构[CA/CAs]；

f.一个自签署的证书列表构成了授信TLS证书的TLS根

g.提供者考虑一个表示为中间TLS CAs的X.509根证书列表，这些证书应该由授信的TLS根证书中的一个证书进行认证

为满足以上条件，需要使用MSP实例的有效身份

1>以证书的形式存在，并具有可验证的证书路径，以及完全信任的证书的根

2>不包括在任何证书撤销列表中[CRLs]

3>在证书结构的OU中列出MSP配置的一个或多个组织单元

除了验证相关的参数之外，为了使MSP能够在实例化的节点上能够签名和验证，还需要做出如下设置：

a>用于在节点上签署的签名键[目前仅包含ECDSA]

b>节点的X.509证书，在MSP验证下是有效的

**证书永远不会过期，只有将其加入到证书撤销列表中才能撤销它们；当前还没有针对TLS证书的强制撤销的操作

设置MSP中peer节点和order节点

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps79.png) 

 

·admincerts： 组织管理员的身份验证证书， 被根证书签名。

·cacerts： 组织的根证书， 同ca目录下文件。

·tlscacerts： 用于TLS的CA证书， 自签名

·signcerts： 验证本节点签名的证书， 被组织根证书签名。

·keystore： 本节点的身份私钥， 用来签名。

·tls： 存放tls相关的证书和私钥

·ca.crt： 组织的根证书。

·server.crt： 验证本节点签名的证书， 被组织根证书签名。

·server.key： 本节点的身份私钥， 用来签名。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps80.png) 

节点的MSP的标识符作为参数的值提供给peer的localMspId和orderer的LocalMSPID。这些变量可以通过当前环境使用CORE前缀赋给peer（例如：CORE_PEER_LOCALMSPID）以及使用ORDERER前缀赋给orderer（例如：ORDERER_GENERAL_LOCALMSPID）来重写。注意，对于orderer设置，需要生成，并向orderer提供系统channel的创世区块。

验证MSP的参数：MSP标识符，信任证书的根，中间证书CA和管理证书、OU规范和CRLs

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps81.png) 

Ledger账本：即状态切换(state transitions),是有序且不可篡改的。状态切换是有参与方提交的chaincode调用transactions的结果。每个事务都将产生一组资产键值对，这些键值对作为创建，更新而提交给账本。

Ledger有区块链组成，是有序且不可篡改的记录，以及保存当前状态的state database。每一个channel中都会存在一个账本。每一个peer都会维护它作为其中成员的每一个channel中本地拷贝的Ledger。

chain：是一个事务日志，由hash链接的区块结构，其中每个区块都包含了N个事务的序列。区块Header包含了该区块的Hash，以及上一个区块头的Hash。这样账本的交易都是按照顺序进行排列的，并以密码方式链接在一起。

state database：该账本的当前数据状态表示chain事务日志中包含的所有键的最新值。由于当前状态表示channel所知道的所有最新值，所以有时成为“world state”。在chaincode调用对当前状态数据执行操作事务时，为了使这些chaincode交互非常有效，所有键的最新值都存储在一个状态数据库中。状态数据库只是一个索引视图到chain的事务日志中，因此可以在任何时候从chain中重新生成它。在事务接受之前，状态数据库将自动恢复。

Transaction flow:在高层业务逻辑处理上，事务处理流程是由应用程序客户端发起的事务协议，该协议最终发送到指定的背书节点。背书节点会验证客户端的签名，并执行一个chaincode函数来摹拟事物。最终返回客户端chaincode的结果，读集和写集中的键-值 -版本 的集合，同时还会附带有一个背书签名。

客户端将背书组合成一个事务payload,并将其广播至一个order service，order service 为当前channel上所有peers排序服务和生成区块。

configtxgen配置交易生成器：可以配合cryptogen生成的组织结构身份文件使用，离线生成跟通道有关的配置信息，功能有：生成启动Orderer需要的初始化区块，并支持检查区块内容；生成创建应用通道需要的配置交易，并支持检查交易内容；生成锚点Peer的更新配置交易。

用来生成通道相关的配置交易和系统通道初始化块等进行简单的查看。对于给定的两个配置（common.Config结构） ， configtxlater还可以比对它们的不同， 计算出更新到新配置时的更新量（common.ConfigUpdate结构） 。

configtx.yaml配置文件一般包括四个部分： Profiles、 Organizations、 Orderer和Application。

·Profiles： 一系列通道配置模板， 包括Orderer系统通道模板和应用通道类型模板；

·Organizations： 一系列组织结构定义， 被其他部分引用；

·Orderer： Orderer系统通道相关配置， 包括Orderer服务配置和参与Ordering服务的可用组织信息；

·Application： 应用通道相关配置， 主要包括参与应用网络的可用组织信息。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps82.png) 

 

 

1.链码初始化：读取配置[chaincodeName,{ProviderName,HashFamily,Level,Ephemeral}],与客户端建立gRPC连接与Peer节点建立链接[FSM]：七个过程 Register Registered Eatablished Ready  Init  Completed Transaction

2.Peer节点背书提案：链码的安装、初始化、升级、调用、查询；Peer节点加入和退出通道

   背书过程：检查提案的合法性以及相关权限；模拟提案，启动链码容器，对wordState的最新版本进行快照，基于它的执行链码，将结果写在读写集中；对提案内容和读写集合进行签名，返回消息；【检查提案，构造提案完整内容，对提案签名返回】

校验过程：GetProposal  <getheader></getheader>   validateCommonHeader   checkSignatureFromCreator[shdr.Creator, signedProp.Signature, signedProp.ProposalBytes, chdr.ChannelId]   CheckProposalTxID  validateChaincodeProposalMessage    IsSysCCAndNotInvokableExternal

模拟提案：simulateProposal(ctx context.Context, chainID string, txid string, signedProp *pb.SignedProposal, prop *pb.Proposal, cid *pb.ChaincodeID, txsim ledger.TxSimulator)

提案签名：endorseProposal(ctx, chainID, txid, signedProp, prop, res, simulationResult, ccevent, hdrExt.PayloadVisibility, hdrExt.ChaincodeId, txsim, cd)

network_setup.sh是一件测试脚本，该脚本启动5个docker容器，其中4个容器运行peer节点和1个容器运行orderer节点，它组成一个Fabric集群。另外还有一个cli容器用于执行创建channel、加入channel、安装和执行chaincode等操作

path:fabric-1.1.0/examples/e2e_cli/scripts/script.sh

该脚本会一步一步的完成创建通道，将其他节点加入通道，更新锚节点，创建ChainCode，初始化账户，查询，转账，再次查询等链上代码的各个操作都可以自动化实现。

使用crypto-config.yaml来生成组织证书和私钥

使用configtx.yaml来创建名为genesis.block的block【构建创世区块】

/bin/configtxgen -profile ExampleOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

使用configtx.yaml生成channel源文件

生成channel变量前，需要预先设置channel环境变量

 export CHANNEL_NAME=mychannel

 # this file contains the definitions for our sample channel

 ../bin/configtxgen -profile ExampleChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

./bin/configtxgen -profile ExampleChannel -outputCreateChannelTx ./channel-artifacts/example_channel.tx -channelID channel01

生成channel下节点集合认证文件

./bin/configtxgen -profile ExampleChannel -outputAnchorPeersUpdate ./channel-artifacts/DEMOMSPanchors.tx -channelID channel01 -asOrg DemoMSP

先启动order节点

根据docker-compose-base.yaml 生成 docker-compose-order.yaml文件

docker-compose -f docker-compose-orderer.yaml up -d  来启动order节点

启动orgMSP peer节点

根据docker-compose-cli.yaml 和 辅助文件 docker-compose-base.yaml和peer-bas.yaml生成docker-compose-org.yaml文件

docker-compose -f docker-compose-org.yaml up -d

创建并加入channel

创建：peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA

加入：peer channel join -b example.block

**kafka集群配置要点**

用kafka集群配置的原因也很简单，为orderer共识及排序服务提供足够的容错空间，当我们向peer节点提交Transaction的时候，peer节点会得到或返回（基于SDK）一个读写集结果，该结果会发送给orderer节点进行共识和排序，此时如果orderer节点突然down掉，致使请求服务失效而引发的数据丢失等问题，且目前的sdk对orderer发送的Transaction的回调会占用极长的时间，当大批量数据导入的时候该回调可认为不可用。固此，在部署生产环境时，需要对orderer进行容错处理，而所谓的容错即搭建一个orderer节点集群，该集群会依赖于kafka和zookeeper。

 通过crypto-config.yaml文件生成所需的order节点的验证文件

**Orderer节点排序过程原理**

Orderer节点在接入kafka集群后，利用kafka共识机制，来进行排序和打包区块功能。

Kafka 的一个重要特性就是支持数据的过期删除，数据可以在 Broker 上保留一段时间。

leader控制器负责管理整个集群的操作，包括分区的分配、失败节点的检测等。一个 partition 只能出现在一个 broker 节点上即leader。

消息都是以主题 Topic 的方式组织在一起， Topic 也可以理解成传统数据库里的表，或者文件系统里的一个目录。

一个topic由 broker 上的一个或者多个Partition 分区组成。在 Kafka 中数据是以 Log 的方式存储，一个 partition 就是一个单独的 Log。消息通过追加的方式写入日志文件，读取的时候则是从头开始按照顺序读取。注意，一个topic通常都是由多个partition组成的，每个partition内部保证消息的顺序行，partition之间是不保证顺序的。如果你想要 kafka 中的数据按照时间的先后顺序进行存储，那么可以设置分区数为 1。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps83.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps84.png) 

 

**Orderer Service Node结构**

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps85.png) 

 

MultiChain：封装了chain的相关接口。chain目前的三种方式：Systemchain[必须]，chain[kafka/solo]等方式。

Orderer模块下Broadcast的过程：

main->server->broadcast->msgprocess

->chain

->registrar

->standardchannel

整个过程分为三步：

解析消息：判断接受的消息是配置消息还是普通消息；若为创建消息则交由SystemChannel进行处理

处理消息：普通消息进行检查后进行入队操作，之后交由kafka处理；配置消息合并配置消息，入队操作

响应：

对消息进行过滤检查时，过滤器会在创建ChainSupport结构时候初始化：

应用通道：msgprocessor/standardchannel.go的CreateStandardChannelFilters(filterSupport channelconfig.Resources) *RuleSet

规则包括EmptyRejectRule,NewExpirationRejectRule,NewSizeFilter,NewSigFilter

Orderer 模块下Deliver的过程：

客户端从Orderer节点上获取排序后的数据

客户端链码的安装过程：[SignedProposal]客户端封装安装消息；Peer节点处理请求InitCmdFactory,install [ChaincodeDeploymentSpec]

客户端链码实例化：[SignedProposal]创建一个 SignedTX   [ChaincodeDeploymentSpec]

客户端链码调用：[SignedProposal]   [ChaincodeSpec]；注意 invoke 是异步操作，invoke 成功只能保证交易已经进入 Orderer 进行排序，但无法保证最终写到账本中。

客户端链码查询：query，与invoke不同，不需要创建SingedTx

创建通道：sendCreateChainTransaction()、ORDERER_TRANSACTION  -> chainID + ".block"

加入通道：CSCC.JoinChain,利用 ChaincodeSpec 构造一个 ChaincodeInvocationSpec 结构,通过配置系统链码完成初始化，进而收到gossip消息

更新节点配置：

交易模拟的代码实现过程

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps86.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps87.png) 

 

系统链码指的是Fabric Peer中负责系统配置、 背书、 验证等平台功能的逻辑， 运行在Peer进程内。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps88.png) 

 

构建镜像条件：需要链码，背书节点的TLS根证书，各个平台的DockerFIle配置

构建镜像过程的操作是：镜像的名称由链码的名称和版本号构成；构建镜像时fabric信息和TLS根证书信息。

确定镜像时候存在的条件：组织，链码名称，版本信息相同时，就可以确定是同名镜像，不会重新构建。

构建镜像的过程：依据链码语言，在源码的基础上添加背书节点的SDK和其他的一些依赖，再根据相应语言的编译环境编译为二进制的文件放入镜像文件中。

 

使用 docker inspect 可以查看构建的标签信息

获取Docker的标签信息列表

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps89.png) 

 

链码容器继承背书节点的环境变量列表

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps90.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps91.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps92.png)  

交易号的构成是通过：TxID = HASH(Nonce + Creator) 两者共同构成的部署

genession.block解析

data,header,metedate等信息，data中存有channel_group，其中定义了组织及组织的策略信息

组织锚节点Org1MSPanchors.tx信息

交易在经过batch压缩后，将消息发送给kafka集群，集群中对交易进行hash后，放入到partition中，partition中的顺序是一定的，partition间的顺序不固定，此时可以通过controller对顺序进行某种操作，确定顺序

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps93.png) 

 

 

 

 

 

 

 

 