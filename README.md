# AirPlay-Receiver-on-Android
在Android中实现Airplay的接收端

# 项目介绍
目前是移动互联网的时代，小屏幕已经占领了我们生活的大部分时间，然而在家庭内的另一个屏幕就是电视屏幕，如果让两个屏幕连接起来，
几年前已经成为了一个热门的话题（最近似乎不是很热了），但是如何占领家庭内部的市场仍是一个重点和热点，如近期出现的智能路由器。
在媒体共享方面，出现了DLNA和apple的Airplay两个比较好的东西。
### DLNA
DLNA全开放，目前各个部分已经有很好的实现，如Cling和CyberGarage等实现了DLNA的协议库。
### airplay
aiplay是apple的东西，比较封闭，仅仅用于ihone（ipad）与apple自己tv：apple-tv之间进行交互，而且不同的IOS版本可能还会有变化，如果你用apple的官方接口应该没有问题，但是网络上对airplay的抓包和分析，不同的版本可能还不一样，在国内，虽然iphone是在移动端的比例很大，而appletv在国内始终用户很少，国内大部分是android的智能电视，或者普通的电视加上一个androidd盒子。因此iphone和android之间实现airplay就很有必要了。目前国内的盒子几乎全部支持了，如小米盒子、funbox等。这些支持airplay的盒子有没有申请apple的东西，谁也不知道，如果apple要搞一下，估计许多都不能用了，或者不停的针对apple的版本进行破解、升级。

此项目基于几年前我所在的项目组的研究，但是随着ios版本的升级，原来的研究有些不管用了，基于个人爱好，开始了此项目。
实现iphone6 IOS8.4上的图片或者视频推送到我的android手机nubia上，国内的一些app store上的应用，如：优酷等客户端等是支持airplay的。

## 服务注册
airplay的服务发现是与M_DNS 和 DNS_SD协议的，目前开源的java实现为jmdns，百度搜索即可。苹果视频和图片的推送服务名称为._airplay._tcp.local，airplay注册服务的时候需要用到。

## 具体的协议分析
  简单的来说需要你的android 实现一个httpserver，然后apple设备（手机，pad）作为client将内容推送到你的server上，然后server（android）设备根据不同的内容进行显示，client（苹果）设备可以对推送的内容进行控制：推送下一张图片、视频的暂停、seek和推送结束等。

### 对于图片  
  首先你会受到一个http get /server-info的请求   
  
  然后收到一个http post /reverse请求  
  
  最后就有收到 http put /photo请求，请求的httpbody中就含有实际的jpeg格式的图片二进制文件信息，在android中你decode就可以直接显示。  
  
  具体的日志如下：  
  
airplay  incoming HTTP  method = GET; target = /server-info;   

airplay  incoming HTTP  method = POST; target = /reverse;   

airplay  incoming HTTP  method = PUT; target = /photo;    

  airplay推送图片的时候，会有一个缓存的操作，即：将缓存图片一并推送过来，这样可以较快的进行下一张图片的显示，提高用户体现。具体的第一次推送put /photo的时候，会推送三种图片，然后当你在apple客户端滑动显示图片的时候，会推送当前显示的一样和下一张的cache。具体的日志如下：
  


推送第一张图片  

08-25 13:32:48.508  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = 78A1BB2D-5488-4372-95EA-FF32737B563C 缓存 左边  

08-25 13:32:48.523  14608-15497/com.guo.duoduo.airplayreceiver D/MyLineParse
08-25 13:32:48.568  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = F6BC486E-821B-4D74-B257-80AF280C6E5C 缓存 右边  

08-25 13:32:48.725  14608-15497/com.guo.duoduo.airplayreceiver D/MyLineParser﹕
08-25 13:32:48.752  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay display image; assetKey = X-Apple-AssetKey: 8B792485-B6B6-4CF4-91D9-A14734E9E790 显示 当前    



右滑动   

08-25 13:35:46.201  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay display cached image, assetKey = 78A1BB2D-5488-4372-95EA-FF32737B563C 原来左边变为当前显示  

08-25 13:35:46.345  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = B6711879-7539-4980-8213-98FA76FDD11A  缓存左边  


右滑动   

08-25 13:38:01.586  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay display cached image, assetKey = B6711879-7539-4980-8213-98FA76FDD11A 上一个的左边变为当前  


08-25 13:38:01.642  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = 15FE410D-84D3-4ED8-A741-673CD2DFD0F4 缓存左边  


左滑动  

08-25 13:42:04.883  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay display cached image, assetKey = 78A1BB2D-5488-4372-95EA-FF32737B563C  

08-25 13:42:04.980  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = 8B792485-B6B6-4CF4-91D9-A14734E9E790   


左滑动  

08-25 13:47:06.468  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay display cached image, assetKey = 8B792485-B6B6-4CF4-91D9-A14734E9E790  

08-25 13:47:06.542  14608-15497/com.guo.duoduo.airplayreceiver D/WebServiceHandler﹕ airplay cached image, assetKey = F6BC486E-821B-4D74-B257-80AF280C6E5C  

  而且每一个图片都对应这个一个 唯一的id：assetKey.
  
  结束推送的时候：
  
  airplay  incoming HTTP  method = POST; target = /stop 
### 视频推送
  视频推送是通过优酷客户端进行的。
  ##未完待续
