---
title: "My_first_blog"
date: 2020-05-12T00:14:56+08:00
draft: false
---

#  1. TCP服务搭建
## 1.1 socket
 > 先来粗略了解下socket
  
套接字（socket）是一个抽象层，应用程序可以通过它发送或接收数据，可对其进行像对文件一样的打开、读写和关闭等操作。套接字允许应用程序将I/O插入到网络中，并与网络中的其他应用程序进行通信。网络套接字是IP地址与端口的组合。 (摘自百度百科)

socket用于在两个基于TCP/IP协议的应用程序之间相互通信.最早出现在UNIX系统中,是UNIX系统主要的信息传递方式.在windows系统中，socket称为winsock.

两种形式的socket：流式套接字,对应与TCP协议.
数据报套接字,对应与UDP协议.


![](https://user-gold-cdn.xitu.io/2019/8/9/16c75bff2667eae0?w=500&h=337&f=jpeg&s=31083)

#  2.创建TCP服务端

### server.js(服务端)
```
const net = require("net");
const sever = net.createServer();
// const clients = [];
const users = [];
const types = require("./types");
sever.on("connection", clientSocket => {
  console.log("有连接进来,请注意```");
  // clients.push(clientSocket)
  clientSocket.on("data", data => {
    console.log("监听data事件,有人说:", data.toString());
    data = JSON.parse(data.toString().trim());
    switch (data.type) {
      case types.login:
        if (users.find(item => item.nickName === data.nickName)) {
          return clientSocket.write(
            JSON.stringify({
              type: types.login,
              success: false,
              message: "昵称已存在"
            })
          );
        }
        clientSocket.nickName = data.nickName;
        users.push(clientSocket);
        clientSocket.write(
          JSON.stringify({
            type: types.login,
            success: true,
            message: "登录成功",
            nickName:data.nickName,
            sumUsers: users.length
          })
        );
        users.forEach(user=>{
          if(user!==clientSocket){
            user.write(JSON.stringify({
              type:types.log,
              message:`${data,nickName} 进入聊天室,当前在线用户数${user.length}`
            }))
          }
        })


        break;
      // 群聊天
      case types.broadcast:
         users.forEach(item => {
          item.write(JSON.stringify({
            type:types.broadcast,
            message:data.message,
            nickName:clientSocket.nickName
          }))
        })
        break;
      // 点对点
      case types.p2p:
        const user = users.find(item => item.nickName === data.nickName)
        if(!user){
          return clientSocket.write(JSON.stringify({
            type:types.p2p,
            success:false,
            message:"该用户不存在"
          }))
        }
        console.log('clientSocket.nickName',clientSocket.nickName)
        user.write(JSON.stringify({
          type:types.p2p,
          message:data.message,
          nickName:clientSocket.nickName,
          success:true
        }))

        break;
      default:
        break;
    }
  });
  
  // 离线
  clientSocket.on("end",()=>{
    console.log("有用户离线了~~~")
    const index =users.findIndex(user => user.nickName === clientSocket.nickName)
    if(index !== -1){
      const offlineUser = users[inde]
      users.splice(index,1)
      users.forEach(user=>{
        if(user!==clientSocket){
          user.write(JSON.stringify({
            type:types.log,
            message:`${offlineUser,nickName} 离开了聊天室,当前在线用户数${user.length}`
          }))
        }
      })
    }
  })
  // clientSocket.write('hello,返回的是buffer,用tostring转一下哦')
});

sever.listen(2000, () => {
  console.log("server running  127.0.0.1 2000");
});

```
#  3.创建客户端
### client.js(客户端)
```
const net = require("net");
const types = require("./types");
let nickName = null;
const client = net.createConnection({
  host: "127.0.0.1",
  port: 2000
});

client.on("connect", () => {
  console.log("连接成功了~~~");
  process.stdout.write("请输入昵称:");

  // 连接完毕后,可以监听终端的信息,发给服务端
  process.stdin.on("data", data => {
    data = data.toString().trim();
    console.log("nickName", nickName);
    if (!nickName) {
      client.write(
        JSON.stringify({
          type: types.login,
          nickName: data
        })
      );
    }
    const matches = /^@(\w+)\s(.+)$/.exec(data);
    if (matches) {
      //符合 @xxx xxx  格式
      return client.write(
        JSON.stringify({
          type: types.p2p,
          nickName: matches[1],
          message: matches[2]
        })
      );
    }
    //群聊天
    client.write(
      JSON.stringify({
        type: types.broadcast,
        message: data
      })
    );
  });
});

client.on("data", data => {
  // console.log("服务端发来的data:::", data.toString());
  data = JSON.parse(data.toString().trim());
  switch (data.type) {
    case types.login:
      if (!data.success) {
        console.log("登录失败", `${data.message}`);
        process.stdout.write("请输入昵称");
      } else {
        process.stdout.write("登录成功,当前在线人数:", data.sumUsers);
        nickName = data.nickName;
      }
      break;
    case types.broadcast:
      console.log(`${data.nickName}:${data.message}`);
      break;
    case types.p2p:
      if (!data.success) {
        return console.log(`发送失败:${data.message}`);
      }
      console.log(`${data.nickName}对你说:${data.message}`);
      break;
    case types.log:
      console.log(`${data.message}`);
      break;
    default:
      console.log("未知消息类型哦~");
      break;
  }
});

```
### types.js
```
module.exports = {
  login: 0,
  broadcast: 1,
  p2p: 2,
  log: 3
};
```

# 总结
- 通过net模块建立TCP服务
- TCP必须建立连接(3次握手)后才能通信
- socket通信模型
- 和使用其他node模块(如koa)一样的思路,都是先建立服务(server),指定端口号

## 新手一个,欢迎指正批评~~