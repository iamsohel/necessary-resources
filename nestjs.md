https://github.com/mguay22
https://www.youtube.com/watch?v=0WgO3-HVH94&list=PLJ8v-58rML8_p8vCXjoGVCltwhkmgdMVd

1. decoration has no access on di
2. to access service you have to use interceptor first
3. handle() will pass the next like express next() function
4. TO use DI, you have make it Injectable() and give it to providers

5. middeware vs interceptor vs exception filter
   
As you already implied with your question, all three are very similar concepts and in a lot of cases it is hard to decide and comes down to your preferences. But I can give an overview of the differences:

Interceptors

Interceptors have access to response/request before and after the route handler is called.

Registration

Directly in the controller class with @UseInterceptors() controller- or method-scoped
Globally with app.useGlobalInterceptors() in main.ts

Examples

LoggingInterceptor: Request before route handler and afterwards its result. Meassure time it takes.
ResultMapping: Transform null to [] or wrap result in a response object: users -> {users: users

Conclusion

I like that the registration is closer to the route handlers compared to middleware. But there are some limitations, for example, you cannot set the response code or alter the response with Interceptors when you send the response with the library-specific @Res() object in your route handler, see docs.

Middleware

Middleware is called only before the route handler is called. You have access to the response object, but you don't have the result of the route handler. They are basically express middleware functions.

Registration

In the module, very flexible way of choosing relevant routes (with wildcards, by method,...)
Globally with app.use() in main.ts

Examples

FrontendMiddleware: redirect all routes except API to index.html, see this thread
You can use any express middleware that is out there. There are lots of libraries, e.g. body-parser or morgan

Conclusion

The registration of middleware is very flexible, for example: apply to all routes but one etc. But since they are registered in the module, you might not realize it applies to your controller when you're looking at its methods. It's also great that you can make use of all the express middleware libraries that are out there.

Exception Filters

Exception Filters are called after the route handler and after the interceptors. They are the last place to make changes before a response goes out.

Registration

Directly in the controller class with @UseFilters() controller- or method-scoped
Globally app.useGlobalFilters() in your main.ts

Examples

UnauthorizedFilter: Map to an easy to understand message for the user
NotFoundFilter: Map all routes that are not found (not part of your api) to your index.html.

Conclusion

The basic use case for exception filters are giving understandable error messages (hiding technical details). But there are also other creative ways of usage: When you serve a single page application, then typically all routes should redirect to index.html except the routes of your API. Here, you can redirect on a NotFoundException. Some might find this clever others hacky. Your choice. ;-)

So the execution order is:
Middleware -> Interceptors -> Route Handler -> Interceptors -> Exception Filter (if exception is thrown)

With all three of them, you can inject other dependencies (like services,...) in their constructor.

![image](https://github.com/iamsohel/necessary-resources/assets/9135426/05edf90b-127b-41dd-b63f-4d299bab385a)

