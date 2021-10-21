如何使用
----

1.  下载并转译项目总目录 Proto/resource 目录下的 rsc.proto 文件.注意不是 pinker-resource 目录下
    
2.  创建 go-micro 服务
    
    service :\= micro.NewService(micro.Name("Service\_Name"))
    
3.  新建资源服务客户端
    
    svc :\= rsc.NewRscService("Resource\_Service", service.Client())
    
4.  订阅异步事件
    
    micro.RegisterSubscriber("Event\_RSC\_GetRsc", service.Server(), EventHandler)  
    micro.RegisterSubscriber("Event\_RSC\_GetPresignedURL", service.Server(), EventHandler)  
    micro.RegisterSubscriber("Event\_RSC\_GetURL", service.Server(), EventHandler)
    
5.  RPC 异步调用
    
    //上传资源并获取资源URL  
    req :\= rsc.ReqPutRscGetURL{  
    Key: "test\_video.mp4",//资源名称,也是 S3 唯一资源标识 Key  
    Body: bytes,//资源 \[\]byte  
    }  
    ​  
    //RPC 异步调用  
    svc.SyncGetRscWithKey(context.TODO(), &req)
    

注意事项
----

*   目前只支持 jpg, png, gif, mp4 的资源上传
    
*   请在上传资源之前使用 http.DetectContentType() 函数验证是否能识别资源类型
    
*   如果是同步调用请考虑 RPC 参数传递和资源上传到 S3 的时间合理的设置 context 超时时间
    
*   如果是异步调用只用考虑 RPC 传参时间来设置 context 超时时间
    

错误码
---

*   目前暂时以 1 代表成功 -1 代表失败,后期会根据项目增加详细的错误码
    
*   失败时请仔细阅读 err 信息.并及时反馈
    

运维
--

*   请在 conf/aws/config 文件中设置 S3 所在地区
    
*   请在 conf/aws/credentials 文件中设置 S3 用户 aws\_access\_key\_id 和 aws\_secret\_access\_key
    
*   请填写 conf/conf.ini 文件中所需设置
    
*   当然您可以自定义 ini 文件的位置并在项目启动时给出 conf 参数指定文件路径即可