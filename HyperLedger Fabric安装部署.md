**1. 新环境初始化**

安装ssh   sudo apt-get install ssh

更改源   sudo vim /etc/apt/sources.list

:%s/us./cn./g

sudo apt-get update

安装python  sudo apt-get install python3.5

sudo apt-get install python-pip

sudo -H python -m pip install --upgrade pip  

安装代理网络 sudo pip install shadowsocks

touch shadowsocks.json

{

"server" : "45.78.13.106",

"server_port" : 44,

"localPort" : 1080, 

"password" : "xxxxx",

"method" : "rc4-md5",

"remarks" : ""

}

后台运行sslocal

sudo mv shadowsocks.json /etc/

nohup sslocal -c /etc/shadowsocks.json &

sudo apt-get install polipo

\#sudo vim /etc/polipo/config

​    socksParentProxy = "127.0.0.1:1080"

​    socksProxyType = socks5

​    chunkHighMark = 50331648

​    objectHighMark = 16384

​    serverMaxSlots = 64

​    serverSlots = 16

​    serverSlots1 = 32

​    proxyAddress = "0.0.0.0"

proxyPort = 8123

\# sudo /etc/init.d/polipo restart

配置http代理

\# export http_proxy='http://127.0.0.1:8123'

\# export https_proxy='http://127.0.0.1:8123'

 

验证

\# wget www.google.com

安装Docker

curl -sSL <https://get.docker.com> | sh

完成后如下图所示：版本为18.05

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps1.png) 

 

DAOCloud加速器

curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://85f9f519.m.daocloud.io

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps2.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps3.png) 

 

sudo vim /etc/default/docker

DOCKER_OPTS="-s=aufs -r=true --api-cors-header='*' -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --registry-mirror=curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://85f9f519.m.daocloud.io"

加入用户到docker中

sudo usermod -aG docker fabric

启动docker服务

service docker restart

下载docker-compose

sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

 

sudo chmod +x /usr/local/bin/docker-compose

 

安装环境所需软件

go安装

[sudo apt-get install golang   |  卸载go  sudo apt-get purge golang-go]

\#wget https://storage.googleapis.com/golang/go1.10.1.linux-amd64.tar.gz

\#sudo tar -xzf go1.10.1.linux-amd64.tar.gz -C /usr/local      

设置环境变量

vim  ~/.bashrc

export GOROOT=/usr/local/go

export GOPATH=$HOME/gopath

export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

export DOCKER_HOST=tcp://127.0.0.1:4243

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps4.png) 

 

source .bashrc

**2.下载源码和镜像**

下载fabric代码

mkdir -p $GOPATH/src/github.com/hyperledger

cd $GOPATH/src/github.com/hyperledger

git clone https://github.com/hyperledger/fabric.git

 

下载fabric-ca代码

cd $GOPATH/src/github.com/hyperledger

git clone https://github.com/hyperledger/fabric-ca.git

 

下载案例快速搭建

cd  $GOPATH/src/github.com/hyperledger

git  clone https://github.com/hyperledger/fabric-samples.git

cd  fabric-samples/first-network

使用curl命令进行下载，之前确保外网可访问

sudo  curl **-**sSL https:**//**goo**.**gl**/**6wtTN5 **|** bash **-**s 1.1**.**0

如果上述操作有问题，可使用脚本直接下载镜像

 

 

下载后如图所示

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps5.png) 

 

镜像下载完成后，会产生bin文件，下面有工具，将此工具的绝对路径添加到环境变量中

export PATH=下载位置的绝对路径/bin:$PATH  

[export PATH=/home/ubuntu/gopath/src/github.com/hyperledger/fabric-samples/bin:$PATH]

**3.快速搭建过程**

cd ../first-network/

使用脚本byfn.sh生成初始环境信息

./byfn.sh -h  查看脚本信息

**3.1生成初始化信息**

执行脚本./byfn.sh  up会根据Docker Compose配置文件docker-compose-cli.yaml启动超级账本网络， 还会执行scripts/script.sh脚本安装和实例化链码， 并且执行简单链码调用和查询操作。

执行script.sh脚本时，会执行：创建通道，加入通道，更新通道信息，安装链码，实例化链码，链码调用，链码查询

创建通道

byfn.sh  generate  [根据crypto-config.yaml生成peer,order的MSP证书]

​                  [根据configtx.yaml生成创世区块]

a. 使用cryptogen工具生成证书

命令：

 

 

b. 生成排序节点的初始区块

命令：

此处调用common/tools/configtxgen/main.go:main()->msp/configbuilder.go：getMspConfig()->common/tools/configtxgen/main.go:doOutputBlock()

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps6.png) 

c. 生成通道交易配置文件

命令：

此处调用：common/tools/configtxgen/main.go:main()->common/tools/configtxgen/main.go:doOutputChannelCreateTx()->msp/configbuilder.go:getMspConfig()->common/tools/configtxgen/main.go:doOutputChannelCreateTx()

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps7.png) 

 

d. 生成Org1MSP/Org2MSP锚节点配置

命令：

此处调用：common/tools/configtxgen/main.go:main()->doOutputAnchorPeersUpdate()

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps8.png) 

 

4.2 启动网络

启动网络时，由于docker权限的影响，所以此时运行时

sudo ./byfn.sh up.

开始执行脚本后，首先创建网络 net_byfn

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps9.png) 

 

后续过程都基于fabric-1.1.0\examples\e2e_cli\scripts\script.sh基本调用

**【安装过程中可以通过docker ps -a 查看容器中程序运行；安装****chaincode****后，通过****docker image****查看链码生成的镜像】**

启动完成后开始创建channel

