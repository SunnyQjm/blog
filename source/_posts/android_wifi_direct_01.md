---
layout:     post                    # 使用的布局（不需要改）
title:      Android WI-FI Direct Kotlin 浅析（一）         # 标题
subtitle:   基本使用                       #副标题
date:       2018-03-26              # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - Kotlin
---

> ## 简介

- 百度百科：2010年10月，Wi-Fi Alliance（wi-fi联盟）发布Wi-Fi Direct白皮书，白皮书中介绍了有关于这种技术的基本信息、这种技术的特点和这种技术的功能，Wi-Fi Direct标准是指允许无线网络中的设备无需通过无线路由器即可相互连接。与蓝牙技术类似，这种标准允许无线设备以点对点形式互连，而且在传输速度与传输距离方面则比蓝牙有大幅提升

- 与Android开发者而言呢，Google嗅到了WI-FI直连的发展前景，整了一套WI-FI直连的API，在Android4.0+（API >= 14）的设备上均能使用。这给我们进行手机之间的互联操作提供了一个新的实现思路。它可以不需要路由而让两个设备直接相连，但其传输速度可是远远超过蓝牙传输。而且它不需要热点的建立，甚至两个设备不需要在一个局域网下。想象一下，不用建立热点也可以实现像快牙，茄子一类的面对面快传的需求～》《～


> ## 基本使用

- ### 定义一个广播接收器
  > - 每当当前设备与Wi-fi直连相关的状态发生改变，系统便会以广播的形式通知我们，所以让我们定义一个Receiver欢迎它
  > - 在下面定义的Receiver里面我们监听了4个Action，这是最常用的4个，还有其它的Action可以看官方文档，或者，来去撸一波源码

  ``` kotlin
  class WifiDirectReceiver : BroadcastReceiver() {

      /**
       * 写一个便捷的注册方法，动态注册的时候就不用写intentFilter了
       */
      fun registerReceiver(context: Context) {
          val intentFilter = IntentFilter()
          intentFilter.addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION)
          intentFilter.addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)
          intentFilter.addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION)
          intentFilter.addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION)
          context.registerReceiver(this, intentFilter)
      }

      override fun onReceive(context: Context?, intent: Intent?) {
          when (intent?.action) {
          /**
           *当Wifi功能打开或关闭的时候系统会发送 WIFI_P2P_STATE_CHANGED_ACTION 广播
           * Tip: 并不是指是否已经成功连上WI-FI
           */
              WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION -> {
              }
          /**
           * 当前设备的详细信息发生变化的时候，系统会发送 WIFI_P2P_THIS_DEVICE_CHANGED_ACTION 广播
           */
              WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION -> {
              }

          /**
           * 当可连接的对等节点列表发生改变的时候，系统会发送 WIFI_P2P_PEERS_CHANGED_ACTION 广播
           * invoke when the list of peers find, register, lost
           */
              WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION -> {
              }
          /**
           * 当一个连接建立或断开的时候，系统会发送该广播
           * This action received when the connection setup or dis-setup
           */
              WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION -> {
              }
          }
      }
  }
  ```

- ### 接下来在Activity中动态注册
  ```kotlin
  class WifiDirectActivity : AppCompatActivity() {

      private val wifiDirectReceiver = WifiDirectReceiver()

      override fun onStart() {
          super.onStart()
          wifiDirectReceiver.registerReceiver(this)
      }

      override fun onStop() {
          super.onStop()
          unregisterReceiver(wifiDirectReceiver)
      }

      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
      }
  }
  ```
  - 没错，直接动态注册了！！！Android8.0的新特性让Android开发喵几乎一夜之间丧失在AndroidManifest中静态注册的技能（所以为了，考虑兼容，我们就直接动态注册了）
- ### 好，接下来验证一下WIFI_P2P_STATE_CHANGED_ACTION
  - 我们在Receiver里面加点输出（实际开发还是用Log，极不推荐直接输出，这里笔者还是决定偷懒♪(^∇^*)）
    ```kotlin
    /**
     *当Wifi功能打开或关闭的时候系统会发送 WIFI_P2P_STATE_CHANGED_ACTION 广播
     * Tip: 并不是指是否已经成功连上WI-FI
     */
    WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION -> {
        //get the state of current device
        val state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE,
                WifiP2pManager.WIFI_P2P_STATE_DISABLED)
        if (state == WifiP2pManager.WIFI_P2P_STATE_ENABLED) {
            //Wifi p2p enable
            println("wifi enable")
        } else {
            //wifi p2p disEnable
            println("wifi disEnable")
        }
    }
    ```
  - 然后跑到真机上（万能的模拟器然而并不能模拟wifi功能，只能真机实测一波啦），反复开关Wifi功能
    ```bash
    03-26 16:26:08.744 29313-29313/com.j.ming.eupanwifidirect I/System.out: wifi enable
    03-26 16:26:11.098 29313-29313/com.j.ming.eupanwifidirect I/System.out: wifi disEnable
    03-26 16:32:16.376 29313-29313/com.j.ming.eupanwifidirect I/System.out: wifi enable
    ```
  - OK, 得到上面的输出，我们就知道确实，这个ACTION在WI-FI功能开关的时候会触发
