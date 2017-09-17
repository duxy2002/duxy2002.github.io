# tsconfig.md
它配置了TypeScript编译器的编译参数，主要的配置参数如下：
* module: 组织代码的方式
* target: 编译目标平台(ES3,ES5, ES6等)
* sourceMap: 把ts文件编译成js文件时，是否生成对应的SourceMap文件；
* emitDecoratorMetadata: 让TypeScript支持为带有装饰器的声明生成元数据
* experimentalDecorators: 是否启用实验性装饰器特性
* typeRoots: 指定第三方库的类型定义文件的存放位置，推荐使用node_modules/@type文件夹。