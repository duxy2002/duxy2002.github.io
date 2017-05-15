#angular2 入门

## 1.引导过程

* 通过即时（JiT）编译器动态引导，一般在进行开发调试时，默认采用这种方式。
* 使用预编译器（Ahead-Of-Time, AoT）进行静态引导，静态方案可以生成更小，启动更快的应用。
> Task：怎麼在代码中混合它们。



## 2.模块(Modules)
@NgModule接收一个元数据对象，该对象告诉Angular如何编译和运行模块代码。它指出模块拥有的组件，指令和管道，并把它们的一部分公开出去，以便外部组件使用它们。  
* declarations: 声明本模块中拥有的视图类。Angular有3中视图类:组件，指令和管道。

    1. 列出应用中的顶层组件。
    
    **在module里面声明的组件在module范围内都可以直接使用，
    也就是说在同一module里面的任何Component都可以在其模板文件中直接使用声明的组件。**
    
    
* exports: declarations的子集，可用于其他模块的组件模板。
* imports: 本模块声明的组件模块需要的<strong>类</strong>所在的<strong>其他模块</strong>。

    1. BrowserModule提供了运行在浏览器中的应用所需要的关键服务(Service)和指令(Directive)，这个模块对于所有需要在浏览器中跑的应用来说都必须引用。
    2. FormsModule提供了表单处理和双向绑定等服务和指令。
    3. HttpModule提供HTTP请求和相应的服务。
    
* providers: 服务的创建者，并<strong>加入到全局服务列表中，可用于应用的任何部分</strong>。
   
   **列出会在此模块中“注入”的服务(Service)**
   >个人理解：也就是说可以“注入”的类，在这儿列出。
   
   在根模块中的这个providers是配置在模块中全局可用的service或参数的。定义好以后，我们就可以在任意组件中注入这个依赖了。
   
   ```typescript
   //app.module.ts
   import { AuthService} from '../core/auth/service';
   ...
   @NgModule{
    providers: [
        {provider: 'auth', useClass: AuthService}
    ]
   }
   ------------------------------------------------------------------
     //login.component.ts
    @Component({
    
    })
    export class LoginComponent implements OnInit {
      constructor(@Inject('auth') private service) 
    }
   
   ```
   >注意上面的@Inject的使用。
   
   而第二种方法是直接在login.component.ts文件中注入。方法如下：
   ```typescript
     //login.component.ts
   import {AuthService} from '../core/auth.service';
    
    @Component({
      selector: ....,
      template: ....,
      providers: [AuthService]
    })
    export class LoginComponent implements OnInit {
      constructor(private service:AuthService) 
    }
   
   ```
   >注意上面的Import, providers以及constructor的写法。
   
   > 个人理解：把FW的东西通过第一种方法定义，而各个业务功能的Service通过第二种方法定义。
   
* bootstrap: 指定应用的主视图（称为根组件），它是所有其他视图的宿主。<strong>只有根模块才能设置bootstrap属性</strong>。

### 2.1 子模块
  **根模块（AppModule）都应该从@angular/platform-browser中导入BrowserModule。在其他任何模块中都不要导入BrowerModule.应该改成导入CommonModule。。**
  对于子模块而言，它们不需要重新初始化全应用级别的提供商（也就是说app.module.ts中的providers)
  
## 3. 组件(Components)

### 组件和视图(View)
 
 **组件**负责控制屏幕上的一小块区域（称为“**视图**”）
 


    
## 4.指令(Directives)

## 5.模板(Templates)
 1. (eventName)
 例如下面代码中的(click)表示我们要处理这个button的click事件。

        <buttong (click)="onClick()">Login</button>。

## 6.依赖注入(Dependency Injection)

## 7.服务(Service)

## 8.数据绑定(Data Binding)

 1. 给控件加引用(reference)  
    例如下面的#usernameRef。如果想传递input的值，可以用usernameRef.value.
        
        <input #usernameRef type="text">
        <button (click)="onClick(usernameRef.value)">Login</button>
        
 2. 双向数据绑定
    方括号[]的作用是说把等号后面当成表达式来解析而不是当成字符串。它的含义是单向绑定，就是说我们在组件中给model赋值会设置到HTML的input控件中。
    [()]是双向绑定的意思，就是说HTML对应控件的状态改变会反射设置到组件的model中。ngModel是FormModel中提供的指令，它负责从Domain Model中创建一个FormControl的实例，并将这个实例和表单的控件绑定起来。  

    ```typescript
        <input type="text" [(ngModel)]="username"
    ```  
        
    有了[()]就不需要1.的usernameRef.value了。
 3. 表单数据的验证  

