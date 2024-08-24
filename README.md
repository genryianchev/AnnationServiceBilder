
# AnnationServiceBilder

AnnationServiceBilder is an ASP.NET Core library that simplifies dependency injection by using custom annotations to automatically register services in the DI container.

## Prerequisites

### Web Application

1. **.NET 6.0 or later**
2. **Visual Studio 2022** (or any other IDE with .NET support)
3. **Git** (for version control)

### Recommended Tools

- **Visual Studio Code** with extensions:
  - [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) (for .NET development)
  - [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) (for code formatting)
  - [EditorConfig](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) (for consistent coding styles)

### Important Note

- Ensure that your .NET SDK is up-to-date. This project requires .NET 6.0 or later.

## Installation

1. Clone the repository to your local machine:

   ```bash
   git clone https://github.com/yourusername/AnnationServiceBilder.git
   cd AnnationServiceBilder
   ```

2. Open the project in Visual Studio or your preferred IDE.

3. Restore the necessary packages by building the solution.

## Setting up Annotations

The `Annotations` folder in your project contains the key attributes used for automatically registering services in the DI container. Below are the files and their purposes, along with examples of how they are used:

### 1. **Scoped Annotations**

**Files:**
- `ScopedServiceAttribute.cs` (path: `Annotations/Scoped/ScopedServiceAttribute.cs`)
- `ScopedGenericServiceAttribute.cs` (path: `Annotations/Scoped/ScopedGenericServiceAttribute.cs`)

**Description:**
- **ScopedServiceAttribute:** This attribute is used to mark classes that should be registered with a *scoped* lifetime in the DI container. Optionally, you can specify the service type with which the class will be registered.

**Example Code:**

```csharp
namespace AnnationServiceBilder.Annotations.Scoped
{
    [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
    public sealed class ScopedServiceAttribute : Attribute
    {
        public Type ServiceType { get; }

        public ScopedServiceAttribute(Type serviceType = null)
        {
            ServiceType = serviceType;
        }
    }
}
```

- **ScopedGenericServiceAttribute:** This attribute is used to register generic classes with a *scoped* lifetime.

**Example Code:**

```csharp
namespace AnnationServiceBilder.Annotations.Scoped
{
    [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
    public sealed class ScopedGenericServiceAttribute : Attribute
    {
        public Type ServiceType { get; }

        public ScopedGenericServiceAttribute(Type serviceType)
        {
            ServiceType = serviceType;
        }
    }
}
```

### 2. **Singleton Annotations**

**File:**
- `SingletonServiceAttribute.cs` (path: `Annotations/Singleton/SingletonServiceAttribute.cs`)

**Description:**
- **SingletonServiceAttribute:** This attribute is used to mark classes that should be registered with a *singleton* lifetime in the DI container.

**Example Code:**

```csharp
namespace AnnationServiceBilder.Annotations.Singleton
{
    [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
    public sealed class SingletonServiceAttribute : Attribute
    {
    }
}
```

### 3. **Transient Annotations**

**File:**
- `TransientServiceAttribute.cs` (path: `Annotations/Transient Services/Transient Services.cs`)

**Description:**
- **TransientServiceAttribute:** This attribute is used to mark classes that should be registered with a *transient* lifetime in the DI container.

**Example Code:**

```csharp
namespace AnnationServiceBilder.Data.Transient_Services
{
    [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
    public sealed class TransientServiceAttribute : Attribute
    {
    }
}
```

### 4. **Refit Annotations**

**File:**
- `RefitClientAttribute.cs` (path: `Annotations/Refit/RefitClient.cs`)

**Description:**
- **RefitClientAttribute:** This attribute is used to mark interfaces that should be registered as Refit clients, enabling easy integration with HTTP APIs.

**Example Code:**

```csharp
namespace AnnationServiceBilder.Annotations.Refit
{
    [AttributeUsage(AttributeTargets.Interface, Inherited = false, AllowMultiple = false)]
    public sealed class RefitClientAttribute : Attribute
    {
        public string BaseUrl { get; }

        public RefitClientAttribute(string baseUrl = null)
        {
            BaseUrl = baseUrl;
        }
    }
}
```

### 5. **Service Registration Extensions**

**File:**
- `ServiceCollectionExtensions.cs` (path: `Annotations/ServiceCollectionExtensions.cs`)

**Description:**
- This file contains extension methods for `IServiceCollection` that automatically register all annotated services in the DI container.

**Example Code:**

