# 应用层SIP协议通讯流程

SIP协议是处理现场海康威视业务发现时发现的一种视频协议，用来处理视频级联传输数据。

## SIP协议是干啥的


## SIP协议传输端口
SIP协议传输数据默认使用的7100端口，特殊的地方在于源和目的都使用的是7100端口。

## 海康威视视频级联处理方法
级联是海康威视两个或者多个流媒体服务器之间传输视频数据的一种方法。可以是两个流媒体服务器之间的级联，也可以是多个流媒体服务器的级联。**上下级流媒体服务器都需要配置对应的设备信息才能正常工作，配置的内容主要包括IP、传输端口和所属域等内容**。

### 通讯流程
1. 下级服务器是主动发起方，上级流媒体服务器是被动方，级联视频数据的传输都是下级发送到上级。
2. 下级流媒体通过UDP7100向上级发送带有【REGISTER】标志的数据包，上级流媒体服务器会做出相应反馈
3. 上级流媒体通过UDP7100向下级发送带有【SUBSCRIBE】标志的数据包，下级流媒体服务器会做出相应反馈
4. 上级流媒体通过UDP7100向下级发送带有【INVITE】标志的数据包，**此数据中包含上级流媒体即将开放的UDP端口来接收数据**，下级流媒体会通过UDP7100发送带有【trying】标志数据包，同时会对于【INVITE】数据包进行应答，**此数据包含下级流媒体服务器即将使用的UDP发送数据端口**。
5. 上级流媒体会对【trying】数据包进行应答。
6. 通过步骤4协商出来的端口，下级流媒体会主动发送数据到上级流媒体，**此数据包协议被识别为RTP**
7. 通过步骤4协商出来的端口，**两个端口分别加一，作为新的传输确认端口**，每当下级流媒体服务器发送一定数据后，下级流媒体会通过传输确认端口发送一个数据包，上级流媒体也会对于数据包做出应答，**此数据包协议被识别为RTCP**。
8. 传输数据后，上级流媒体通过UDP7100向下级流媒体发送带有【BYE】标志的数据包，下级流媒体会对于此标志数据包进行应答，整个传输流程结束

### SIP数据包中包含的方法