- ### WIFI_P2P_THIS_DEVICE_CHANGED_ACTION
  - 大新闻：这个ACTION携带了一团神秘数据，^``^,那就是本设备的设备信息啦～
  - 同样的，我们加一点输出，看一下都包含哪些信息（当然，最直接的是看WifiP2pDevice这个数据）
    ```kotlin
    /**
     * 当前设备的详细信息发生变化的时候，系统会发送 WIFI_P2P_THIS_DEVICE_CHANGED_ACTION 广播
     */
    WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION -> {
        val device: WifiP2pDevice = intent.getParcelableExtra(WifiP2pManager.EXTRA_WIFI_P2P_DEVICE)
        println(device)
    }
    ```
  - 同样真机测试，来调皮的开关一下WI-FI功能（当然，一开始也会打印出设备信息，那就是在这个Receiver被注册的时候）
    ```bash
    03-26 16:43:29.420 30294-30294/com.j.ming.eupanwifidirect I/System.out: wifi enable
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out: Device: 魅蓝 Note3
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  deviceAddress: 2e:57:31:98:45:35
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  primary type: 10-0050F204-5
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  secondary type: null
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  wps: 0
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  grpcapab: 0
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  devcapab: 0
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  status: 3
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  wfdInfo: WFD enabled: falseWFD DeviceInfo: 0
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  WFD CtrlPort: 0
    03-26 16:43:29.436 30294-30294/com.j.ming.eupanwifidirect I/System.out:  WFD MaxThroughput: 0
    ```
    - 可以看到包括设备的名字，本设备的MAC地址，状态balabala
- ### 后面两种ACTION涉及到连接，我们等下讨论

> ## WifiP2pManager 的异种接口
  - requestPeers、requestGroupInfo、discoverPeers、createGroup、removeGroup等等
  - 当然可以直接用，不过为了简介，我们一边用一遍封装出一个管理类

- ### 获得WifiP2pManager对象
  - **下面的操作需要添加以下权限**
    ```xml
    <uses-permission
        android:name="android.permission.ACCESS_WIFI_STATE"
        android:required="true" />
    <uses-permission
        android:name="android.permission.CHANGE_WIFI_STATE"
        android:required="true" />
    ```

  ```Kotlin
  object WifiDirectManager{
      private var manager: WifiP2pManager by Delegates.notNull()
      private var channel: WifiP2pManager.Channel by Delegates.notNull()

      fun init(context: Context): WifiDirectManager{
          //通过获取系统服务的方式获得Manager对象
          manager = context.getSystemService(Context.WIFI_P2P_SERVICE) as WifiP2pManager
          channel = manager.initialize(context, Looper.getMainLooper()){
              //初始化操作成功的回调
          }
          return this
      }
  }
  ```
  - 利用Kotlin 的object关键字，快速创建一个单例对象
  - channel在后续的操作中要用到，这个是manager初始化成功之后返回的渠道对象
  - 上述init方法建议在Application的onCreate中调用，至少在使用前调用
- ### discoverPeers
  - 在上述的WifiDirectManager类里面添加如下方法
    ```kotlin
    /**
     * discover available peer list
     */
    fun discoverPeers(): WifiDirectManager {
        manager.discoverPeers(channel, object : WifiP2pManager.ActionListener{
            override fun onSuccess() {
                println("discover Peers success")
            }

            override fun onFailure(reason: Int) {
                println("discover Peers fail: $reason")
            }
        })
        return this
    }

    /**
     * request the peer list available
     */
    fun requestPeers(): WifiDirectManager {
        manager.requestPeers(channel) { peers ->
            //请求对等节点列表操作成功
            println(peers)
        }
        return this
    }
    ```
  - 在Activity里面拖一个button，在点击事件里面调用这个函数
    ```Kotlin
    btnSendTestBroadcast.setOnClickListener {
        WifiDirectManager.discoverPeers()
    }
    ```
  - 上面的方法，望文生义，就是用来检索附近的可用对等节点列表，然后后面就是这个整套API最坑的一个回调了
  - **WifiP2pManager.ActionListener** 这个回调里面的success和failure**表示的是一个操作是否执行成功，而并不能反映出这个操作的结果**
  - 所以从上面这个回调里我们只能知道，执行discoverPeers这个操作有没有成功，是无法获取到对等节点列表的
  - 那怎么获取检索的结果呢，答案是:
    - 在Receiver里面直接获取（API >= 18）
    - 在Receiver里面调用requestPeers()获取
    -
