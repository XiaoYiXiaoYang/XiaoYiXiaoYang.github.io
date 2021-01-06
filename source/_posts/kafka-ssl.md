---
title: kafka SSL加密通信相关
date: 2020-10-13 22:36:00
tags: [Kafka]
categories: [学习笔记]
---

翻译 http://kafka.apache.org/documentation/

<!--more--> 

### 安全性

#### 安全概述

在0.9.0.0版中，Kafka社区添加了许多功能，这些功能可以单独使用或一起使用，从而提高了Kafka集群的安全性。当前支持以下安全措施：

1. 使用SSL或SASL对来自客户端（生产者和消费者），其他broker和工具的的连接进行身份验证。Kafka支持以下SASL机制：
   - SASL / GSSAPI（Kerberos）-从0.9.0.0版本开始
   - SASL / PLAIN-从版本0.10.0.0开始
   - SASL / SCRAM-SHA-256和SASL / SCRAM-SHA-512-从版本0.10.2.0开始
   - SASL / OAUTHBEARER-从2.0版开始
2. 从broker到ZooKeeper的连接的身份验证
3. 使用SSL加密在节点和客户端之间，集群broker之间或broker和工具之间传输的数据（请注意，启用SSL时性能会降低，其大小取决于CPU类型和JVM实现。）
4. 客户授权的读/写操作
5. 授权是可插入的，并支持与外部授权服务的集成

值得注意的是，安全性是可选的-支持不安全的集群，以及经过身份验证，未经身份验证，加密和未加密的客户端的混合。以下指南说明了如何在客户端和broker中配置和使用安全功能。



#### 使用SSL的加密和身份认证

Apache Kafka允许客户端使用SSL进行流量加密和身份验证。默认情况下，SSL是禁用的，但可以根据需要打开。以下各段详细说明了如何设置自己的PKI基础结构，如何使用它来创建证书以及如何配置Kafka以使用它们。



##### 1.为每个kafka broker生成SSL秘钥和证书



部署一个或多个具有SSL支持的brokers的第一步是为每个服务器生成一个公共/私有密钥对。由于Kafka希望所有密钥和证书都存储在密钥库中，因此我们将使用Java的keytool命令来执行此任务。该工具支持两种不同的密钥库格式，即现已弃用的Java特定的jks格式以及PKCS12。PKCS12是Java版本9的默认格式，为确保无论使用的Java版本如何都使用此格式，所有以下命令均明确指定PKCS12格式。



```
keytool -keystore {keystorefile} -alias localhost -validity {validity} -genkey -keyalg RSA -storetype pkcs12
```



您需要在上面的命令中指定两个参数：

1. keystorefile：存储此broker的密钥（以及以后的证书）的密钥库文件。密钥库文件包含此broker的私钥和公钥，因此需要确保其安全。理想情况下，此步骤是在将使用该密钥的Kafka broker上运行的，因为该密钥永远不会被传输/离开它打算用于的服务器。

2. validity：密钥的有效时间（天）。请注意，这与证书的有效期不同，有效期将在“签署证书”中确定。您可以使用同一密钥来请求多个证书：如果密钥的有效期为10年，但是您的CA仅会签署有效期为一年的证书，则随着时间的推移，您可以将同一密钥与10个证书一起使用。

   

为了获得可以与刚刚创建的私钥一起使用的证书，需要创建一个证书签名请求。由受信任的CA签名后，此签名请求将生成实际证书，然后可以将其安装在密钥库中并用于身份验证。
要生成证书签名请求，请对到目前为止创建的所有服务器密钥库运行以下命令。

```
keytool -keystore server.keystore.jks -alias localhost -validity {validity} -genkey -keyalg RSA -destkeystoretype pkcs12 -ext SAN=DNS:{FQDN},IP:{IPADDRESS1}
```

该命令假定您要向证书添加主机名信息，如果不是这种情况，则可以省略extension参数`-ext SAN=DNS:{FQDN},IP:{IPADDRESS1}`。请参阅下面的详细信息。



