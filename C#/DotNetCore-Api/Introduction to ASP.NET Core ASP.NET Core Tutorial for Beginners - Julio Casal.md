
https://www.youtube.com/watch?v=bKCzoR01lpE&t=5190s

![[Pasted image 20240609210848.png]]
this course mainly focues on minimal api.

![[Pasted image 20240609211406.png]]

![[Pasted image 20240609211452.png]]

![[Pasted image 20240609211543.png]]

![[Pasted image 20240609211714.png]]

![[Pasted image 20240609211754.png]]
![[Pasted image 20240609211816.png]]

//press f2 to rename the variable name

MODULE 3: Code Organization and validations

![[Pasted image 20240612212341.png]]

#using-route-group
instead of defining seperate route groups. We can just use MapGroup to have a common route. 
var group = app.MapGroup("/games");

#Adding-server-side-validation
to make validation before data is sent to the server we can use data-annotations to validate the data. 
![[Pasted image 20240612213920.png]]

#Introduction-to-nuget
![[Pasted image 20240612215821.png]]
just writing data annotation will not force the code to validate the data. so we are using use a nuget package which already has the logic for validation. {End-Point Filter}

#Refactoring-the-endpoint
so we are going to move all the endpoints into a brand new class that is going to be in charge of defining these endpoints and we are going to use extension methods to make this very easy.

since all extension methods are static, we are also going to make this class static.

in the next module, instructor is going to introduce us a few design pattern to future proof our api from future changes.

Module 4: using design patterns and best practices

![[Pasted image 20240612222532.png]]

![[Pasted image 20240612222641.png]]

#Introduction-to-repository-pattern
![[Pasted image 20240612222702.png]]

![[Pasted image 20240612222819.png]]
 in here instructor give us example of how we might face problem in the future. lets say our application is connected to a sql database and after a few year our application becomes popular and we cannot scale due to constraints of relational database and want to move to nosql db but since our code is related to sql we may have to write our entire logic of the application to match the nosql which is very time consuming and error prone. 
so, instead of this, we can use repository pattern
![[Pasted image 20240612223119.png]]

![[Pasted image 20240612223213.png]]

#Adding-the-games-repository
Now we will see how to apply repository pattern. 
There is also an associated pattern called unit of work that we will not be using here because it is mostly related to having transactions across multiple entities or multiple tables in our data storage.

so, we will create a new folder name Repositories.

#Understanding-dependency-injection
![[Pasted image 20240612225208.png]]

![[Pasted image 20240619214056.png]]

![[Pasted image 20240619214139.png]]

This time my logger is not explicitly constructed by my service() instead mylogger is passed in as a constructor parameter. My logger is injected into the my service constructor. this way my service doesn't need to know how to construct or configure the logger, it just receives it and can start using it right away.  But if my service doesn't construct the logger then who does it? 
-> The answer is asp .net core provides the iservice provider which is known as a service container. Our application can register my logger and any other dependencies with iservice provider during startup which is typically done in our program.cs file.  then later when a new HTTP request arrives and our web app need an instance of my service, the service container will notice its dependencies and it will go ahead and resolve, construct and inject those dependencies into a new instance of my service via Constructor.
![[Pasted image 20240619214932.png]]

![[Pasted image 20240619215319.png]]

#Understandinh-service-lifetime 
![[Pasted image 20240619215333.png]]

we know that in startup our application will register the dependencies like my logger here. 
![[Pasted image 20240619215601.png]]
 and later when an HTTP request arrives Iservice provider will resolve, construct and inject an instance of my logger in to a new instance of our class. "my service in this example"
 ![[Pasted image 20240619215810.png]]

  what's not clear is what happens when a new request arrives should I-service provider create a brand new mylogger instance for the new request or should it reuse the same instance 
  ![[Pasted image 20240619220001.png]]

   What is another service that also has dependency on my-logger needs to be created in response to a new request.  save my logger instance or new my-logger instance.
   -> The answer to this lies in the service life time which we configure when we register my-logger with I-service provider
   ![[Pasted image 20240619220324.png]]

There are 3 available service lifetimes .
![[Pasted image 20240619220453.png]]

lets say that my logger is aver light weight and stateless service, so its okay to create a new instance every single time any class needs it. so in this case we should add my-logger to add transient method. 
![[Pasted image 20240619220720.png]]
![[Pasted image 20240619220745.png]]
![[Pasted image 20240619220802.png]]


![[Pasted image 20240619220815.png]]

what if my logger is a class that keeps track of some sort of state that needs to be shared across multiple classes that participates in an HTTP request in that case, we will register my logger with add scooped method.
![[Pasted image 20240619221059.png]]

![[Pasted image 20240619221126.png]]

if another service has the same dependencies on the mylogger then it will recieve the same instance. 
![[Pasted image 20240619221239.png]]

and this is for a new http request
![[Pasted image 20240619221321.png]]

![[Pasted image 20240619221411.png]]

and finally lets say my-logger is not cheap to instantiate and it keeps track of the state that should be shared with all clasees that requested during the entire lifetime of our application. then well will register add logger to add singleton method
![[Pasted image 20240619221551.png]]

![[Pasted image 20240619221625.png]]

for a new http request,
![[Pasted image 20240619221645.png]]

![[Pasted image 20240619221702.png]]

#Understanding_Data-Transfer-Object
![[Pasted image 20240623225347.png]]

![[Pasted image 20240623225430.png]]

![[Pasted image 20240623225541.png]]

![[Pasted image 20240623225600.png]]

#Using-DTO 
for this we will be using record types in c# because they allows us to create immutable classes, primarily designed to hold data and these matches our DTO requirements.

all extentions method should be static

**Module 5: Configuring the API to connect to SQL Server**
#Configuring_the_API_to_connect_toSQL_Server 
![[Pasted image 20240704220057.png]]

![[Pasted image 20240704220133.png]]

![[Pasted image 20240704220356.png]]

![[Pasted image 20240704220422.png]]



#Reading-Configuration-from-appsettingjson
At this point we need to figure out a way to connect our dotnet api to sql server and as we may know that we do that is by defining a connection string and the connection string includes all of the parameters needed to connect to a database.
now we could go ahead and define the connection string directly in the program.cs file 
![[Pasted image 20240705212600.png]]

but that's usually not a good idea because the connection parameters are usually going to change across different environments. like into test, or production environment, development environment. 
and the best way to do this is by using .Net configuration system and one of the basic way to do it by using the file appsettings.json and appsettings.development.json


#Stroring-secrets-for-local-development
![[Pasted image 20240705213755.png]]

in this chapter, we will learn how to use .net secret manager to start our application secret during  local development. 
![[Pasted image 20240705214033.png]]

paste this in the command line 
![[Pasted image 20240705214315.png]]

secret key will placed in our application 
![[Pasted image 20240705214510.png]]


Module 6- Entity Framework Core 

![[Pasted image 20240705215119.png]]

![[Pasted image 20240705215145.png]]

#Introduction-to-Entiity-Framework-Core
![[Pasted image 20240705215221.png]]

![[Pasted image 20240705215331.png]]

![[Pasted image 20240705215407.png]]

![[Pasted image 20240705215509.png]]

add this in the terminal 
```
dotnet add package Microsoft.EntityFrameworkCore.SqlServer 
```
