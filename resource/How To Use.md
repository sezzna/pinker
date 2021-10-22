## 如何使用

1. 下载并转译项目总目录 proto/resource 目录下的 resource.proto 文件.注意不是 pinker-resource 目录下,

   您也可以直接引用 proto/resource/resource 目录下以编译好的 pb 文件

   

2. 创建 go-micro 服务

   ``` go
   service := micro.NewService(micro.Name("Service_Name"))
   ```

3. 新建资源服务客户端

   ``` go
   rsc := resource.NewRscService("Resource_Service", service.Client())
   ```

4. 订阅异步事件

   ``` go
   //异步上传事件
   micro.RegisterSubscriber("Event_Resource_PutRsc", s.Server(), EventHandler)
   ```

5. 编写事件处理函数

   ```go
   //异步上传事件处理
   func EventHandler(ctx context.Context, e *resource.EventSyncPutRsc) error {
   	//TODO
   	return nil
   }
   ```

6. RPC 同步调用

   ``` go
   //同步上传资源
   b, err := ioutil.ReadFile("exmple.jpg")
   if err != nil {
   	return err
   }
   in := resource.ReqPutRsc{
   	FileName: "exmple.jpg",
   	Body:     b,
   }
   
   //注意:这里请考虑 RPC 参数传递和资源上传到 S3 的时间来合理的设置 CallOptions 超时时间
   var opt client.CallOption = func(o *client.CallOptions) {
   	o.RequestTimeout = time.Second * 30
   	o.DialTimeout = time.Second * 30
   }
   
   if _, err := rsc.PutRsc(context.Background(), &in, opt); err != nil {
   	return
   }
   ```

7. RPC 异步调用

   ``` go
   //异步上传资源
   b, err := ioutil.ReadFile("exmple.gif")
   if err != nil {
     return err
   }
   in := resource.ReqPutRscSync{
   	UserID:   888888,
   	FileName: "exmple.gif",
   	Body:     b,
   }
   //注意,如果文件过大这里还是需要像同步上传一样设置超时事件
   if _, err := rsc.PutRscSync(context.Background(), &in); err != nil {
   	return
   }
   ```

## 注意事项

- 目前只支持 jpg, png, gif, mp4 的资源上传
- 请在上传资源之前使用 http.DetectContentType() 函数验证是否能识别资源类型
- 如果是同步调用请考虑 RPC 参数传递和资源上传到 S3 的时间来合理的设置 CallOptions 超时时间
- 如果是异步调用只用考虑 RPC 传参时间来设置 context 超时时间

## 返回值

- 如果发生错误使用标准 error 类型返回
- 异步调用时的返回请查看具体事件内容和流程图中的说明

## 运维

- 请在 conf/conf.toml 文件中设置数据库各项配置
- 请在 conf/conf.toml 文件中设置为服务相关配置
- 请在 conf/aws/config 文件中设置 S3 所在地区
- 请在 conf/aws/credentials 文件中设置 S3 用户 aws_access_key_id 和 aws_secret_access_key

