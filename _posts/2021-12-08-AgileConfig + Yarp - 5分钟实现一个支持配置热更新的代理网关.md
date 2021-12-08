```
Install-Package Yarp.ReverseProxy -Version 1.0.0
```
```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration);
var app = builder.Build();
app.MapReverseProxy();
app.Run();
```
```
{
 "Logging": {
   "LogLevel": {
     "Default": "Information",
     "Microsoft": "Warning",
     "Microsoft.Hosting.Lifetime": "Information"
   }
 },
 "AllowedHosts": "*",
 "ReverseProxy": {
   "Routes": {
     "route1" : {
       "ClusterId": "cluster1",
       "Match": {
         "Path": "{**catch-all}"
       },
     }
   },
   "Clusters": {
     "cluster1": {
       "Destinations": {
         "destination1": {
           "Address": "https://example.com/"
         }
       }
     }
   }
 }
}
```

```
sudo docker run \
--name agile_config \
-e TZ=Asia/Shanghai \
-e adminConsole=true \
-e db:provider=sqlite \
-e db:conn="Data Source=agile_config.db" \
-p 5000:5000 \
#-v /your_host_dir:/app/db \
-d kklldog/agile_config:latest

```

```
Install-Package AgileConfig.Client -Version 1.2.1.5
```
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "AgileConfig": {
    "appId": "yarp_test",
    "secret": "",
    "nodes": "http://localhost:5000/"
  }
}

```
```
var builder = WebApplication.CreateBuilder(args);

//add agileconfig configuration provider
builder.Host.ConfigureAppConfiguration((_, bd) => {
    bd.AddAgileConfig();
});

builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration);

var app = builder.Build();
app.MapReverseProxy();
app.Run();
```