---
layout: post
title: 试水新的Angular4 HTTP API
categories:
- web
tags:
- angular
---

Angular更新了新的 4.3.0-rc.0 版本。在这个版本里，我们可以发现更新了我们一直期待的新功能 —— 一个革新了的HTTP Client API

> HttpClient 是对现存的Angular HTTP API 一次进化，现有的HTTP API存在于一个单独的包中，即@angular/common/http。
这样的结构确保了已有的代码库可以慢慢更新到新的API而不至于出现断崖的更新

### 安装

首先，我们需要更新包版本到 4.3.0-rc.0 版本。
接下来，我们需要把 `HttpClientModule` 引入到我们的 `AppModule` 里

```````````````````````

\\http-init.ts

import { HttpClientModule } from '@angular/common/http';
@NgModule({
 declarations: [
   AppComponent
 ],
 imports: [
   BrowserModule,
   HttpClientModule
 ],
 bootstrap: [AppComponent]
})
export class AppModule { }

````````````````````````

现在我们准备好了。让我们来看看三个令人期待的新功能

### 默认使用JSON

JSON 作为默认的数据格式而不再需要明确地写出来需要解析

我们再也不需要写下如下的代码

`http.get(url).map(res => res.json()).subscribe(...)`

现在我们只需要写下


`http.get(url).subscribe(...)`

### 拦截器（Interceptors）支持

拦截器 允许在 管道语法（pipeline）中注入中间件


```````````````````````````
\\request-interceptor.ts

import {
  HttpRequest,
  HttpHandler,
  HttpEvent
} from '@angular/common/http';

@Injectable()
class JWTInterceptor implements HttpInterceptor {
  
  constructor(private userService: UserService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    const JWT = `Bearer ${this.userService.getToken()}`;
    req = req.clone({
      setHeaders: {
        Authorization: JWT
      }
    });
    return next.handle(req);
  }
}

````````````````````````````

如果我们想要注册一个新的拦截器，我们需要去实现（implements）这个 `HttpInterceptor` 接口（interface）。这个接口有一个方法我们必须要去实现 —— 即拦截器

这个拦截器方法将会给我们一个请求对象（the Request object）、HTTP处理器（the HTTP handler）并且返回一个HttpEvent 类型的可观察对象（observable）

请求和返回对象需要是不可改变的。因此，我们需要在返回之前提前拷贝一个原始请求

接下来，next.handle(req) 方法将会调用一个带上新请求的底层的XHR然后返回一个返回事件的事件流（stream）


`````````````````````````````
\\interceptor-response.ts

@Injectable()
class JWTInterceptor implements HttpInterceptor {

  constructor(private router: Router) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    
    return next.handle(req).map((event: HttpEvent<any>) => {
      if (event instanceof HttpResponse) {
        // do stuff with response if you want
      }
    }, (err: any) => {
      if (err instanceof HttpErrorResponse {
        if (err.status === 401) {
           // JWT expired, go to login
        }
      }
    });
  }
}

``````````````````````````````


拦截器也可以选择通过应用附加的 Rx 操作符来转换响应事件流对象，在next.handle()中返回。

最后我们需要去做的注册该拦截器，使用 HTTP_INTERCEPTORS 注册 multi Provider：

`[ { provide: HTTP_INTERCEPTORS, useClass: JWTInterceptor, multi: true } ]`


### 进度事件

进度事件可以既用于请求上传也可以用于返回的下载

```````````````````````````````````````
\\http-progress.ts

import {
  HttpEventType,
  HttpClient,
  HttpRequest
} from '@angular/common/http';

http.request(new HttpRequest(
  'POST',
  URL,
  body, 
  {
    reportProgress: true
  })).subscribe(event => {

  if (event.type === HttpEventType.DownloadProgress) {
    // {
    // loaded:11, // Number of bytes uploaded or downloaded.
    // total :11 // Total number of bytes to upload or download
    // }
  }

  if (event.type === HttpEventType.UploadProgress) {
    // {
    // loaded:11, // Number of bytes uploaded or downloaded.
    // total :11 // Total number of bytes to upload or download
    // }
  }

  if (event.type === HttpEventType.Response) {
    console.log(event.body);
  }

})

``````````````````````````````````````````

如果我们想要获得上传、下载进度的提示信息，我们需要传 `{ reportProgress: true }`给 `HttpRequest`对象

这里还有两个新的功能我们今天没有提到：

基于内部测试框架的 `Post-request verification` 和 `flush` 功能
类型化，同步响应体访问，包括对 JSON body类型的支持。

以上只是对新的HTTP API和它的新功能的概述，完整的代码可以看 [angular/packages/common/http](https://github.com/angular/angular/tree/master/packages/common/http)

译者注

应该在哪里注册拦截器呢？
``````````````````````````````````````````
\\app.module.ts
@NgModule({ 
  imports: [ BrowserModule ], 
  providers: [ 
  { provide: HTTP_INTERCEPTORS, useClass: JWTInterceptor, multi:   true } 
],
  declarations: [ AppComponent ], 
  bootstrap: [ AppComponent ] 
}) 
export class AppModule { }
``````````````````````````````````````````