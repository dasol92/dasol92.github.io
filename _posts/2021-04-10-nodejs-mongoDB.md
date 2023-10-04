---
title:  "NodeJS(Express), MongoDB and Rest API"
categories: 
  - node.js
tags:
  - node.js
  - Express
  - mongoDB
  - mongoose
  - RestAPI
---


NodeJS 로 MongoDB 를 연결하기 위한 모듈 `mongoose` 를 설치한다. 
```
npm install express mongoose
```

가장 간단하게 연결하는 방법은 아래와 같다.
```js
const mongoose = require('mongoose');
const MONGDB_URL = 'mongodb+srv://id:pwd@url';

mongoose.connect(MONGDB_URL, {useNewUrlParser: true, useUnifiedTopology: true}, (err)=>{
    if (err) {
        console.log(err);
    }
    else {
        console.log('Connected to databases successfully');
    }
});
```

실행 결과
```
Connected to databases successfully
```

하지만 위와 같이 코드 안에 mongoDB URL 을 포함하게 되면 코드를 공유할 때 URL 이 그대로 노출 되므로, 별도로 관리하는 것이 좋다. 


따라서 별도의 환경 파일 (`variables.env`)에 URL을 작성하고, 이를 `dotenv` 모듈이 읽어오는 방식으로 진행한다. 

`variables.env` 파일을 아래와 같이 작성한다. 
```
MONGODB_URL=mongodb+srv://id:pwd@url
```

`dotenv` 모듈을 설치한다. 
```
npm install dotenv
```


모듈을 통해서 변수 파일을 읽어오고, 콘솔에 출력해보면 URL 을 잘 읽어온 것을 확인할 수 있다. 
```js
require('dotenv').config({path:'variables.env'});
console.log(process.env.MONGODB_URL);
```


따라서 이 방식으로 코드 상에서 연결을 시키려면 아래와 같이 작성한다. 

```js
const mongoose = require('mongoose');
require('dotenv').config({path:'variables.env'});

mongoose.connect(process.env.MONGODB_URL, {useNewUrlParser: true, useUnifiedTopology: true}, (err)=>{
    if (err) {
        console.log(err);
    }
    else {
        console.log('Connected to databases successfully');
    }
});
```


연결이 잘 되는 것을 확인했으니, 실제 DB 데이터를 활용해보자. 

### Schema

먼저 데이터의 schema 를 정의해야 한다. 

별도의 파일 (`models/User.js`) 에 스키마를 정의한다. 
```js
const mongoose = require('mongoose');
// const Schema = mongoose.Schema;
const { Schema } = mongoose; // Same with above line; Object Destructuring


const userSchema = new Schema(
    {
        email:{
            type:String,
            required:true,
        },
        name:String,
        age:{
            type:Number,
            min: 18,
            max: 65
        }
    },
    {
        timestamps: true // TimeStamping when data updates
    }
);

module.exports = mongoose.model('User', userSchema);
```

### Insert Data

앞서 정의한 스키마 모듈을 import 하고, 테스트용으로 Get 메소드에서 데이터를 추가한다.
```js
server.get('/', (req,res)=>{
    const newUser = new User();
    newUser.email = "danny@example.com";
    newUser.name = "Danny";
    newUser.age = 30;
    newUser.save()
        .then((user)=>{
            console.log(user);
            res.json({
                message: 'User Created Successfully'
            });
        })
        .catch((err)=>{
            res.json({
                message: 'User was not successfully created'
            });
        });
});
```
