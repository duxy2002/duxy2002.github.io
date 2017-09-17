# typings ERR! message Unable to connect to "https://api.typings.org/entries/dt/es6-shim/tags/0.31.2%2B20160602141504"

## Configuration

If you have special connection requirements, you can setup your connection configuration with a  .typingsrc  file. Typings looks for it in the user's home directory (eg:  c:\Users\myName\  on Windows,  $HOME  /  ~  on Linux), and in the current working directory. More details on setting up rc configuration files can be found from the rc package.


## Example configuration

This  .typingsrc  file shows how to disable SSL when connecting from behind a corporate firewall.
```
 registryURL=http://api.typings.org/
 rejectUnauthorized=false
```

 .typingsrc  also supports json format. The above settings can be written as:
```
{
  "rejectUnauthorized": false,
  "registryURL": "http://api.typings.org/"
}
```

### proxy 

A HTTP(s) proxy URI for outgoing requests. Setting this sets both httpProxy and httpsProxy if they are not set, as this is used as a fallback variable for both  httpProxy  and  httpsProxy . An example setting might be,  "http://127.0.0.1:8888/"  or a real proxy from incloak.com/proxy-list/. If your proxy requires authentication, you may need to include a username and password in your url:  "http://domain\\myusername:password@myproxyServer:port" , or if you are not on an AD domain:  "http://username:mypassword@myProxyServer:port"  

### rejectUnauthorized
Reject invalid SSL certificates (default:  true ). Useful behind (corporate) proxies that act like man-in-the middle on https connections.




## URL
[FAQ](https://github.com/typings/typings/blob/master/docs/faq.md#configuration)