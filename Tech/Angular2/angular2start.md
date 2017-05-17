# angular2 入门

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
所有组件的输出内容都必须是EventEmitter的实例。
在Angular2语境下，直接定义在组件模板里面子标签叫做view children(视图子节点)，内嵌在标签中的子标签叫做content children(内容子节点)。
父子组件通信的过程中，模板充当类似于桥梁的角色，连接着二者的功能逻辑。

###3.1 组件和视图(View)
 
 **组件**负责控制屏幕上的一小块区域（称为“**视图**”）

脏值检测机制运行的上下文是独立的组件。分区会拦截浏览器发出的所有异步请求，并为框架的脏值检测机制提供执行上下文。
注意：只有组件才会绑定脏值检测器实例，而Directive会使用它们所属Component上的检测器。

### 3.2 脏值检测策略
* CheckOnce
* Checked
* CheckAlways
   默认会使用这个。
* Detached
* Default
* OnPush
  可以利用不可变数据和OnPush策略提升性能。当组件产生的结果只和它的输入参数有关的时候，这种策略非常有用。

    
## 4.指令(Directives)
与组件相反，指令没有对应的视图和模板。
Angular2引入了一些更加简单地约定并且内置到了框架里面：
* propertyName=“value”
  value是一个字符串字面量。Angular不会对这个值做进一步处理，这个值在模板里面是什么样就会怎么样去使用。
* \[propertyName]="expression"
  将会提示Angular2这个属性的值应该当成表达式来处理。当Angular2发现属性包围在方括号内部时，就会尝试去执行这个表达式，执行时的上下文就是模板所对应的组件。
* (eventName)="handler()" 
* <td \[class.completed]=""todo.completed"></td>
  假如todo.complted是true的话，那么上面这句话会变成<td class="completed"></td>
* <td \[attr.colspan]="colspanCount""></td>
  假如colspanCount的值是2的话，那么上面这句话会变成<td colspan="2"></td>
