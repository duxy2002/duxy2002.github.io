# cannot find module node_modules\css-loader\index.js?sourcemap&minimize'

The "ng build --prod" works for me now with the list posted by castamir above, thanks for everyone in this thread.  

```
{
 "@ngtools/webpack": "1.1.5", 
 "extract-text-webpack-plugin": "2.0.0-beta.5",  
 "webpack": "2.1.0-beta.25",  
 "loader-runner": "2.2.0",
 "tsickle": "0.2.4",
 "source-map-support": "0.4.10",
"angular-cli": "1.0.0-beta.21"
 }
``` 



NOTE: I didn't have to even install angular-cli on my npm, but I am just declaring it in the package.json as shown above. I uninstalled my angular cli using "npm uninstall -g angular-cli" and "npm cache clean". Now I am able to build prod using package.json script task itself that runs the angular-cli based build with in the local confines of package.json and not at global level installed angular-cli.
"scripts": {
         "build:prod": "ng build --prod"
}