##### 主机名验证

启用主机名验证是指根据要连接的服务器提供的证书中的属性，对该服务器的实际主机名或IP地址进行检查，以确保您确实连接至正确的服务器。

进行此检查的主要原因是为了防止中间人攻击。对于Kafka，默认情况下已禁用此检查很长时间，但自Kafka 2.0.0起，默认情况下已为客户端连接以及broker间连接启用服务器的主机名验证。

可以通过设置`ssl.endpoint.identification.algorithm`为空字符串来禁用服务器主机名验证。

对于动态配置的broker listeners，可以使用`kafka-configs.sh`以下命令禁用主机名验证：

```
 bin/kafka-configs.sh --bootstrap-server localhost:9093 --entity-type brokers --entity-name 0 --alter --add-config "listener.name.internal.ssl.endpoint.identification.algorithm="
```

**注意：**

正常情况下，除了作为最快的“禁用主机名”方法，然后承诺“有更多时间以后修复它”之外，没有充分的理由禁用主机名验证！

在正确的时间完成正确的主机名验证并不困难，但是一旦集群启动并运行，则变得更加困难-帮自己一个忙，现在就开始吧！

如果启用了主机名验证，则客户端将根据以下两个字段之一验证服务器的标准域名（FQDN）或IP地址：

1. Common Name（CN）
2. Subject Alternative Name（SAN）



尽管Kafka选中了这两个字段，但自2000年以来，不赞成将CN段用于主机名验证 ，因此应尽可能避免使用。此外，SAN字段更加灵活，允许在证书中声明多个DNS和IP条目。

另一个优点是，如果将SAN字段用于主机名验证，则可以将common name设置为更有意义的值以进行授权。由于我们需要将SAN字段包含在签名证书中，因此将在生成签名请求时指定它。也可以在生成密钥对时指定它，但不会自动将其复制到签名请求中。

要添加SAN字段` -ext SAN=DNS:{FQDN},IP:{IPADDRESS} `，请在keytool命令后附加以下参数：

```
keytool -keystore server.keystore.jks -alias localhost -validity {validity} -genkey -keyalg RSA -destkeystoretype pkcs12 -ext SAN=DNS:{FQDN},IP:{IPADDRESS1}
```



#### 2.创建自己的CA



完成此步骤后，集群中的每台计算机都具有一个公钥/私钥对，可以将其用于加密通信和证书签名请求，这是创建证书的基础。要添加身份验证功能，此签名请求需要由受信任的机构签名，该机构将在此步骤中创建。



证书颁发机构（CA）负责签署证书。CA就像颁发护照的政府一样，政府会在每本护照上盖章（签名），以使护照变得难以伪造。其他政府则对邮票进行核实，以确保护照是真实的。同样，CA对证书进行签名，而加密技术则保证了签名的证书在计算上难以伪造。因此，只要CA是真实可信的授权机构，客户就可以有力保证他们正在连接到真正的计算机。



对于本指南，我们将成为我们自己的证书颁发机构。在公司环境中建立生产集群时，这些证书通常由整个公司范围内受信任的公司CA签名。请参阅生产中的常见陷阱，以了解此案例的一些注意事项。



由于OpenSSL中的错误，x509模块不会将请求的扩展字段从CSR复制到最终证书中。由于我们希望SAN扩展出现在我们的证书中以启用主机名验证，因此我们将使用*CA*模块。这需要在生成CA密钥对之前进行一些其他配置。

将以下清单保存到一个名为openssl-ca.cnf的文件中，并根据需要调整有效性和通用属性的值。

