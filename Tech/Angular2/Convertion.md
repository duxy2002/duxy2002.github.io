# Angular2 命名规则

## 1.命名

* 组件的TS文件命名： 组件名称.component.ts
* 组件的HTML文件命名： 组件名称.component.html
* 组件的CSS文件命名：组件名称.component.css
* 根模块的类名叫做AppModule,放在app.module.ts文件中。

## 2. 编程规则(Rule)
应该为每个模块建立独立的路由文件。

## 3.关于模块的最佳实践
### 3.1 共享特性模块
* 坚持在shared目录中创建名叫SharedModule的特性模块（例如在app/shared/shared.module.ts中定义SharedModule）  

    ```typescript
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { FormsModule } from '@angular/forms';
    import { MdlModule } from 'angular2-mdl';
    
    @NgModule({
      imports: [
        CommonModule,
        FormsModule,
        MdlModule
      ],
      exports: [
        CommonModule,
        FormsModule,
        MdlModule
      ]
    })
    export class SharedModule { }
    ```
* 坚持把可能被应用其他特性模块使用的**公共**组件，指令和管道放在SharedModule中，这些资产倾向于共享自己的新实例。
* 坚持在SharedModule中声明所有组件，指令和管道。
* 坚持从SharedModule中导出其他特性模块所需的全部符号。
* 避免在SharedModule中指定应用级的**单例**服务提供商。但如果是故意设计的单例也可以，不过还是小心。
> 在子模块中导入SharedModule  

```typescript
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { SharedModule } from '../shared/shared.module';
import { BingImageService } from './bing-image.service';
import { LoginRoutingModule } from './login-routing.module';
import { LoginComponent } from './login.component';
import { RegisterDialogComponent } from './register-dialog.component';

@NgModule({
  imports: [
    SharedModule,
    LoginRoutingModule,
    ReactiveFormsModule
  ],
  declarations: [
    LoginComponent,
    RegisterDialogComponent
    ],
  entryComponents: [RegisterDialogComponent],
  providers: [
    { provide: 'bing', useClass: BingImageService }
  ]
})
export class LoginModule { }


```

### 3.2 核心特性模块
* 坚持把那些“只用一次”的类收集到CoreModule中，并对外隐藏它们的实现细节。简化的AppModule会导入CoreModule，并且把它作为整个应用的总指挥。
```typescript
//core.module.ts
import { ModuleWithProviders, NgModule, Optional, SkipSelf } from '@angular/core';
import { AuthService } from './auth.service';
import { UserService } from './user.service';
import { AuthGuardService } from './auth-guard.service';

@NgModule({
  providers: [
    { provide: 'auth', useClass: AuthService },
    { provide: 'user', useClass: UserService },
    AuthGuardService
    ]
})
export class CoreModule {
  constructor (@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error(
        'CoreModule is already loaded. Import it in the AppModule only');
    }
  }
}
```
 我们希望只在应用启动时导入AuthService，UserService,AuthGuardService一次，而不会在其他地方导入它。
 在模块的构造函数中我们会要求Angular把CoreModule注入自身，这看起来像一个危险的循环注入。不过@SkipSelf装饰器意味着在当前注入器的所有祖先注入器中寻找CoreModule。
 如果该构造函数在我们所期望的AppModule中运行，那就没有任何祖先注入器能够提供CoreModule的实例，于是注入器会放弃查找。
 默认情况下，当注入器找不到想找的提供商时，会抛出一个错误。但@Optional装饰器表示找不到该服务业无所谓。于是注入器会返回null,parentModule参数也就被赋成了控制，
 而构造函数没有任何异常。
 
 
* 坚持在core目录下创建一个名叫CoreModule的特性模块（例如在app/core/core.module.ts中定义CoreModule）。
* 坚持把一个要共享给整个应用的单例服务放进CoreModule中（例如ExceptionService和LoggerService)
> 找找有没有已经做好的LoggerService
* 坚持导入CoreModule中的资产所需要的全部模块（例如CommonModule和FormsModule）
* 坚持把应用级，只用一次的组件收集到CoreModule中。只在应用启动时从AppModule中导入它一次，以后再也不要导入它（例如NavComponent和SpinnerComponent等)
* 坚持防范多次导入CoreModule，并通过添加守卫逻辑来尽快失败。
* 避免在AppModule之外的任何地方导入CoreModule.
    ```typescript
        import { BrowserModule } from '@angular/platform-browser';
        import { NgModule } from '@angular/core';
        import { HttpModule, JsonpModule } from '@angular/http';
        import { MdlModule } from 'angular2-mdl';
        import { CoreModule } from './core/core.module';
        import { AppRoutingModule } from './app-routing.module';
        import { TodoModule } from './todo/todo.module';
        import { LoginModule } from './login/login.module';
        
        import { AppComponent } from './app.component';
        
        @NgModule({
          declarations: [
            AppComponent
          ],
          imports: [
            ...,
            CoreModule,
          ],
          bootstrap: [AppComponent]
        })
        export class AppModule { }

    ```