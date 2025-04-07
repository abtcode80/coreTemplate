# coreTemplate


# .NET 8 Console App with Dependency Injection and HttpClient

## 🛠️ Project: MyConsoleApp

### 📁 File Structure

```
MyConsoleApp/
│
├── Program.cs
├── Services/
│   ├── IAgifyService.cs
│   └── AgifyService.cs
├── Http/
│   ├── IHttpClientWrapper.cs
│   └── HttpClientWrapper.cs
```

---

## 1. Create the Console App

```bash
dotnet new console -n MyConsoleApp
cd MyConsoleApp
```

---

## 2. Make Sure You Target .NET 8 in .csproj

```xml
<TargetFramework>net8.0</TargetFramework>
```

---

## 3. Create the Service Layer

### Services/IAgifyService.cs

```csharp
namespace MyConsoleApp.Services;

public interface IAgifyService
{
    Task GetAgeAsync(string name);
}
```

### Services/AgifyService.cs

```csharp
using MyConsoleApp.Http;

namespace MyConsoleApp.Services;

public class AgifyService : IAgifyService
{
    private readonly IHttpClientWrapper _httpClient;

    public AgifyService(IHttpClientWrapper httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task GetAgeAsync(string name)
    {
        var query = $"?name={name}";
        var response = await _httpClient.GetAsync("https://api.agify.io", query);
        Console.WriteLine(response);
    }
}
```

---

## 4. Create HTTP Client Wrapper

### Http/IHttpClientWrapper.cs

```csharp
namespace MyConsoleApp.Http;

public interface IHttpClientWrapper
{
    Task<string> GetAsync(string baseUrl, string query);
}
```

### Http/HttpClientWrapper.cs

```csharp
using System.Net.Http.Headers;

namespace MyConsoleApp.Http;

public class HttpClientWrapper : IHttpClientWrapper
{
    private readonly HttpClient _client;

    public HttpClientWrapper(HttpClient client)
    {
        _client = client;
    }

    public async Task<string> GetAsync(string baseUrl, string query)
    {
        var request = new HttpRequestMessage(HttpMethod.Get, baseUrl + query);
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Headers.Add("X-Demo-Header", "HeaderValue123");

        var response = await _client.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsStringAsync();
    }
}
```

---

## 5. Program.cs — Register Dependencies and Run

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using MyConsoleApp.Http;
using MyConsoleApp.Services;

var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((_, services) =>
    {
        services.AddHttpClient();
        services.AddTransient<IHttpClientWrapper, HttpClientWrapper>();
        services.AddTransient<IAgifyService, AgifyService>();
    })
    .Build();

var agifyService = host.Services.GetRequiredService<IAgifyService>();

Console.Write("Enter a name to predict age: ");
var name = Console.ReadLine();

await agifyService.GetAgeAsync(name ?? "John");
```

---

## ✅ Sample Output

```
Enter a name to predict age: Alice
{"name":"Alice","age":51,"count":12345}
```

---

## ✅ API Used

- **Agify.io**: https://api.agify.io
