# Asp.net-Core-2.1-Jwt-Authentication-Middleware

### What is JWT?
I have explained the about the Jwt token based authentication system in the https://github.com/hidayatarg/Decode-JWT-Token.

### What is a middleware

A middleware is nothing but a component (class) which is executed on every request in ASP.NET Core application. In the classic ASP.NET, HttpHandlers and HttpModules were part of request pipeline. Middleware is similar to HttpHandlers and HttpModules where both needs to be configured and executed in each request.

Typically, there will be multiple middleware in ASP.NET Core web application. It can be either framework provided middleware, added via NuGet or your own custom middleware. We can set the order of middleware execution in the request pipeline. visit www.tutorialsteacher.com for more explaintion

![](http://www.tutorialsteacher.com/Content/images/core/middleware-1.png)

![](http://www.tutorialsteacher.com/Content/images/core/request-processing.png)

## Authentication middleware
In my Project, When ever the user request it will start and check the user Role according to the user role the user would be able to request from the system. 
```csharp
public class AuthenticationMiddleware
{
   private readonly RequestDelegate _next;
   
   // Dependency Injection
   public AuthenticationMiddleware(RequestDelegate next)
   {
       _next = next;
   }

   public async Task Invoke(HttpContext context)
   {
	   //Reading the AuthHeader which is signed with JWT
       string authHeader = context.Request.Headers["Authorization"];

       if (authHeader != null)
       {   
	       //Reading the JWT middle part           
           int startPoint = authHeader.IndexOf(".") + 1;               
           int endPoint = authHeader.LastIndexOf(".");

           var tokenString = authHeader
	           .Substring(startPoint, endPoint - startPoint).Split(".");
           var token = tokenString[0].ToString()+"==";

           var credentialString = Encoding.UTF8
               .GetString(Convert.FromBase64String(token));
        
           // Splitting the data from Jwt
           var credentials = credentialString.Split(new char[] { ':',',' });

           // Trim this Username and UserRole.
           var userRule = credentials[5].Replace("\"", ""); 
           var userName = credentials[3].Replace("\"", "");

            // Identity Principal
           var claims = new[]
           {
               new Claim("name", userName),
               new Claim(ClaimTypes.Role, userRule),
           };
           var identity = new ClaimsIdentity(claims, "basic");
           context.User = new ClaimsPrincipal(identity);
       }
       //Pass to the next middlware
       await _next(context);
   } 
}
```
To call this Middle-ware, we need to place it in the startup.cs
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
       {
           ...
 
           // Authorization Middleware
           app.UseMiddleware<AuthenticationMiddleware>();
	       // Other Middlewares are also placed here 
        //   app.UseMiddleware<AuthMiddleware>();
	      
	      ...
   }
```
These middlewares are executed with order from top to bottom.