* <td \[style.backgroundImage="expression"></td>
  假如expresssion是1.bmp的话，那么上面这句话会变成<td style="backgroundImage:1.bmp"></td>
Angular2中的指令分成三种：
* 结构型(Structural)指令
    * ngIf
    * ngFor
      ngForOf属性外面包着一个方括号。乍一看方括号貌似不是合法的HTML语法。但是根据HTML规范，方括号可以用在标签的属性名里面。
      ```typescript
        <template ngFor var-todo [ngForOf]="todos">
          <li>{{todo}}</li>
        </tempalte>
      ```
      我们通过var-tdo这种语法来告诉Angular:我们需要创建一个新的变量叫做todo。不过Angular2提供了一个更简便的语法糖：我们可以用#toto来代替var-todo.
    * ngSwitch/ngSwitchCase/ngSwitchDefault
* 属性型(Attribute)指令
    * ngModel
    * ngClass
* Component

###4 .1.自定义管道
我们可以使用ES2015装饰器@Pipe来创建一个新的管道。这个装饰器可以通过添加元数据的方式把类声明为一个管道。我们唯一要做的事情就是给管道一个名字并定义好数据格式化逻辑。
```typescript
@Pipe({name: 'lowercasel'})
class LowerCasePipe1 implements  PipeTransform {
    transform(value: string) : string {
        if (!value) return value;
        if (typeof value != 'string') {
            throw new Error('Invalid pipe value', value);
        }
        return value.toLocaleLowerCase();
    }
}

@Component({
selector: 'app',
declarations: [LowerCasePipe1],
template: '<h1>{{“SAMPLE" | lowercase1 }}</h1>'
})
class App{}
```

## 5.模板(Templates)
 1. (eventName)
 例如下面代码中的(click)表示我们要处理这个button的click事件。

        <buttong (click)="onClick()">Login</button>。

## 6.依赖注入(Dependency Injection)
注入到组件里的服务只能使用在该组件机器子组件上，而注入到模块里的服务在整个应用里均能使用，因为所有模块都共享着同一个应用级别的根注入器。

## 7.服务(Service)
@Injectable()表示ContactService需要注入它所依赖的其他服务（如HTTP服务）
## 8.数据绑定(Data Binding)

 1. ~~给控件加引用(reference)~~  
    ~~例如下面的#usernameRef。如果想传递input的值，可以用usernameRef.value.~~
        
        <input #usernameRef type="text">
        <button (click)="onClick(usernameRef.value)">Login</button>
        
 2. ~~双向数据绑定~~
    ~~方括号[]的作用是说把等号后面当成表达式来解析而不是当成字符串。它的含义是单向绑定，就是说我们在组件中给model赋值会设置到HTML的input控件中。~~
    ~~\[()]是双向绑定的意思，就是说HTML对应控件的状态改变会反射设置到组件的model中。ngModel是FormModel中提供的指令，它负责从Domain Model中创建一个FormControl的实例，并将这个实例和表单的控件绑定起来。~~  

    ```typescript
        <input type="text" [(ngModel)]="username"
    ```  
        
    ~~有了\[()]就不需要1.的usernameRef.value了。~~
 3. ~~表单数据的验证~~  

    ```typescript
        <input required type="text" [(ngModel)]="username" #usernameRef="ngModel"/>
        <div *ngIf="usernameRef.errors?.required">this is required</div>
    ```
~~这儿的#usernameRef="ngModel"和1.的#usernameRef都是引用，但是这次的引用指向了ngModel，这个引用时要在模板中使用的，所以才加入这个引用。~~


### 8.1 响应式表单
响应式表单意味着我们不会使用ngModel，required等其他的类似的指令来帮助我们完成绑定和验证等动作。也就是说我们希望自己对表单（Form）有完全的控制，而不是像1.2.3做的那样。
响应式表单有两个明显优点
* 可以让我们将所有处理的逻辑放在一起，而不是像非响应式表单那样，验证在网页模板中，绑定在模板中，逻辑处理在组件中等等。
* 更大的灵活性，因为我们可以从头到脚的控制表单，而不是依赖某些内建的机制（虽然那些机制优势会给你很多便利之处，但一旦你的需求变复杂时，它就无法满足你了。）

    1. 在我们可以使用ReactiveForms前，首先要告诉@NgModule引入。注意：平时使用时如果要时间模板驱动型表单需要引入FormModule，而相应式表单需要引入ReactiveForms。
        ```typescript
         //login.module.ts
          import { ReactiveFormsModule } from '@angular/forms';
          @NgModule({
            imports: [
              ....,
              ReactiveFormsModule
            ],
            declarations: [... ],
            entryComponents: [RegisterDialogComponent],
            providers: [....]
            })
           export class LoginModule { }
         ```
       
     > entryComponents: [RegisterDialogComponent],是什么意思？
     
     2. 表单控件和表单组,表单构造器
        1. 在组件中import下面的类  
             ```typescript
             //login.module.ts
            import {
              FormGroup,
              Validators,
              FormControl,
              FormBuilder
            } from '@angular/forms';
             ```
         2. 而后在constructor方法中假如下面的代码
             ```typescript
             //register.component.ts
                public form: FormGroup;
                ....
                  constructor(
                    private dialog: MdlDialogReference,
                    private fb: FormBuilder,
                    private router: Router,
                    @Inject('auth') private authService) {
                      this.form = fb.group({
                        'username':  new FormControl('',  Validators.required),
                        'passwords': fb.group({
                          'password': new FormControl('', Validators.required),
                          'repeatPassword': new FormControl('', Validators.required)
                        },{validator: this.passwordMatchValidator})
                      });
             ```
          也就是说通过构造函数注入FormBuilder,我们得到表单构造器。
          3. 相对应的HTML如下
             ```html
               //register.component.html
                <form [formGroup]="form">
                  <h3 class="mdl-dialog__title">Register</h3>
                  <div class="mdl-dialog__content">
                    <mdl-textfield
                      #firstElement
                      type="text"
                      label="Username"
                      formControlName="username"
                      floating-label>
                    </mdl-textfield>
                    <br/>
                    <div formGroupName="passwords" #passwordsRef>
                      <mdl-textfield
                        type="password"
                        label="Password"
                        formControlName="password"
                        floating-label>
                      </mdl-textfield>
                      <br/>
                      <mdl-textfield
                        type="password"
                        label="Repeat Password"
                        formControlName="repeatPassword"
                        floating-label>
                      </mdl-textfield>
                    </div>
                  </div>
                  <div class="status-bar">
                    <p class="mdl-color-text--primary">{{statusMessage}}</p>
                    <mdl-spinner [active]="processingRegister"></mdl-spinner>
                  </div>
                  <div class="mdl-dialog__actions">
                    <button
                      type="button"
                      mdl-button
                      (click)="register()"
                      [disabled]="!form.valid || processingRegister"
                      mdl-button-type="raised"
                      mdl-colored="primary" mdl-ripple>
                      Register
                    </button>
                  </div>
                </form>
             ```
         [formGroup]="form"中的form就是定义在component中的form。而 formControlName="password"就是component中form里面定义的password.formGroupName="passwords"对应的是form中的passwords. 
        
        
     
## 9. 路由(routing)

导航操作，URL变更，加载合适的模板，以及在视图加载完成后调用特定逻辑，与这些内容相关的职责都交给路由组件来承担。
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

页面迁移的方法：
this.router.navigate(['todo'])


## 9.2 路由守卫
路由器支持多种守卫
* 用CanActivate来处理导航到某路由的情况。
* 用CanActivateChild处理导航到子路由的情况。
* 用CanDeactivate来处理从当前路由离开的情况。
* 用Resolve在路由激活之前获取路由数据。
* 用CanLoad来处理异步导航到某特性模块的情况。
在分层路由的每个级别上，我们都可以设置多个守卫。路由器会先按照从最深的子路由由下往上检查的顺序来检查CanDeactivate守护条件。然后他会按照从上到下的顺序检查CanActivate守卫。如果任何守卫返回false，其他尚未完成的守卫会被取消，这样整个导航就被取消了。

> 用户权限检查可以在这儿做。  


### 9.3 异步路由
loadChildren


## 10.动画
在添加动画之前，先引入一些与动画有关的类库
```typescript
//login.component.ts
import {
    Component,
    Inject,
    trigger,
    state,
    style,
    transition,
    animate,
    OnDestroy
} from '@angular/core';
@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css'],
  animations: [
    trigger('loginState', [
      state('inactive', style({
        transform: 'scale(1)'
      })),
      state('active',   style({
        transform: 'scale(1.1)'
      })),
      transition('inactive => active', animate('100ms ease-in')),
      transition('active => inactive', animate('100ms ease-out'))
    ])
  ]
})
export class LoginComponent implements OnDestroy {
    
}
```
我们通过上面的方法定义了一个动画，要想使用它，可以在模板中用[@triigerName]="xxx""的形式来把它附加到一个或多个元素上。
```html
//login.component.html
    <button
      mdl-button mdl-button-type="raised"
      mdl-colored="primary"
      mdl-ripple type="submit"
      [@loginState]="loginBtnState"
      (mouseenter)="toggleLoginState(true)"
      (mouseleave)="toggleLoginState(false)">
      Login
    </button>
```
这里我们对Login这个按钮应用了loginState触发器，并且绑定这个触发器的状态值到一个成员变量**loginBtnState**。而且我们定义了在鼠标进入按钮区域和离开按钮区域时应该通过一个函数toggleLoginState来改变loginBtnState的值。在LoginComponent中定义这个方法即可。
* 假如不想是一个成员变量。我们可以直接定义为[@loginState]="'**'inactive'**"，也就是说用''把实际值包含。
* 假如在组件中没有给loginBtnState附上初始值，那么动画处于void状态(空状态)，这个时候我们可以通过下面的方法定义给没有初始值时的动画。

```typescript
  animations: [
    trigger('loginState', [
      state('void', style({
        transform: 'scale(1)'
      })),
      ......
```

## 11.Anguar2的组件声明周期  

* ngOnChanges 组件和指令  
    在ngInit之前触发，当Angular设置数据绑定属性或输入性属性时会得到一个包含当前和之前属性值的对象（SimpleChanges)
