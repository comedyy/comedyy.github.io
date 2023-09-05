## dotnet core asp
目前看代码很简洁。

创建一个dotnet core asp的应用
```
dotnet new webapp -o MyWebApp --no-https -f net7.0 
```

动态的修改代码自动生效
```
dotnet watch 
```

#### 代码层
Program.cs:

```
// 创建一个WebApplication
var builder = WebApplication.CreateBuilder(args);
    
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
     c.SwaggerDoc("v1", new OpenApiInfo { Title = "PizzaStore API", Description = "Making the Pizzas you love", Version = "v1" });
});
    
var app = builder.Build();    

// 创建一个WebApplication
app.MapPost("/upload", Uploader.Upload);
app.MapPost("/download", Uploader.Download);

// 启动
app.Run();
```

然后是定义各种的 行为；
```

public class Uploader
{
    // 下载一个文件， HttpContext中，有一个request跟一个response。
    async internal static Task Download(HttpContext context)
    {
        context.Response.ContentType = "image/png";
        using (var fileStream = File.OpenRead("save/xxxx.zip")) 
        {
            await fileStream.CopyToAsync(context.Response.Body);
        }
    }

    // 上传一个文件， HttpContext中，有一个request跟一个response。里面有stream，可以读写。
    async internal static Task<IResult> Upload(HttpRequest request)
    {
        if(!request.HasFormContentType)
        {
            return Results.BadRequest("not form content " + request.ContentType);
        }

        var fileName = request.Form["name"];
        var form = await request.ReadFormAsync();
        var fileForm = form.Files["file"];

        if(fileForm == null || fileForm.Length == 0)
        {
            return Results.BadRequest("file not exits");
        }

        if(!Directory.Exists("save"))
        {
            Directory.CreateDirectory("save");
        }

        fileName = "save/" + fileName;
        await using(var stream = fileForm.OpenReadStream())
        {
            await using(var file = File.Create(fileName))
            {
                await stream.CopyToAsync(file);
            }    
        }

        return Results.Ok();
    }
}
```

对应的unity的上传跟下载：

```
// 下载一个文件， 使用DownloadHandlerFile，可以自动下载到目标位置。
private IEnumerator GetMessage()
{
    var x = UnityWebRequest.Post("https://localhost:7192/download", "");
    x.certificateHandler = new IgnoreCertificateHander();
    x.downloadHandler = new DownloadHandlerFile(Application.dataPath + "/../aaa.zip");
    yield return x.SendWebRequest();

    Debug.LogError("send" + x.downloadedBytes +  " " + x.isHttpError + " " + x.responseCode );
}

// 上传一个文件， Post有可以填充一个binary。然后发送给服务器，服务器从 request的Read的form中读取出Files。
// IgnoreCertificateHander 可以同http服务器交互，而不需要https。
private IEnumerator SendMessage()
{
    WWWForm form = new WWWForm();
    form.AddField("name", "xxxx.zip");
    form.AddBinaryData("file", new byte[]{1, 2, 3, 4,5});

    var x = UnityWebRequest.Post("https://localhost:7192/upload", form);
    x.certificateHandler = new IgnoreCertificateHander();
    yield return x.SendWebRequest();

    Debug.LogError("send" + x.downloadHandler.text + " " + x.isHttpError + " " + x.responseCode );
}
```
