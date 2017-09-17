# 如何引入第三方类库

1. 如果类库的npm包中含有类型定义文件（查看node_modules/第三方类库 中是否有.d.ts后缀的文件），那么直接使用
    ``npm install --save 要引入包的名称``即可。
2. 如果类库中没有类型定义文件，可先使用
    ``npm install --save 要引入包的名称``正常安装，然后执行
    ``npm install @types/要引入包的名称 --save-dev``。这个命令要在@types/中检索安装类型定义文件。
3. 假如还是找不到类型定义文件，需要手动添加类型定义
3.1.  首先在src/typings.d.ts中写``declare module '要引入包的名称'``  
3.2.  然后在组件中可以这样引入``import * as friendName from '要引入包的名称'``,friendName是一个友好别名，起一个你认为符合你风格的名称就可行。
3.3.  使用时就可以这样调用方法了
``friendName.method();``

