# General principles
When testing a Web API, you can choose to test at different levels. 
1. The API surface (endpoints)
2. The different parts making up your service
3. A mixture of both. 

Option 2 is the classic unit test. For my services I tend to avoid this approach.  
My tests are focused on manipulating the endpoints; that way I can refactor the internals without impacting tests. 

API testing has become a lot easier with ASP.NET Core since it's built for it from the ground up. 
Microsoft has also provided some classes that makes things easier. 

## The test context
All test must be independent of each other; that allows them to run in parallel. It's also to avoid tests polluting each other. 
Static fields that get 'remembered' between tests can cause a lot of grief. 

In order to isolate tests, I use a ```TestContext``` class. This is based on the ```WebApplicationFactory``` provided by Microsoft. 

```CS
public class TestContext : WebApplicationFactory<Startup>
```

The ```WebApplicationFactory``` configures some things for ASP.NET Core. The rest of the code in the ```TestContext``` can be used to set things up for our tests. I have a number of extensibilty points that allow individual tests to change things. 

- ```AdditionalServices```
- ```AdditionalConfiguration```
- ```AdditionalLogging```

The ```TestContext``` has also been set up to capture logging information so that it's available as part of the xUnit test. Very helpful when troubleshooting why a test fails. 

The ```TestContext``` is something I set up for each test project. It can be used to set up mocking data if your service has external dependencies. You can configure a default mock, then override it for specific tests using the extensibilty points. 

## Test structure
This is the general structure of the tests.

```CS
public class SomeTests
{
    private readonly ITestOutputHelper _output;

    public SomeTests(ITestOutputHelper output)
    {
        _output = output;
    }

    [Fact]
    public async Task SomeTest()
    {
        using (var tc = new TestContext(_output))
        {
            tc.AdditionalConfiguration = (context, builder) =>
            {
                // perform configuration customization if needed
            };
            
            tc.AdditionalLogging = (context, builder) =>
            {
                // perform additional logging configuration if needed
            };
            tc.AdditionalServices = (context, services) =>
            {
                // register additional services id needed
            };

            // the ApiClients is a strongly typed client to access your service from tests
            var result  = await tc.ApiClient.DoSomething();
            Assert.NotNull(result);
        }
    }
}
```

## The API Client
If you are building a Web API service, it should expose an Open API definition (Swagger). 
Your service may have multiple users, so maintaining the contract and avoiding breaking changes is important. One of the dangers with autogenerated contracts is that they can change, and thus break existing clients. 

My personal prefrence is to generate a version and store it as file. That way source control can track who made what changes. If your tests use this file as a source for your tests, you can detect if the service makes breaking changes. It doesn't cost much to do and can prevent you from making mistakes. 