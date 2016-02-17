# 背景
时间：1年前的某日

坐标：深圳

在一个平常的不能再平常的周末下午，几个小伙伴聚在了一起，一起畅想“万物互联”的物联网未来。小伙伴中有硬件开发者、嵌入式开发者、软件开发者；有互联网公司的全栈工程师、也有核电厂的工控系统维护者、还有路由器厂商的wifi协议开发者。我们发现，世面上没有开源且可商用的物联网平台或系统。这里的可商用，不是搭建几个demo把硬件连上网、app操作两下这么简单！

有以下几点都是必须着重考虑的：

* 必须有完备的硬件、嵌入式、云端一体的协议及架构设计
* 能够实现真正的硬件智能化，能够基于数据学习并自主工作
* 必须有很高的性能、稳定性及扩展性
* 必须能够适应成千上万种不同资源的硬件设备，从PC到手机、从计算资源极其有限的单片机到网络带宽极其有限的控制器
* 必须能适应不同的网络场景，包括有线、wifi、3g/4g、gprs等
* 必须有很可靠的安全性
* 需要尽可能降低研发和生产成本

在媒体和科技工作者都抱着物联网是未来的观点并翘首观望时，我们决定做点什么，而不是当看客！这个平常的不能再平常的周末下午，也许对我们不太平凡。

我们决定启动全套可商用物联网系统的设计和研发，并在不久的将来，全部开源。