```csharp
using AnnationServiceBilder.Annotations.Refit;
using AnnationServiceBilder.Annotations.Scoped;
using AnnationServiceBilder.Annotations.Singleton;
using AnnationServiceBilder.Data.Transient_Services;
using Refit;
using System.Reflection;

namespace AnnationServiceBilder.Annotations
{
    public static class ServiceCollectionExtensions
    {
        public static void AddRefitClientsFromAttributes(this IServiceCollection services, Assembly assembly, string defaultBaseUrl)
        {
            var refitClientTypes = assembly.GetTypes()
                .Where(t => t.IsInterface && t.GetCustomAttribute<RefitClientAttribute>() != null);

            foreach (var interfaceType in refitClientTypes)
            {
                var attribute = interfaceType.GetCustomAttribute<RefitClientAttribute>();
                var baseUrl = new Uri(attribute.BaseUrl ?? defaultBaseUrl);

                services.AddRefitClient(interfaceType)
                        .ConfigureHttpClient(c => c.BaseAddress = baseUrl);
            }
        }

        public static void AddAnnotatedSingletonServices(this IServiceCollection services, Assembly assembly)
        {
            var singletonTypes = assembly.GetTypes()
                .Where(t => t.GetCustomAttribute<SingletonServiceAttribute>() != null && t.IsClass && !t.IsAbstract);

            foreach (var type in singletonTypes)
            {
                services.AddSingleton(type);
            }
        }

        public static void AddAnnotatedTransientServices(this IServiceCollection services, Assembly assembly)
        {
            var transientTypes = assembly.GetTypes()
                .Where(t => t.GetCustomAttribute<TransientServiceAttribute>() != null && t.IsClass && !t.IsAbstract);

            foreach (var type in transientTypes)
            {
                services.AddTransient(type);
            }
        }

        public static void AddAnnotatedScopedServices(this IServiceCollection services, Assembly assembly)
        {
            var scopedTypes = assembly.GetTypes()
                .Where(t => t.GetCustomAttribute<ScopedServiceAttribute>() != null && t.IsClass && !t.IsAbstract);

            foreach (var type in scopedTypes)
            {
                var attribute = type.GetCustomAttribute<ScopedServiceAttribute>();
                var serviceType = attribute.ServiceType ?? type;
                services.AddScoped(serviceType, type);
            }

            var genericScopedTypes = assembly.GetTypes()
                .Where(t => t.GetCustomAttribute<ScopedGenericServiceAttribute>() != null && t.IsClass && !t.IsAbstract);

            foreach (var type in genericScopedTypes)
            {
                var attribute = type.GetCustomAttribute<ScopedGenericServiceAttribute>();
                services.AddScoped(attribute.ServiceType, type);
            }
        }
    }
}
```

## Usage

Here are examples of how to use each annotation in your project:

### **1. Using Scoped Services**

```csharp
using AnnationServiceBilder.Annotations.Scoped;

[ScopedService(typeof(IMyScopedService))]
public class MyScopedService : IMyScopedService
{
    // Implementation...
}
```

### **2. Using Singleton Services**

```csharp
using AnnationServiceBilder.Annotations.Singleton;

[SingletonService]
public class MySingletonService
{
    // Implementation...
}
```

### **3. Using Transient Services**

```csharp
using AnnationServiceBilder.Annotations.Transient_Services;

[TransientService]
public class MyTransientService
{
    // Implementation...
}
```

### **4. Using Refit Clients**

```csharp
using AnnationServiceBilder.Annotations.Refit;
using AnnationServiceBilder.Data.Models;
using Refit;

namespace AnnationServiceBilder.Network.Repositories
{
    [RefitClient]
    public interface IPostsApi
    {
        [Get("/posts")]
        Task<List<Post>> GetPostsAsync();

        [Get("/posts/{id}")]
        Task<Post> GetPostByIdAsync(int id);
    }
}
```

### **5. Registering Services**

In your `Startup.cs` or `Program.cs`, you should register the services as follows:

```csharp
var assembly = Assembly.GetExecutingAssembly();
services.AddAnnotatedSingletonServices(assembly);
services.AddAnnotatedScopedServices(assembly);
services.AddAnnotatedTransientServices(assembly);
services.AddRefitClientsFromAttributes(assembly, "https://api.yourservice.com"); // Replace with your API base URL
```

## Contributing

We welcome contributions! Please submit a pull request or open an issue to discuss your ideas or report bugs.

## License

This project is licensed under

 the MIT License - see the [LICENSE](LICENSE) file for details.