条件：初始化时生成的根证书，证书，私钥，MSP等信息，及通道配置交易文件创建通道

命令：

此处主要调用peer/channel/channel.go的InitCmdFactory(),初始化Endorser和Orderer节点

channel创建完成后，Org1的peer1、peer2与Org2的peer1、peer2加入mychannel

条件：利用上面的根证书、证书、私钥、MSP等信息，加入channel

命令：

此处调用的是peer/channel/join.go的executeJoin(cf *ChannelCmdFactory)方法

更新Org1和Org2的anchor peer

命令：

peer/main.go:channel.Cmd()->peer/channel/channel.go:updateCmd()->update()

给Org1、Org2的peer0安装链码

命令：

调用：peer/chaincode/chaincode.go:Cmd()->installCmd()->genChaincodeDeploymentSpec()->getChaincodeSpec()->checkChaincodeCmdParams()

emcc和vscc没有定义，使用默认配置进行安装

对Org2的peer0进行链码初始化

命令：

调用：peer/chaincode/chaincode.go:Cmd()->instantiateCmd()->genChaincodeDeploymentSpec()->getChaincodeSpec()->checkChaincodeCmdParams()

emcc和vscc没有定义，使用默认配置进行初始化

查询Org1的peer0信息

通过初始化Org2的peer0，查询Org1的peer0获取链码结果

命令：

进行交易

命令：

网络测试

通过安装Org2.peer1链码，在Org2.peer1上查询a的值，得到正确的结果，说明安装链码时交易信息自动同步，网络正常

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps10.png) 

 

4.3关闭网络

可以看到关闭peer节点，排序服务节点容器，链码容器，删除自动生成的镜像文件。

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps11.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps12.png) 

 

运行./byfn.sh up 后的docker显示信息与./byfn.sh down 后的docker显示信息

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps13.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps14.png) 

 

**3.2逐步搭建网络**

1. 使用cryptogen生成MSP证书

 cryptogen generate --config=./crypto-config.yaml

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps15.png) 

 

2. 生成排序服务的创世区块

export "FABRIC_CFG_PATH=$PWD"

configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps16.png) 

 

3. 生成通道配置创世区块

export CHANNEL_NAME=onechannel

configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps17.png) 

 

4. 定义组织AnchorPeers

configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps18.png) 

 

启动网络

sudo docker-compose -f docker-compose-cli.yaml up -d

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps19.png) 

 

创建并加入网络通道

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps20.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps21.png) 

 

 

sudo docker exec -it cli bash

export “CHANNEL_NAME=onechannel”

peer channel create -o orderer.example.com:7050 -c ‘CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile ’/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps22.png) 

 

peer0.org1.example.com加入通道

由于默认为peer0.org1.example.com因此环境变量不需要更改

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps23.png) 

 

peer channel join -b onechannel.block

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps24.png) 

 

peer channel list

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps25.png) 

 

peer0.org2等加入通道

执行如下命令加入

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps26.png) 

 

安装和实例化链码

Peer节点peer0.org1.example.com安装链码

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps27.png) 

安装的过程中如果遇到找不到文件的问题，可先搜索下example的文件位置，然后替换既可以了

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps28.png) 

 

重启终端，进入peer0容器，查看链码是否安装

docker exec peer0.org1.example.com ls /var/hyperledger/production/chaincodes

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps29.png) 

 

其余Peer节点peer节点与peer0类似

链码的初始化

任意节点上都可

命令：

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps30.png) 

实例化后检查是否成功，使用docker ps -a ，dev-peer1.org2.example.com-mycc-1.0

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps31.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps32.png) 

 

peer chaincode query -C $CHANNEL_NAME -nmycc -c '{"Args":["query","a"]}'

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps33.png) 

 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps34.png) 

 

执行链码操作

| peer chaincode invoke -oorderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -c'{"Args":["invoke","a","b","10"]}' |
| ------------------------------------------------------------ |
| 执行查询peer chaincode query -C $CHANNEL_NAME -nmycc -c '{"Args":["query","a"]}' |

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps35.png) 

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps36.png) 

 

**4.FQA**

生成cryptogen等工具的方法

a.在fabric目录下 make cryptogen进行生成

如过遇到“vendor/github.com/miekg/pkcs11/pkcs11.go:26:18: fatal error: ltdl.h: No such file or directory”，则是系统需要安装

sudo apt-get install libtool libltdl7 libltdl-dev

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps37.png) 

 

b.直接在fabric/common/tools/cryptogen执行go build

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps38.png) 

 

生成configtxgen工具，同上

![img](file:///C:\Users\htkj2016\AppData\Local\Temp\ksohtml3028\wps39.png) 

 

configtx.yaml文件中的一些说明

Profile结构中需要包括

Application:若无，创建通道会失败

Consortiums:若无，启动服务节点会失败

-inspectBlock :检查和输出创世区块的内容

-inspectChannelCreateTx :检查和输出创始区块的内容

-outputAnchorPeersUpdate:创建在configtx.yaml中指定的AnchorPeers

 

**5.KN**

tox概述

通用的虚拟环境管理和测试命令行工具。tox能够让我们在虚拟环境上定义出多套相互隔离的python环境（tox是openstack社区最基本的测试工具，比如python程序的兼容性，UT等）。它的目标是提供最先进的自动化打包，测试和发布功能。

优点：

1. 作为持续集成服务器的前端，减少测试工作所需时间

2. 检查软件包能否在不同python版本环境下正常安装

3. 在不同的环境中运行测试代码

travis和Jenkins

两者都是持续集成服务，区别是travis采用的是yaml脚本，简洁，明了，与github绑定，提供测试环境，可在代码变动后持续集成和打包。

 