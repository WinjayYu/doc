
* net原理介绍

http://www.cnblogs.com/hustskyking/p/nodejs-net-module.html

## http模块

http://blog.csdn.net/gxhacx/article/details/12433285

### server部分

#### createServer
 
createServer方法获取Server对象,基于事件的通信机制

* request事件

        回调返回ServerRequest、ServerResponse
        ServerRequest : 分析用户请求的内容
            url\httpVersion\header\method\statusCode等属性
            分析post请求体是会用到    
                data事件
                end
                close
        ServerResponse : 
            writeHeader
            write
            end

* connection事件

        返回socket对象

* close事件

### client部分

request\get 方法返回ClientRequest对象

options参数控制请求的格式、内容等等

    hostname
    port
    path
    header
    method

#### ClientRequest

类似于ServerResponse对象(需要发送内容)

* write 通常post、put请求发送请求体
* end 必须调用end说明请求发送结束

* abort 终止请求

#### ClientResponse

类似于ServerRequest用于接受内容

事件

    data
    end
    close

函数
    
    setEncoding
    