- ### 两种获取对等节点列表的方式
  > Tip：在调用discoverPeers之后才能获取到对等节点列表

  ```kotlin
  /**
   * 当可连接的对等节点列表发生改变的时候，系统会发送 WIFI_P2P_PEERS_CHANGED_ACTION 广播
   * invoke when the list of peers find, register, lost
   */
  WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION -> {
      //api > 18 have this extra info,
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
          val wifiP2pList: WifiP2pDeviceList = intent.getParcelableExtra(WifiP2pManager.EXTRA_P2P_DEVICE_LIST)
          println(wifiP2pList)
      } else { //if the sdk version lower than 18
          //get WifiP2pDeviceList by call WifiP2pManager.requestPeers to get
          WifiDirectManager.requestPeers()
      }
  }
  ```
  - 可以看到，当可用对等节点列表变化时，系统会发送 WIFI_P2P_PEERS_CHANGED_ACTION 广播，我们有两种方式获取对等节点列表
  - API >= 18 的时候可以直接在 intent 携带数据中获取到 WifiP2pDeviceList 对象
  - 当 API 在[14, 18) 之间的时候，只能调用manager的requestPeers方法，在回调当中获取
  - **其实**：只要都采用第二种方式，就能保持一致性了
  - 具体输出结果可以自己试一试
- ### connect
  ```kotlin
  /**
   * connect by MAC address(hardware address)
   */
  fun connect(deviceAddress: String) {
      val config = WifiP2pConfig()
      config.deviceAddress = deviceAddress
      config.wps.setup = WpsInfo.PBC
      manager.connect(channel, config, object : WifiP2pManager.ActionListener {
          override fun onSuccess() {
              println("connect operator success")
          }

          override fun onFailure(reason: Int) {
              println("connect operator fail: $reason")
          }
      })
  }

  /**
   * invoke this method to connect a p2p device
   */
  fun connect(device: WifiP2pDevice): WifiDirectManager {
      connect(device.deviceAddress)
      return this
  }
  ```
  - 在前面我们已经获取到对等节点列表了，每个对等节点设备信息里面包含其mac地址，用mac地址就能连接
  - 连接成功以后 系统就会发送 WIFI_P2P_CONNECTION_CHANGED_ACTION 广播了，你可在里面获取对方设备的信息以及群组的信息
  - 如果两个对等节点都没有创建群组，则连接之后其中一端设备会自动成为群组，每个组员都能获取到群组的IP
- ### WIFI_P2P_CONNECTION_CHANGED_ACTION 包含的信息
  - 给WifiDirectManager类添加几个封装函数
    ```Kotlin

    /**
     * request the group info
     */
    fun requestGroup(): WifiDirectManager {
        manager.requestGroupInfo(channel) { group ->
        }
        return this
    }

    /**
     * create a group
     */
    fun createGroup(): WifiDirectManager {
        manager.createGroup(channel, object : WifiP2pManager.ActionListener {
            override fun onFailure(reason: Int) {
                println("create group fail: $reason")

            }

            override fun onSuccess() {
                println("create group success")
            }
        })
        return this
    }

    /**
     * remove a group
     */
    fun removeGroup(): WifiDirectManager {
        manager.removeGroup(channel, object : WifiP2pManager.ActionListener {
            override fun onSuccess() {
                println("remove group success")
            }

            override fun onFailure(reason: Int) {
                println("remove group success: $reason")
            }

        })
        return this
    }


    ```
  - 看看Receiver里面的代码
    ```kotlin
    /**
     * 当一个连接建立或断开的时候，系统会发送该广播
     * This action received when the connection setup or dis-setup
     */
        WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION -> {
            val networkInfo = intent.getParcelableExtra<NetworkInfo>(WifiP2pManager.EXTRA_NETWORK_INFO)
            val wifiP2pInfo = intent.getParcelableExtra<WifiP2pInfo>(WifiP2pManager.EXTRA_WIFI_P2P_INFO)

            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
                val wifiP2pGroupInfo =  intent.getParcelableExtra<WifiP2pGroup>(WifiP2pManager.EXTRA_WIFI_P2P_GROUP)
            } else {

            }
        }
    }
    ```
  - 从上面可以看出，当收到系统发送的 WIFI_P2P_CONNECTION_CHANGED_ACTION 广播的时候，我们可以获取下面信息
    - NetworkInfo
    - WifiP2pInfo (对端设备的信息)
    - WifiP2pGroup （两种方式获取， 和之前获取对等节点列表的操作类似）

- ### 应用方式
  - 可以一个设备主动创建一个Group，然后其它设备检索后连接
  - 一旦连接成功，这些设备就处于同一自组网下了，组员是可以直接在WifiP2pGroup里面获取到群主的ip。接下来就可以直接用Socket进行各种骚操作了
  - 至于群主如何获取组员的IP，目前想到的一种方式是群主开一个Socket监听，组员连接成功通过socket向群主发送一个请求，然后群主就可以在请求的socket对象里面获取组员的IP了