```
HOME            = .
RANDFILE        = $ENV::HOME/.rnd

####################################################################
[ ca ]
default_ca    = CA_default      # The default ca section

[ CA_default ]

base_dir      = .
certificate   = $base_dir/cacert.pem   # The CA certifcate
private_key   = $base_dir/cakey.pem    # The CA private key
new_certs_dir = $base_dir              # Location for new certs after signing
database      = $base_dir/index.txt    # Database index file
serial        = $base_dir/serial.txt   # The current serial number

default_days     = 1000         # How long to certify for
default_crl_days = 30           # How long before next CRL
default_md       = sha256       # Use public key default MD
preserve         = no           # Keep passed DN ordering

x509_extensions = ca_extensions # The extensions to add to the cert

email_in_dn     = no            # Don't concat the email in the DN
copy_extensions = copy          # Required to copy SANs from CSR to cert

####################################################################
[ req ]
default_bits       = 4096
default_keyfile    = cakey.pem
distinguished_name = ca_distinguished_name
x509_extensions    = ca_extensions
string_mask        = utf8only

####################################################################
[ ca_distinguished_name ]
countryName         = Country Name (2 letter code)
countryName_default = DE

stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Test Province

localityName                = Locality Name (eg, city)
localityName_default        = Test Town

organizationName            = Organization Name (eg, company)
organizationName_default    = Test Company

organizationalUnitName         = Organizational Unit (eg, division)
organizationalUnitName_default = Test Unit

commonName         = Common Name (e.g. server FQDN or YOUR name)
commonName_default = Test Name

emailAddress         = Email Address
emailAddress_default = test@test.com

####################################################################
[ ca_extensions ]

subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always, issuer
basicConstraints       = critical, CA:true
keyUsage               = keyCertSign, cRLSign

####################################################################
[ signing_policy ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

####################################################################
[ signing_req ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, keyEncipherment
```

然后创建一个数据库和序列号文件，这些文件将用于跟踪使用此CA签名的证书。这两个都是简单的文本文件，与CA密钥位于同一目录中。

```
echo 01 > serial.txt
touch index.txt
```



完成这些步骤后，您现在即可生成您的CA，该CA将在以后用于签署证书。

```
openssl req -x509 -config openssl-ca.cnf -newkey rsa:4096 -sha256 -nodes -out cacert.pem -outform PEM
```

CA只是由其自身签名的公钥/私钥对和证书，仅用于签署其他证书。
此密钥对应该保持非常安全，如果有人可以访问它，他们可以创建并签署将由您的基础结构信任的证书，这意味着当连接到信任此CA的任何服务时，他们将可以模拟任何人。



下一步是将生成的CA添加到“客户端的信任库”中，以便客户端可以信任此CA：

```bash
keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert
```



**注意：** 如果通过在Kafka broker配置中将ssl.client.auth设置为“ requested”或“ required”，则kafka broker配置为要求客户端身份验证， 则您还必须为Kafka代理提供信任库，并且该库应具有全部客户端密钥签名的CA证书。



```bash
 keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert
```

与步骤1中存储每台计算机自己的身份的密钥库不同，客户端的信任库存储客户端应信任的所有证书。将证书导入一个人的信任库也意味着信任该证书签名的所有证书。

与上面的类推类似，信任政府（CA）也意味着信任它已颁发的所有护照（证书）。此属性称为信任链，在大型Kafka群集上部署SSL时特别有用。您可以使用单个CA对群集中的所有证书进行签名，并使所有计算机共享信任该CA的同一信任库。这样，所有计算机都可以对所有其他计算机进行身份验证。



### 签署证书

然后与CA签署

```bash
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out {server certificate} -infiles {certificate signing request}
```



最后，您需要将CA的证书和已签名的证书都导入到密钥库中：

```bash
keytool -keystore {keystore} -alias CARoot -import -file {CA certificate}
 keytool -keystore {keystore} -alias localhost -import -file cert-signed
```





参数的定义如下：

1. keystore：密钥库的位置
2. CA certificate：CA的证书
3. certificate signing request：使用服务器密钥创建的csr
4. server certificate：将服务器的签名证书写入的文件



这将为您提供一个名为*truststore.jks的信任库*-对于所有客户端和broker而言，该*信任库*可以相同，并且不包含任何敏感信息，因此无需保护此信息。

