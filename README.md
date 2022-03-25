# dotnet6-csharp10-minimal-api
Playing with minimal APIs

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
Resulting size: 55,484 KB
```