* ngOninit  组件和指令
    只调用一次，在设置完输入性属性后，通过这个函数初始化组件或指令。
* ngDoCheck 组件和指令
    在ngInit之后，每次检测到变化时触发，可以在此检查一下angular自身无法检查的变化。
* ngAfterContentInit 组件
    在ngDoCheck后触发，只调用一次，把要装载到组件视图的内容初始化。
* ngAfterContentChecked 组件 
    ngAfterContentInit之后每次ngDoCheck都会在之后触发ngAfterContentChecked，对要装载到组件视图的内容进行检查。
* ngAfterViewInit  组件
    在第一个ngAfterContentInit被调用后触发，只调用一次，在angular初始化视图后相应。
* ngAfterViewChecked 组件
    在ngAfterViewInit后以及每个ngAfterContentChecked后触发。
* ngOnDestroy 组件和指令
    在组件或指令被销毁前，清理环境，可以在此处取消Observable的订阅。
    


## 12. RxJS
响应式编程需要描述数据流，而不是单个点的数据变量，我们需要把数据的每个变化汇聚成一个数据流。
map这个操作符做的事情就是允许你对元数据流中的每一个元素应用一个函数，然后返回并形成一个新的数据流，这个数据流中的每一个元素都是原来数据流中的元素应用函数后的值。
```typescript
let todo = document.getElementById('todo');
let input$ = Rx.Observable.fromEvent(todo, 'keyup');
input$.map(ev=>ev.target.value).subscribe(value=>console.log(value));

```
而类似.map(ev=>ev.target.value)的场景太多了，可以用pluck这个专门的操作符来应对。这个操作符可以从一系列嵌套的属性中把值提取出来，形成新的流。
另外subscribe可以有三个函数参数，分别对应于三个状态：next,error和completed。这三个函数就是分别处理这3中状态的。
* 合并类操作符
    1. combineLatest操作符
       是在组合的两个元数据流中选择最新的两个数据进行配对，如果其中一个源之前没有任何数据产生，那么结果流也不会产生数据。
    2. zip操作符
        zip和combineLatest非常像，但重要的区别点在于zip严格需要多个源数据流中的每一个相同顺序的元素配对。
    3. merge操作符
        他把两个流的元素混在一起，合并成一个，完全按各自的实践顺序，不会等待另外一个流，也不会对两个做什么操作，就是简单地合并成一个流。
    4. concat操作符
        严格等待某个流结束后再合并另一个流。
