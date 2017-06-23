---
layout:     post
title:      "Nodejs 实现基于 Token 的认证应用"
subtitle:   "Nodejs 实现基于 Token 的认证应用"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - nodejs
---

[推荐阅读](https://github.com/FrontendMagazine/Works/blob/master/archive/token-based_authentication_with_angularjs_%26_nodejs.md2)

## JWT

JWT 代表 JSON Web Token ，它是一种用于认证头部的 token 格式。这个 token 帮你实现了在两个系统之间以一种安全的方式传递信息。

## 实例

我们一般不直接使用 MongoDB 的函数操作数据库，而使用 `Mongoose` 来操作 MongoDB([参考手册](https://cnodejs.org/topic/548e54d157fd3ae46b233502))。

### 创建用户 Model

`src/models/user.js`

```
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var bcrypt = require('bcrypt');     // 第三方的密码加密库

var UserSchema = new Schema({
  name: {
    type: String,
    unique: true, // 是否唯一
    require: true, // 是否为空
  },
  password: {
    type: String,
    require: true
  },
  token: {
    type: String
  }
})

// 添加用户保存时中间件对password进行bcrypt加密,这样保证用户密码只有用户本人知道
UserSchema.pre('save', function (next) {
  var _this = this

  if (this.isModified('password') || this.isNew) {
    bcrypt.genSalt(10, function (err, salt) {
      if (err) {
        return next(err)
      }
      bcrypt.hash(_this.password, salt, function (err, hash) {
        if (err) {
          return next(err)
        }
        _this.password = hash
        next()
      })
    })
  } else {
    return next()
  }
})

// 校验用户输入密码是否正确
UserSchema.methods.comparePassword = function (passw, cb) {
  bcrypt.compare(passw, this.password, function (err, isMatch) {
    if (err) {
      return cb(err)
    }
    cb(null, isMatch)
  })
}

var UserModel = mongoose.model('User', UserSchema)


module.exports = UserModel

```

### 链接数据库

`src/app.js`

```
var mongoose = require("mongoose");
var config = require('./config/default');

mongoose.connect(config.db); // 连接数据库
```

`src/config/default`

```
module.exports = {
  'secret': 'LyfAccoutApp',
  'db': 'mongodb://localhost:27017/accoutapp'
}
```

### 配置 HTTP 请求

我们需要允许不同网站的请求，解决跨域请求的错误。

- `Access-Control-Allow-Origin` 允许所有的域名
- 你可以向这个设备发送 `POST` 和 `GET` 请求
- 允许 `X-Requested-With` 和 `content-type` 头部

`src/app.js`

```
app.use(function(req, res, next) {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST');
    res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With,content-type, Authorization');
    next();
});
```

### 路由处理

`src/app.js`

```
var index = require('./routes/index');
var user = require('./routes/user');

app.use('/', index);
app.use('/api/user', user);
```

`src/routes/user.js`

```
var express = require('express')
var User = require('../models/user')
var jwt = require('jsonwebtoken')
var config = require('../config/default')
var router = express.Router();


// 注册用户
router.post('/signup', function (req, res) {
  if (!req.body.name || !req.body.password) {
    res.json({
      success: false,
      message: '请输入您的账号密码。'
    })
  } else {
    var newUser = new User({
      name: req.body.name,
      password: req.body.password
    })

    // 保存用户账号
    newUser.save(function (err) {
      if (err) {
        return res.json({
          success: false,
          message: '注册失败!'
        })
      }
      res.json({
        success: true,
        message: '成功创建新用户!'
      })
    })
  }
})

// 检查用户名和密码生成一个 accesstoken 如果验证通过
router.post('/accesstoken', function (req, res) {
  User.findOne({
    name: req.body.name
  }, function (err, user) {
    if (err) {
      throw err;
    }

    // 用户不存在
    if (!user) {
      res.json({
        success: false,
        message: '认证失败，用户不存在！'
      })
    } else if (user) {
      // 检查密码是否正确
      user.comparePassword(req.body.password, function (err, isMatch) {
        if (isMatch && !err) {
          var token = jwt.sign(
            {
              name: user.name
            },
            config.secret,
            {
              expiresIn: 3600
            }
          )

          user.token = token
          user.save()

          res.json({
            success: true,
            message: '验证成功!',
            token: 'Bearer ' + token,
            name: user.name
          })
        } else {
          res.json({
            success: false,
            message: '认证失败，密码失败！'
          })
        }
      })
    }
  })
})

// 获取用户信息
router.get('/info', ensureAuthorized, function(req, res) {
    User.findOne({token: req.token}, function(err, user) {
        if (err) {
            res.json({
                success: false,
                data: "Error occured: " + err
            });
        } else {
            res.json({
              username: user.name
            });
        }
    });
});

// 验证 Authorization
function ensureAuthorized (req, res, next) {
  var bearerToken;
  var bearerHeader = req.headers['authorization']
  console.log(req.headers)
  if (typeof bearerHeader !== 'undefined') {
    var bearer = bearerHeader.split(" ")
    bearerToken = bearer[1]
    req.token = bearerToken
    next()
  } else {
    res.status(401).json({
      success: false,
      message: 'Unauthorized'
    })
  }
}

module.exports = router;
```