![](http://ruizeng.net/content/images/2015/11/design-iot.jpg)

于是大家利用业余时间，开始了协议设计及系统设计，将项目慢慢启动了起来。几个月后，第一个商用版本的研发成功完成。这期间，好几个小伙伴辞去了工作，全职进行研发。我们在没有融资、没有资源的情况下一路走到现在，其中辛酸就不多言了。谨以此文记录我们在系统设计和研发中的走过的路，以飨同样是物联网爱好者的你。

## 整体设计
一个物联网系统涉及硬件、软件、云端、app各个环节，必须从整体进行顶层设计，只倚重某个单一的环节进行设计的系统都不具备良好的适用性和扩展性。我们在设计时为了避免这种情况，使系统能够适应最广泛的物联网场景（甚至包括工业场景），每次的架构设计讨论都是所有团队成员参与。大体的系统架构如下：

![](http://ruizeng.net/content/images/2015/11/----.jpg)

## 协议
在一个物联网系统中，协议是串通上下层的关键纽带。在物联网系统中，我们将协议分为两大层：通信层和业务层。

![](http://ruizeng.net/content/images/2015/11/protocol.jpg)

通信层基本上是传统互联网的网络基础设施，负责将数据在物联网系统节点中的传输

业务层分为两层，底层是负责物联网场景下数据交换格式的规范，上层是物联网场景需要具体传输的业务数据规范。

通信层互联网基础架构目前已经非常成熟且通用，但是业务层协议目前还是种类繁多。可以确定的一点是，最终能在物联网应用中称霸的协议，一定也像互联网时代的TCP/IP一样是开放的、免费的。目前符合此特性并使用比较多的有XMPP、MQTT、COAP等。关于具体的对比，可以参考我之前的另一篇文章《[物联网通信协议介绍](http://ruizeng.net/wu-lian-wang-tong-xin-xie-yi/)》。

文中总结如下：

互联网中使用较多的HTTP、websocket以及XMPP等协议，在设计时都是根据互联网应用场景设计的，虽然很多厂商把他们应用在物联网系统中，但是必然会水土不服，这些协议的通病就是根本无法适用物联网设备的多样性，无法适用很多物联网设备对低功耗、低成本的需求，难以在极低资源的物联网设备中运用。

COAP协议针对资源受限的嵌入式设备设计，但由于很多物联网设备隐藏在局域网内部，COAP设备作为服务器无法被外部设备寻址，在ipv6没有普及之前，coap只能适用于局域网内部（如wifi）通信，这也很大限制了它的适用范围。

MQTT在协议设计时就考虑到不同设备的计算性能的差异，所以所有的协议都是采用二进制格式编解码，并且编解码格式都非常易于开发和实现。最小的数据包只有2个字节，对于低功耗低速网络也有很好的适应性。有非常完善的QOS机制，根据业务场景可以选择最多一次、至少一次、刚好一次三种消息送达模式。运行在TCP协议之上，同时支持TLS（TCP+SSL）协议，并且由于所有数据通信都经过云端，安全性得到了较好地保障。

我们最终选择基于MQTT来作为业务传输层主要协议。但是MQTT协议本身的设计是针对开放设备，对于可商用的物联网系统不得不保证设备的安全性和完善的授权机制。所以我们在实现MQTT协议时进行了一些定制和限制。

在业务层的上层（business层），目前的物联网系统都是各自针对自己的业务场景设计协议规范。有没有可能根据物联网场景统一业务数据的规范呢？我们认为是可行的，并且也是必要的。如果把通信协议比作声音，光有通信协议，任何人之间还是无法交流。只有统一语言，大家才能顺畅沟通。所以我们抽象出物联网节点中传感器和执行器的业务场景，并设计出含有物联网业务数据语义的业务层协议。目前已经将业务层协议开源，希望对广大爱好者和从业者带来一定参考价值。

协议的设计文档已经在[GitHub](https://github.com/PandoCloud/pando-protocol)开源。

## 云端平台
互联网时代的用户上网终端主要是PC和手机等设备，可以想象，物联网时代，上网终端会呈多样化、海量化趋势。保守估计每人拥有数十套联网设备，数据规模必然也是几何倍增长。所以物联网云端平台注定是一个大规模的海量分布式系统。

目前很多爱好者或者厂商通过搭建简单的web系统（如php、nodejs、python实现的web接口）可以实现设备的联网，但是可以想象，在真正的商用场景中，稳定性、性能、扩展性都必然遭受冲击，无法应对。

在进行技术选型和架构设计时，我们也综合考虑以上因素进行设计和实现：

* 采用go语言作为主要开发语言。go语言有着简洁的语法，并且能够很方便地进行高并发程序的开发，在高性能云计算系统的开发中有着得天独厚的优势。
* 采用microservice分布式架构。microservice架构能够构建出更稳定、扩展性更好的分布式系统，也是目前分布式系统中最流行的架构方式。
* 使用docker降低运维成本。docker能够方便地对系统就行升级和出错回滚，保证了系统发布时的稳定性。
* 对外接口采用REST风格进行设计。REST风格的接口便于升级和兼容，并拥有非常易于理解的语义，降低开发者的学习门槛。
* 多副本部署。任何服务模块我们都保证同时至少有两个运行实例，并根据服务发现机制自动进行负载和调度，以增加系统可用性。
大体的云端架构如下图

![](http://ruizeng.net/content/images/2015/11/arch-001.jpg)

目前我们的系统已经发布到0.8.0版本，后续会在安装和运维的便捷性上进行优化，这里是[GitHub](https://github.com/PandoCloud/pando-cloud)地址。

## 嵌入式
物联网硬件的嵌入式软件除了传统部分，必须加入联网逻辑以及传感器、控制器的管理。为了提高开发效率、方便复用，我们设计并开发了轻量级的物联网嵌入式开发框架，并对物联网业务进行了抽象，以便移植到不同的硬件平台。我们希望做到的是，在不需要更改任何业务层代码的情况下，一个物联网嵌入式应用可以在不同的硬件平台运行。

![](http://ruizeng.net/content/images/2015/11/iot.jpg)

当前很多大企业（华为、惠普、google等）都纷纷推出了物联网操作系统，后续物联网领域会出现多种操作系统共存的局面。不同的操作系统能运行的最低系统资源以及具体应用场景都不尽相同，但我们相信，物联网的上层业务是通用的，这也是我们设计物联网嵌入式开发框架的原因。

项目已经在[GitHub](https://github.com/PandoCloud/pando-embeded-framework)开源。

## 安全
近些日子，各种厂商的物联网设备纷纷传出被黑的消息。从TCL到特斯拉，黑客都成功实现破解和随意操控。和互联网时代一样，安全在物联网目前的早期阶段注定是容易被忽略的问题。为此我们也在设计系统时也没有掉以轻心：

所有接入层通信都采用tls进行加密，包括对app、业务服务器的开放接口。
用户、设备关键信息进行加密保存
针对设备有完善的用户鉴权机制
针对互联网安全场景的其他安全措施
安全不是一朝一夕的事情，需要从系统开始构建时就考虑，并不断完善安全手段和规则。

## 开发板
为了降低物联网硬件的开发成本，我们基于esp8266设计了物联网开发板Tisan，并在Tisan实现了我们的嵌入式开发框架及物联网协议。开发板相关的代码已经全部[开源](https://github.com/tisan-kit)，目前在淘宝进行众筹。

为什么推出开发板？我们认为这是一种互相学习、交流及沉淀技术的工具，希望更多的爱好者能一起做出好产品。

以开源之名
光阴荏苒，白驹过隙。一路走来，我们执着地将所有设计慢慢付诸实现，为未来的物联网技术贡献自己的力量。物联网技术涉及的方向众多，我们的力量毕竟是有限的，这也是我们从一开始就以开源形式开发项目的原因。

Now,  we are calling for contributors！

如果你对物联网和开源技术有着和我们一样的执着和爱好，欢迎关注或加入我们的开源项目。后续我们会逐渐开源发布所有的系统设计和项目。我们希望更多的小伙伴一起为项目做贡献，无论是提交issue，还是贡献代码。

欢迎加入我们的物联网平台与协议讨论群一起交流学习：488074716

Hope we are not alone.