* 创建类操作符
    1. from操作符  
        可以支持从数组，类似数组的对象，Promise,iterable对象或类似Observable的对象（其实这个主要指ES2015中的Observable）来创建一个Observable。
    2. fromEvent操作符
        这个操作符是专门为事件转换成Observable而制作的。
    3. fromEventpattern操作符
    4. defer操作符
        defer是直到有订阅者之后才创建Observable，而且它为每个订阅者都会这样做，也就是说其实每个订阅者都是接到到自己的单独数据流序列。
    5. Interval操作符
        Rx提供内建的可以创建和计时器相关的Observable方法，第一个是Interval，他可以在指定时间间隔发送证书的自增长序列。
    6. Timer操作符
        有2种形式的Timer，一种是指定时间后返回一个序列中只有一个元素（值为0）的Observable。
        第二种类似于Interval，除了第一个参数是一开始的延迟时间，第二个参数是间隔时间，也就是说，在一开始的延迟时间后，每个一段时间就会返回一个整数序列。
* 过滤类操作符
    1. filter
       Filter操作只允许数据流中满足其predicate测试的元素发射出去，这个predicate函数接收3个元素。
        * 原始数据流元素
        * 索引，指该元素在元数据流中的位置（从0开始）
        * 源Observable对象。
    2. debounceTime
        对于一些发射频率比较高的数据流，我们有时会想给它按个”整流器",它可以滤掉指定间隔时间的事件等。
    3. distinct
        可以把重复的元素过滤掉。
    4. skip
        可以跳过几个元素。
* 工具类操作符
* 条件型操作符
* 数学和聚集性操作符
* 连接型操作符
等等。

### 12.1 Subject  
* BehaviorSubject
   BehaviorSubject是Subject的一个衍生类，它保存“最近一个值”
* ReplaySubject 
    ReplaySubject可以提供n个最近的值。
这儿我们需要在页面（Component）销毁时取消订阅，那么我们利用生命周期的OnDestroy来完成。
```typescript
  ngOnDestroy(){
    if(this.subscription !== undefined)
      this.subscription.unsubscribe();
  }
```
更好的方法请参考Async管道。
### 12.2 Async管道
有了这个管道，我们**无需管理**琐碎的**取消订阅**，以及**订阅**了。
```typescript
<p>
{{clock | async}
</p>
```
下面直接上例子
* 在Service层

