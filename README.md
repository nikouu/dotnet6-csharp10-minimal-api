# dotnet6-csharp10-minimal-api
Playing with minimal APIs

---

Hey, so this was a fun attempt at minimal APIs that got sidetracked into creating a small `.exe`. If you want to see some real work in creating a shrunken `.exe` check out my project [TinyWordle](https://github.com/nikouu/TinyWordle)!

---

To create: `dotnet new webapp`

Via [David Fowler](https://twitter.com/davidfowl/status/1419574650124738562/photo/1)

And remebering that a [single file executable](https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file/overview) has to contain all the `.dll` files required to run it
```xml
	<PublishSingleFile>true</PublishSingleFile>
	<RuntimeIdentifier>win-x64</RuntimeIdentifier>
	<SelfContained>true</SelfContained>
	<PublishReadyToRun>true</PublishReadyToRun>
```

## Making a smaller .exe
Checking out what (Building a self-contained game in C# under 8 kilobytes)[https://medium.com/@MStrehovsky/building-a-self-contained-game-in-c-under-8-kilobytes-74c3cf60ea04] does.

### Attempt 1
```xml
	<PublishSingleFile>true</PublishSingleFile>
	<RuntimeIdentifier>win-x64</RuntimeIdentifier>  
```

```
dotnet publish -r win-x64 -c Release
```

```
Resulting size: 83,923 KB
```

### Attempt 2

```
dotnet publish -r win-x64 -c Release /p:PublishTrimmed=true
```

```
Resulting size: 36,057 KB
```

### Attempt 3
[Using experimental AOT](https://github.com/dotnet/runtimelab/blob/feature/NativeAOT/docs/using-nativeaot/compiling.md), need to remove single file part


```
dotnet publish -r win-x64 -c Release
```

```
Resulting size: 21,933 KB
```

### Attempt 4
Some probably unnecessary (since using AOT) trimming commands via [Trimming in .NET 5](https://devblogs.microsoft.com/dotnet/app-trimming-in-net-5/)

```
dotnet publish -r win-x64 -c Release -p:PublishTrimmed=True -p:TrimMode=Link
```

```
Resulting size: 21,933 KB
```

### Attempt 5
Back to reading the [AOT docs](https://github.com/dotnet/runtimelab/tree/feature/NativeAOT/docs/using-nativeaot)

```xml
  <PropertyGroup>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
```

```
dotnet publish -r win-x64 -c Release -p:PublishTrimmed=True -p:TrimMode=Link
```

```
Resulting size: 21,718 KB
```

### Attempt 6

```xml
	<PropertyGroup>
		<IlcGenerateCompleteTypeMetadata>true</IlcGenerateCompleteTypeMetadata>
	</PropertyGroup>
	<ItemGroup>
		<TrimmableAssembly Include="dotnet6-csharp10-minimal-api" />
	</ItemGroup>
```

```
Resulting size: 22,617 KB
```

Except, it doesn't run.
```
System.Reflection.MissingMetadataException: 'System.Text.Json.Serialization.Converters.IEnumerableOfTConverter<System.Linq.Enumerable.SelectRangeIterator<WeatherForecast>,WeatherForecast>' is missing metadata. For more information, please visit http://go.microsoft.com/fwlink/?LinkID=392859
```

### Attempt 7

```
<IlcDisableReflection>true</IlcDisableReflection>
```

Immediately terminates, but it makes sense since I'm pretty sure some of the aspnet core uses it.

```
Unhandled Exception: EETypeRva:0x0067BDB0: Reflection_Disabled
   at Internal.Reflection.ReflectionExecutionDomainCallbacksImplementation.GetDelegateMethod(Delegate) + 0x33
   at Microsoft.AspNetCore.Hosting.GenericWebHostBuilder.Configure(Action`2) + 0x3e
   at Microsoft.AspNetCore.Builder.WebApplicationBuilder.<>c__DisplayClass6_0.<.ctor>b__1(IWebHostBuilder) + 0x33
   at Microsoft.Extensions.Hosting.GenericHostWebHostBuilderExtensions.ConfigureWebHost(IHostBuilder, Action`1, Action`1) + 0x92
   at Microsoft.AspNetCore.Builder.WebApplicationBuilder..ctor(WebApplicationOptions, Action`1) + 0x188
   at Microsoft.AspNetCore.Builder.WebApplication.CreateBuilder(String[]) + 0x41
   at Program.<Main>$(String[]) + 0x21
   at dotnet6-csharp10-minimal-api!<BaseAddress>+0x4a436f

```
```
Resulting size: 10,553 KB
```

### Attempt 8
Turns out, it's not been working for awhile due to 
```
System.Reflection.MissingMetadataException: 'System.Text.Json.Serialization.Converters.IEnumerableOfTConverter<System.Linq.Enumerable.SelectRangeIterator<WeatherForecast>,WeatherForecast>' is missing metadata. For more information, please visit http://go.microsoft.com/fwlink/?LinkID=392859
         at System.Reflection.Runtime.General.TypeUnifier.WithVerifiedTypeHandle(RuntimeConstructedGenericTypeInfo, RuntimeTypeInfo[]) + 0x88
         at System.Text.Json.Serialization.Converters.IEnumerableConverterFactory.CreateConverter(Type, JsonSerializerOptions) + 0x656
         at System.Text.Json.Serialization.JsonConverterFactory.GetConverterInternal(Type, JsonSerializerOptions) + 0x15
         at System.Text.Json.JsonSerializerOptions.GetConverterFromType(Type) + 0x1c0
         at System.Collections.Concurrent.ConcurrentDictionary`2.GetOrAdd(TKey, Func`2) + 0x85
         at System.Text.Json.JsonSerializerOptions.GetConverterFromMember(Type, Type, MemberInfo) + 0x9a
         at System.Text.Json.Serialization.Metadata.JsonTypeInfo.GetConverter(Type, Type, MemberInfo, Type&, JsonSerializerOptions) + 0x57
         at System.Text.Json.JsonSerializerOptions.<InitializeForReflectionSerializer>g__CreateJsonTypeInfo|120_0(Type, JsonSerializerOptions) + 0x3a
         at System.Collections.Concurrent.ConcurrentDictionary`2.GetOrAdd(TKey, Func`2) + 0x85
         at System.Text.Json.JsonSerializer.GetTypeInfo(JsonSerializerOptions, Type) + 0x73
         at System.Text.Json.JsonSerializer.SerializeAsync[TValue](Stream, TValue, JsonSerializerOptions, CancellationToken) + 0x37
         at Microsoft.AspNetCore.Http.HttpResponseJsonExtensions.<WriteAsJsonAsyncSlow>d__3`1.MoveNext() + 0x47
```

but that [link takes us to a page](https://dotnet.github.io/native/troubleshooter/type.html) that sorts out a `rd.xsl` file

## After a night on it, prob not worth starting to understand optimising outputs when it involes something as complex a the asp.net pipeline lol

Next time I'll use the [AOT documentation](https://github.com/dotnet/runtimelab/tree/feature/NativeAOT/docs/using-nativeaot) properly.