此外，每个节点将有一个*server.keystore.jks*文件，其中包含该节点的密钥，证书和您的CA证书， 有关如何使用这些文件的信息，请参阅配置kafka broker和 配置kafka 客户端

要获得有关此主题的一些工具帮助，请查看easyRSA项目，该项目具有大量的脚本来帮助执行这些步骤。



### 生产中常见陷阱

上面的段落显示了创建自己的CA并将其用于为集群签名证书的过程。尽管对于沙盒，开发，测试和类似系统非常有用，但这通常不是在公司环境中为生产集群创建证书的正确过程。企业通常将运行自己的CA，用户可以发送要与此CA签署的CSR，这样做的好处是，用户无需负责确保CA的安全以及每个人都可以信任的中央权威。但是，它也剥夺了对用户签署证书的过程的大量控制。

1. 扩展秘钥使用

   证书可能包含扩展字段，用于控制可以**[使用](https://tools.ietf.org/html/rfc5280#section-4.2.1.12)**证书的目的。如果此字段为空，则不会限制使用，但是如果在此处指定了任何用法，则有效的SSL实现必须强制执行这些用法。

   Kafka的相关用法是：

   - Client authentication  客户端认证
   - Server authentication 服务端认证

   Kafka broker需要同时允许这两种用法，

   因为对于集群内通信，每个broker都会充当其他broker的客户端和服务器。

   公司CA具有用于Web服务器的签名配置文件并将其用于Kafka的情况并不少见，该配置文件仅包含serverAuth的使用值，并且会导致SSL握手失败。

   

2. 中间证书

   出于安全原因，**中间证书公司**和CA通常保持脱机状态。为了启用日常使用，创建了所谓的中间CA，然后将它们用于签署最终证书。将证书导入由中间CA签名的密钥库中时，有必要提供完整的信任链，直到根CA。这可以通过简单地完成cating的证书文件荷兰国际集团合并为一个证书文件，然后使用密钥工具导入此。

3. 复制扩展字段的失败

   CA运营商通常不愿从CSR复制和请求扩展字段，而是倾向于自己指定这些字段，因为这会使恶意方更难获得具有潜在误导性或欺诈性价值的证书。建议仔细检查已签名的证书，这些证书是否包含所有请求的SAN字段以启用正确的主机名验证。可以使用以下命令将证书详细信息打印到控制台，该证书详细信息应与最初请求的内容进行比较：

   ```bash
    openssl x509 -in certificate.crt -text -noout
   ```



### 配置kafka broker



Kafka Brokers支持侦听多个端口上的连接。我们需要在server.properties中配置以下属性，该属性必须具有一个或多个逗号分隔的值：

```
listeners
```



如果没有为broker之间的通信启用SSL（请参见下面的启用方法），则PLAINTEXT和SSL端口都是必需的。

```text
            listeners=PLAINTEXT://host.name:port,SSL://host.name:port
```



broker端需要以下SSL配置

```text
            ssl.keystore.location=/var/private/ssl/server.keystore.jks
ssl.keystore.password=test1234
ssl.key.password=test1234
            ssl.truststore.location=/var/private/ssl/server.truststore.jks
ssl.truststore.password=test1234
```

注意：ssl.truststore.password在技术上是可选的，但强烈建议使用。如果未设置密码，则仍然可以访问信任库，但是将禁用完整性检查。值得考虑的可选设置：

1. ssl.client.auth = none（“ required” =>需要客户端身份验证，“ requested” =>请求客户端身份验证，并且没有证书的客户端仍然可以连接。不鼓励使用“ requested”，因为它提供了错误的含义安全性和配置错误的客户端仍将成功连接。）
2. ssl.cipher.suites（可选）。密码套件是认证，加密，MAC和密钥交换算法的命名组合，用于协商使用TLS或SSL网络协议的网络连接的安全设置。（默认为空列表）
3. ssl.enabled.protocols = TLSv1.2，TLSv1.1，TLSv1（列出要从客户端接受的SSL协议。请注意，不建议使用SSL，而建议使用TLS，不建议在生产中使用SSL）
4. ssl.keystore.type = JKS
5. ssl.truststore.type = JKS
6. ssl.secure.random.implementation = SHA1PRNG



如果要启用SSL进行broker之间的通信，请将以下内容添加到server.properties文件中（默认为PLAINTEXT）

```text
security.inter.broker.protocol=SSL
```

由于某些国家/地区的进口法规，Oracle实施限制了默认情况下可用的加密算法的强度。如果需要更强大的算法（例如，具有256位密钥的AES），则必须获取[JCE无限强度管辖权策略文件](http://www.oracle.com/technetwork/java/javase/downloads/index.html)并将其安装在JDK / JRE中。有关更多信息，请参见 [JCA提供者文档](https://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html)。



JRE / JDK将具有用于加密操作的默认伪随机数生成器（PRNG），因此不需要配置与一起使用的实现`ssl.secure.random.implementation`。但是，某些实现存在性能问题（值得注意的是，在Linux系统上选择的默认设置`NativePRNG`使用全局锁定）。如果SSL连接的性能成为问题，请考虑明确设置要使用的实现。该`SHA1PRNG`实现是非阻塞的，并且在高负载（产生的消息每秒50 MB，加上每个代理的复制流量）下表现出非常好的性能。

启动broker后，您应该可以在server.log中看到

```text
with addresses: PLAINTEXT -> EndPoint(192.168.64.1,9092,PLAINTEXT),SSL -> EndPoint(192.168.64.1,9093,SSL)
```



要快速检查服务器密钥库和信任库是否正确设置，可以运行以下命令

```
openssl s_client -debug -connect localhost：9093 -tls1
```

（注意：TLSv1应该列在ssl.enabled.protocols下）。
在此命令的输出中，您应该看到服务器的证书：

```text
 -----BEGIN CERTIFICATE-----
{variable sized random bytes}
 -----END CERTIFICATE-----
subject=/C=US/ST=CA/L=Santa Clara/O=org/OU=org/CN=Sriharsha Chintalapani
issuer=/C=US/ST=CA/L=Santa Clara/O=org/OU=org/CN=kafka/emailAddress=test@test.com
```

如果未显示证书或存在任何其他错误消息，则说明密钥库设置不正确。



### 配置Kafka 客户端

仅新的Kafka Producer和Consumer支持SSL，不支持旧的API。对于生产者和使用者，SSL的配置将相同。

如果broker中不需要客户端身份验证，则以下是最小配置示例：

```text
security.protocol=SSL          ssl.truststore.location=/var/private/ssl/client.truststore.jks
ssl.truststore.password=test1234
```

注意：ssl.truststore.password在技术上是可选的，但强烈建议使用。如果未设置密码，则仍然可以访问信任库，但是将禁用完整性检查。

如果需要客户端身份验证，则必须像步骤1一样创建密钥库，并且还必须配置以下内容：

```text
ssl.keystore.location=/var/private/ssl/client.keystore.jks
ssl.keystore.password=test1234
ssl.key.password=test1234
```

根据我们的要求和代理配置，可能还需要其他配置设置：

1. ssl.provider（可选）。用于SSL连接的安全提供程序的名称。默认值为JVM的默认安全提供程序。

2. ssl.cipher.suites（可选）。密码套件是认证，加密，MAC和密钥交换算法的命名组合，用于协商使用TLS或SSL网络协议的网络连接的安全设置。

3. ssl.enabled.protocols = TLSv1.2，TLSv1.1，TLSv1。它应列出在代理方配置的至少一种协议

4. ssl.truststore.type = JKS

5. ssl.keystore.type = JKS

   


使用console-producer和console-consumer的示例：

```bash
kafka-console-producer.sh --bootstrap-server localhost:9093 --topic test --producer.config client-ssl.properties
 
kafka-console-consumer.sh --bootstrap-server localhost:9093 --topic test --consumer.config client-ssl.properties
```