```typescript
//todo.service.ts
@Injectable()
export class TodoService {
  private _todos: BehaviorSubject<Todo[]>;
  private dataStore: {  // This is where we will store our data in memory
    todos: Todo[]
  };
    constructor(private http: Http, @Inject('auth') private authService) {
        this.authService.getAuth()
          .filter(auth => auth.user != null)
          .subscribe(auth => this.userId = auth.user.id);
        this.dataStore = { todos: [] };
        this._todos = new BehaviorSubject<Todo[]>([]);
      } 
  get todos(){
    return this._todos.asObservable();
  }
  // POST /todos
  addTodo(desc:string){
    let todoToAdd = {
      id: UUID.UUID(),
      desc: desc,
      completed: false,
      userId: this.userId
    };
    this.http
      .post(this.api_url, JSON.stringify(todoToAdd), {headers: this.headers})
      .map(res => res.json() as Todo)
      .subscribe(todo => {
        this.dataStore.todos = [...this.dataStore.todos, todo];
        this._todos.next(Object.assign({}, this.dataStore).todos);
      });
  }
}
//todo.component.ts
@Component({
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.css']
})
export class TodoComponent implements OnInit {

  todos : Observable<Todo[]>;
    constructor(
      @Inject('todoService') private service,
      private route: ActivatedRoute,
      private router: Router) {}
  
    ngOnInit() {
      this.route.params
        .pluck('filter')
        .subscribe(filter => {
          this.service.filterTodos(filter);
          this.todos = this.service.todos;
        })
    }
  addTodo(desc: string) {
    this.service.addTodo(desc);
  }
```
我们组件(todo.component.ts)中的todos变成了一个Observable，在构造是直接把Service的属性方法todos赋值上去了。这样改造后，我们只需改动模板的两行代码就大功告成了，那就是替换原有的="todos..."为
="todos | async"。请注意这个通道(Pipe)是需要直接应用于Observable的，也就是说如果我们要显示todos.length，我们应该写成(todos|async).length。
相对应的模板(todo.component.html)片段如下：
```typescript
<div>
  <app-todo-header
    placeholder="What do you want"
    (onEnterUp)="addTodo($event)" >
  </app-todo-header>
  <app-todo-list
    [todos]="todos | async"
    (onToggleAll)="toggleAll()"
    (onRemoveTodo)="removeTodo($event)"
    (onToggleTodo)="toggleTodo($event)"
    >
  </app-todo-list>
  <app-todo-footer
    [itemCount]="todos?.length | async"
    (onClear)="clearCompleted()">
  </app-todo-footer>
</div>
```

## 13. redux
应用开发中，UI上显示的数据，控件状态，登陆状态等等全部可以看作是状态。应用程序开发中最复杂的部分就在于这些状态的管理。
Redux提出了几个概念：Store,Reducer,Action

* Store
Store是一个应用内的数据（状态）中心。Store在Redux中有一个基本原则：他是一个“唯一的，状态不可修改”的树，状态的更新只能通过显性定义的Action发送后触发。
Store中一般负责：保存应用状态，提供访问状态的方法，派发Action的方法以及对于状态订阅者的注册和取消等。

* Reducer
Reducer的作用是接收一个状态和对应的处理(Action)，进行处理后返回一个新状态。
Reducer是一个纯JavaScript函数，接收2个参数：第一个参数是处理之前的状态，第二个参数是一个可能携带数据的动作(Action)。
Reducer的TypeScript定义为：
```typescript
export interface Reducer<T> {
    (state: T, action:Action): T;
}
```
* Action
Action让Redux和Store之间通信。在Redux规范中，所有会引发状态更新的交互行为都必须通过一个显性定义的Action来进行。
Action的TypeScript定义为：
```typescript
export interface Action{
    type: string;
    payload?: any;
}
```

ngrx是一套利用RxJS的类库，其中的@ngrx/store(https://github.com/ngrx/store)就是基于Redux规范指定的Angular2框架。

### 13.1 利用方法
* 在项目根目录下安装npm install @ngrx/core @ngrx/store --save。
* 在CoreModule中引入StoreModule,StoreDevtoolsModule
```typescript
import { StoreModule } from '@ngrx/store';
import { todoReducer, todoFilterReducer } from '../reducers/todo.reducer';
import { authReducer } from '../reducers/auth.reducer';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
@NgModule({
  imports:[
    HttpModule,
    StoreModule.provideStore({ 
      todos: todoReducer, 
      todoFilter: todoFilterReducer,
      auth: authReducer
    }),
    StoreDevtoolsModule.instrumentOnlyWithExtension()
  ],
```
* 而后定义自己的todoReducer和todoFilterReducer
* 在组件中引入Store,并且在适当的时候dispatch ADD_TODO这个Action就OK了。
```typescript
import { Store } from '@ngrx/store';
export class TodoComponent {

  todos : Observable<Todo[]>;

  constructor(
    private service: TodoService,
    private route: ActivatedRoute,
    private store$: Store<AppState>) {
      const fetchData$ = this.service.getTodos()
        .flatMap(todos => {
          this.store$.dispatch({type: FETCH_FROM_API, payload: todos});
          return this.store$.select('todos')
        })
        .startWith([]);
```