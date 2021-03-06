# 描述

基于阿里云DNS解析的`Let's Encrypt` 一键式申请管理SSL脚本。

众所周知（划掉）SSL只需要申请一个证书即可，这个证书包含了根域名及泛域名，但是申请`Let's Encrypt`泛域名需要DNS验证，目前已经有很多脚本可以支持阿里云&腾讯云DNS解析了，但是感觉那些脚本配置略麻烦，所以写了一个小脚本能够通过json配置化的方式来一键申请SSL，也方便管理。

注意：
- 本脚本依赖阿里云
- NODE版本>=10 (支持async语法即可)
- 本脚本目前不支持定时任务（下个版本可能会加）所以你得2个月5天-3个月之内的时候手动执行一下，建议收到邮件的时候执行一下`npm run start`就行了，邮件会提前10天通知你的（账户邮箱）。


如果对ACME比较感兴趣可以参考 [letsencrypt的ACME规范开发折腾记](https://zhuanlan.zhihu.com/p/73981808)

如果对软件开发感兴趣可[来找我定制软件呀](https://www.airio.cn/)

# 安装

1. ```npm install```
2. 重命名`conf-default.json`为`conf.json`
3. 更改`conf.json`配置文件 建议看完下面的conf配置变量说明再做变更
4. 请一定要更改`contact`数组里的联系方式


注意 
程序会自动生成`Let's Encrypt`管理账号 所以如果误填了contact的联系方式，只需要把`accountUrl`和当前项目目录下的`account.pem`删除即可

# conf配置变量说明

```
{
  "confData": [ // 证书配置目录
    {
      "name": "xx域名相关", // 相关描述
      "ali": {  // 阿里云key配置
        "accessKeyId": "accessKeyId",
        "accessKeySecret": "accessKeySecret",
        "domain": "airio.cn", // 当前域名信息，需要该域名出现在该账号的 域名管理列表里
        "cdn":{ // 如果需要配置cdn自动上传则添加此字段 如果不需要移除此字段即可
          "domain": ["xx.cn","www.xx.cn"] // 当前配置的cdn域名 需要域名列表中的域名出现在该账号的管理域名内
        }
      },
      "contact": [ // 联系方式 目前该字段无实际用处 你可以把下面的邮箱改为自己的信息
        "mailto:soul@xx.cn" 
      ],
      "identifiers": [ // 需要申请SSL域名的列表 建议只申请两个 一个根域名 一个泛域名即可
        "xx.cn",
        "*.xx.cn"
      ],
      "cName": "xx.cn", // 重要请填写您的根域名
      
      "sftp": { // 服务器sftp配置，目前只支持账号密码鉴权
        "host": "host", // 服务器host
        "username": "username", // 服务器账号
        "password": "password", // 服务器密码
        "uploadPath": "/etc/letsencrypt-ssl-list", // 如果你设置了服务器sftp信息 会自动把证书上传至该目录
        "execShell": "lnmp restart", // 上传完毕后 自动执行的命令
      },
      
      "time": "2020-12-10" // 这里为更新时间，let's ssl为3个月过期 目前根据此字段判断是否过期，如果你想提前获取或者重新获取 更改这个时间 距离目前本地的时间超过3个月即可
    }
  ],
  "accountUrl": false, // 配置的账户信息 默认如果没有let's ssl账号的话会自动生成
  "contact": [ // 重要！！ 域名到期的时候会发送域名到期信息到此邮箱
    "mailto:soul@xx.cn" // 注意需要加mailto:前缀
  ],
  "sleepTime": 300000 // 5分钟后再进行域名校验
}

```

# 运行

### 获取证书&自动推送本地证书至阿里云

如果你未配置`ali.cdn`则不会自动推送

```
npm run start
```

### 推送本地证书至阿里云
需要你配置`ali.cdn`

```
npm run pushaly
```


# 常见问题

Q: 为什么我申请一个域名执行了那么久？

A: DNS的TXT验证时效一直是个谜，这里是等待了300000（5分钟）进行校验，如果5分钟后还是没有成功建议把这个数值调大，比如400000（单位毫秒），所以根据域名列表的数量，基本上执行后就可以后台化喝杯咖啡了 再回来看结果

Q: 中途报错怎么办？

A: 很简单，只要是没有生成cert 那么重来即可 重新执行 `npm run start`，你也可以把错误信息贴到issue里，看见了就会回复（狗头）

Q: 脚本会把信息泄露出去吗？

A: 你自己别作死泄露就行了，纯本地的放心好了

# 版本

### 下一步规划

- [ ] sftp支持pem鉴权
- [ ] 定时任务调用
- [ ] 支持腾讯云等其他云DNS解析
- [ ] 抽离&整理代码

### v0.1

- [x] 自动申请&配置Let's Encrypt 管理账号
- [x] 自动配合阿里云DNS解析 直接申请泛域名&根域名
- [x] 生成的证书自动push至服务器
- [x] push至服务器后自动执行对应的shell脚本
- [x] 生成的证书自动push至阿里云CDN域名列表，前提是你的手动去开通阿里云CDN控制台和添加对应的域名