#### REGISTER方法
参考链接:[https://blog.csdn.net/yunmao2882/article/details/86736194](https://blog.csdn.net/yunmao2882/article/details/86736194)

在海康威视级联通讯中，REGISTER方法主要体现在**下级流媒体服务器**首先要通过此方法在**上级流媒体服务器**进行注册。

注册

	REGISTER sip:37280000000000000000@10.69.80.187:7100 SIP/2.0
	Via: SIP/2.0/UDP 10.69.91.7:7100;rport;branch=z9hG4bK4226189633
	From: <sip:37280600000000000000@10.69.91.7:7100>;tag=4063080099
	To: <sip:37280600000000000000@10.69.91.7:7100>
	Call-ID: 59498979
	CSeq: 1 REGISTER
	Contact: <sip:37280600000000000000@10.69.91.7:7100>
	Max-Forwards: 70
	User-Agent: NCG V3.3.6.686194
	Expires: 7200
	Content-Length: 0

应答

	SIP/2.0 200 OK
	Via: SIP/2.0/UDP 10.69.91.7:7100;rport;branch=z9hG4bK4226189633
	From: <sip:37280600000000000000@10.69.91.7:7100>;tag=4063080099
	To: <sip:37280600000000000000@10.69.91.7:7100>;tag=1036776414
	Call-ID: 59498979
	CSeq: 1 REGISTER
	Contact: <sip:37280600000000000000@10.69.91.7:7100>
	User-Agent: NCG V3.3.6.624213
	Date: 2019-12-25T16:45:44
	Expires: 7200
	Content-Length: 0
	

#### SUBSCRIBE
在海康威视级联通讯中，SUBSCRIBE方法主要是**上级流媒体服务器**通过此方法定订阅**下级流媒体**服务器。

请求
	
	SUBSCRIBE sip:37280600000000000000@10.69.91.7:7100 SIP/2.0
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport;branch=z9hG4bK1845960006
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=1353738433
	To: <sip:37280600000000000000@10.69.91.7:7100>
	Call-ID: 3513458978
	CSeq: 20 SUBSCRIBE
	Contact: <sip:37280000000000000000@10.69.80.187:7100>
	Content-Type: Application/MANSCDP+xml
	Max-Forwards: 70
	User-Agent: NCG V3.3.6.624213
	Expires: 1800
	Event: Catalog;id=945357
	Content-Length:   129

	<?xml version="1.0"?>
	<Query>
	<CmdType>Catalog</CmdType>
	<SN>945357</SN>
	<DeviceID>37280600000000000000</DeviceID>
	</Query>

应答

	SIP/2.0 200 OK
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport=7100;branch=z9hG4bK1845960006
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=1353738433
	To: <sip:37280600000000000000@10.69.91.7:7100>;tag=1652678304
	Call-ID: 3513458978
	CSeq: 20 SUBSCRIBE
	Contact: <sip:37280600000000000000@10.69.91.7:7100>
	Event: Catalog;id=945357
	User-Agent: NCG V3.3.6.686194
	Expires: 1800	
	Content-Length: 0
	
经过上述两步，上下级服务器相应对方的状态即可确定。

#### INVITE

INVITE状态数据包在级联数据中主要用来上级流媒体服务器获取下级流媒体服务器的视频数据。

请求数据包，此数据包由上级流媒体服务器发出，其中**m=video 5798**中5798为上级流媒体服务器接收UDP数据包的端口。
	
	INVITE sip:37280601411319001158@10.69.91.7:7100 SIP/2.0
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport;branch=z9hG4bK3351538455
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=3817019866
	To: <sip:37280601411319001158@10.69.91.7:7100>
	Call-ID: 918077789
	CSeq: 20 INVITE
	Contact: <sip:37280000000000000000@10.69.80.187:7100>
	Content-Type: Application/SDP
	Max-Forwards: 70
	User-Agent: NCG V3.3.6.624213
	Subject: 37280601411319001158:0,37280000000000000000:0
	Content-Length:   252

	v=0
	o=37280601411319001158 0 0 IN IP4 10.69.80.187
	s=Play
	c=IN IP4 10.69.80.187
	t=0 0
	m=video 5798 RTP/AVP 96 97 98
	a=rtpmap:96 PS/90000
	a=rtpmap:97 MPEG4/90000
	a=rtpmap:98 H264/90000
	a=recvonly
	a=streamMode:MAIN
	a=filesize:-1
	y=0999999999
	
应答数据包，其中**m=video 5930 RTP/AVP 96**中5930端口为下级流媒体服务器发送数据使用的UDP端口。

	SIP/2.0 200 OK
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport=7100;branch=z9hG4bK3351538455
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=3817019866
	To: <sip:37280601411319001158@10.69.91.7:7100>;tag=1001466019
	Call-ID: 918077789
	CSeq: 20 INVITE
	Contact: <sip:37280600000000000000@10.69.91.7:7100>
	Content-Type: Application/SDP
	User-Agent: NCG V3.3.6.686194
	Content-Length:   176

	v=0
	o=37280601411319001158 0 0 IN IP4 10.69.80.187
	s=Play
	c=IN IP4 10.69.91.7
	t=0 0
	m=video 5930 RTP/AVP 96
	a=rtpmap:96 PS/90000
	a=sendonly
	a=filesize:-1
	y=0100007399

#### BYE
上级级联服务器发出的断开连接数据包

	BYE sip:37280600000000000000@10.69.91.7:7100 SIP/2.0
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport;branch=z9hG4bK1009780211
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=3817019866
	To: <sip:37280601411319001158@10.69.91.7:7100>;tag=1001466019
	Call-ID: 918077789
	CSeq: 21 BYE
	Contact: <sip:37280000000000000000@10.69.80.187:7100>
	Max-Forwards: 70
	User-Agent: NCG V3.3.6.624213
	Content-Length: 0

下级服务器发出的应答数据包
 
	SIP/2.0 200 OK
	Via: SIP/2.0/UDP 10.69.80.187:7100;rport=7100;branch=z9hG4bK349726410
	From: <sip:37280000000000000000@10.69.80.187:7100>;tag=3069112529
	To: <sip:37280601411319001113@10.69.91.7:7100>;tag=3328635930
	Call-ID: 3357339688
	CSeq: 21 BYE
	User-Agent: NCG V3.3.6.686194
	Content-Length: 0

如果出现此数据包，那么协商出来的传输数据的连接就可以断开。

## RTP通讯流程
RTP数据包主要是用来传输视频数据，使用的端口是通过上述INVITE数据包流程所协商出来的数据包。


## RTCP通讯流程
RTCP数据包主要作用是当上下级服务器传输视频结束以后，通过RTCP数据包将传输数据包的传输情况告诉对方，来确认数据的发送状态。RTCP使用的端口是通过INVITE协商出来的端口，源和目的分别加一作为新的传输端口。