```typescript
    <input required type="text" [(ngModel)]="username" #usernameRef="ngModel"/>
    <div *ngIf="usernameRef.errors?.required">this is required</div>
```
这儿的#usernameRef="ngModel"和1.的#usernameRef都是引用，但是这次的引用指向了ngModel，这个引用时要在模板中使用的，所以才加入这个引用。

## 9. 路由(routing)

1. 在src/index.html中指定基准路径，即在\<header>中加入\<base href="/">,它指向你的index.html所在的路径，浏览器也会根据这个路径下载css,图像和js文件，所以请将这个语句放在header的最顶端
2. 在src/app/app.module.ts中引入RouterModule:

```typescript
import {RouterModule} from '@angular/router';
```
3. ~~而后在imports中定义和配置路由数组。~~

```typescript
imports: [
 BrowserModule,
 Forms,
 HttpModule,
 RouterModule.forRoot([
     path: 'login',
     component: LoginComponent
 ])
 ]
```

forRoot其实是一个静态工厂方法，它返回的仍然是Module。因为这个路由定义是应用在应用根部的。

路由定义这个对象包括若干属性：
* path: 路由器会用它来匹配路由中指定的路径和浏览器地址中的当前路径。如/login
* component: 导航到此路由时，路由器需要创建的组件，如LoginComponent。
* redirectTo: 重定向到某个path,使用场景的话，比如在用户输入不存在的路径时重定向到首页。
> C: 这个怎么实现
* pathMatch: 路径的字符匹配策略
* children：子路由数组


注意：路径配置的顺序是非常重要的，Angular2使用“先匹配优先”的原则，也就是说，如果一个路径可以同时匹配几个路径匹配的规则，以第一个匹配的规则为准。

直接在app.module.ts中定义路由不是很好的方式。这部分最好还是用单独的文件来定义。
新建一个文件src/app/app.routes.ts，将上面在app.modules.ts中定义的路径删除并在app.routes.ts中重新定义。

```typescript
import {Route, RouterModule} from '@angular/router';
import {LoginComponent} from './login/login.component';
import {ModuleWithProviders} from './@angular/core';
export const routes: Routes = [{
    path: 'login',
    component: LoginComponent
}];
export const routing:ModuleWithProviders = RouterModule.forRoot(routes);
```

接下来在app.module.ts中引入routing。

```typescript
import {routing} from './app.routes';
@NgModule({
imports: [
    ..........
    routing]
})
```

### 9.1 子路由
对于子模块而言我们需要子路由。这时在子路由文件中的最后应该使用RouterModule.forChild(routes);
而这个时候怎么把根模块，根路由，子模块，子路由连接起来呢。比如说我们有子模块todo.那么
```typescript
//src/app/todo/todo.module.ts
import {routing} from './todo/routes';
import {TodoComponent} from ',/todo.component';
.....
@NgModule({
 imports: [
     .......
     routing
 ] ,
 declarations: [
     TodoComponent,
     .....
 ]
})
export class TodoModule{}

//todo.routes.ts
import {Route, RouterModule} from '@angular/router';
import {TodoComponent} from ',/todo.component';
export const routes: Routes = [{
        path: 'todo',
        component: TodoComponent
}];
export const routing:ModuleWithProviders = RouterModule.forChild(routes);

//app.routes.ts
export const routes: Routes =[ {
  path： ‘todo'',
   redirectTo: 'todo'
}];
export const routing = RouteModule.forRoot(routes);

//app.module.ts

@NgModule({
imports:[
    TodoModule
]
})
```
首先app.routes.ts中去掉了TodoComponent的依赖，而且更改todo路径定义为redirectTo到todo路径，但没有给出组件，这叫做“无组件路由”，也就是说后面的事情是TodoModule负责的。

注意：这个时候其实没有任何一个地方还需要引用<app-todo></app-todo>了，这就是说我们可以安全地把selector:'app-todo'，从Todo组件中的@Component修饰符中删除了。

>也就是说子模块中的父组件不需要selector?
