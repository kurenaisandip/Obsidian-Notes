
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
just writting data annotation will not force the code to validate the data. so we are using use a nuget package which already has the logic for validation. {End-Point Filter}

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
