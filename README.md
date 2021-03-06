### 说明

安装包主要根据官方包进行重构以及根据官方api开放文档进行开发，主要有以下改进：
* 修复了携程官方包在docker以network为host时获取localip失败导致程序无法运行的问题
* 由于直接基于.net standard 2.0 开发代码也比较轻量级
* 支持离线缓存文件(应用根目录/appsettings.cache)

**[升级日志](https://github.com/zengqinglei/Creekdream.Configuration.Apollo/releases)**

##### 安装SDK
```
Install-Package Creekdream.Configuration.Apollo
```
##### 初始化配置
```
// appsettings.json 配置如下：
{
  "apollo": {
    "AppId": "FabricDemo.UserService",
    "MetaServer": "http://pro.meta-server.zengql.local",
    "Namespaces": ["application","TEST1.ConsulPublic"]
  }
}
```

```
// 方式一：
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
	WebHost.CreateDefaultBuilder(args)
		.ConfigureAppConfiguration(
			(hostingContext, builder) =>
			{
				builder.AddApollo(builder.Build().GetSection("apollo"));
			})
		.UseStartup<Startup>();
```

```
// 方式二：
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration(
            (hostingContext, builder) =>
            {
                builder.AddApollo(
                    optionsConfig =>
                    {
                        var apolloOptions = builder.Build().GetSection("apollo");
                        optionsConfig.AppId = apolloOptions.GetValue<string>("AppId");
                        optionsConfig.MetaServer = apolloOptions.GetValue<string>("MetaServer");
                        apolloOptions.GetValue<List<string>>("Namespaces")
                            .ForEach(namespaceName=> optionsConfig.Namespaces.Add(namespaceName));
                    });
            })
        .UseStartup<Startup>();
```

##### 使用示例，请查看单元测试
```
private readonly IConfiguration _configuration;

public ApolloConfigurationTest()
{
    var builder = new ConfigurationBuilder();
    builder.AddApollo(
        optionsConfig =>
        {
            optionsConfig.AppId = "FabricDemo.UserService";
            optionsConfig.MetaServer = "http://pro.meta-server.zengql.local";
            optionsConfig.Namespaces.Add("application");
            optionsConfig.Namespaces.Add("TEST1.ConsulPublic");
        });
    _configuration = builder.Build();
}

[Fact]
public void Test_Get_ConsulAddress()
{
    var address = _configuration.GetValue<string>("ConsulClient:ClientAddress");
    address.ShouldNotBeNull();
}
```