# sequelize 튜토리얼 강좌

# 목차

## 1. sequelize란

## 2. sequelize 세팅
    1. 설치 
    2. sequelize를 mysql에 적용해보기
    3. (추가)connection pool

## 3. 테이블 모델링
    1. 모델링 옵션 넣기
## 4. db와 생성한 모델 동기화 하기
    1. 한번에 모든 모델들을 동기화 하기
    2. (추가) 배포할 때
## 5. sql 쿼리
## 6. promise와 async/await
--- 

### 1. Sequelize란

프로미스(promise)를 기반으로한 node의 ORM 패키지입니다.  
PostgreSQL, MySQL, SQLite and MSSQL를 지원합니다.

- ORM이란 객체와 관계를 서로 매핑해주는 것이라고 생각하면 됩니다.  
  => 즉, JS의 객체들을 테이블로 바꿔주는 행위라고 생각합시다.

---

### 2. Sequelize 세팅
#### 1. 설치 
`npm init`

`npm i sequelize`

`npm i mysql2`

node는 12.7.0 버전을 사용합니다.
sequelize는 5.18.3 버전을 사용합니다.
mysql2는 1.7.0 버전을 사용합니다.

---

#### 2. Sequelize를 mysql에 적용해 보기

우선 mysql db에 연결하기 전에, 우리는 sequelize instance를 생성해야 하겠죠?  
생성 방법은 두가지로 나뉩니다.

1. constructor(생성자)에 parameter(매개변수)전달해주기

```
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'
});
```

2. 그냥 uri를 전달해주기

```
const Sequelize = require('sequelize');
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

#### 3. connection pool
만약에 우리가 하나의 프로세스로 데이터베이스와 연결하려고 한다면, sequelize 객체를 하나만 생성해도 좋아요.  
sequelize객체를 생성할 때 connection pool 또한 이니셜라이즈 됩니다.  
connection pool은 constructor의 옵션에서 설정할 수 도 있습니다. 

예) 
```
const sequelize = new Sequelize(/* ... */, {
  // ...
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```  

#### 4. connection 테스트 해보기  
```.authenticate()``` 함수를 이용해서 커넥션을 테스트 할 수 있습니다.  
```
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```

#### 5. connection 종료
기본값으로 sequelize는 하루 죙~일 열려있습니다.  
```seqeulie.close()```를 사용해 종료할 수 있습니다.

---
### 3. 테이블 모델링

모델은 ```sequelize.Model```을 extends 한 클래스입니다.  
두가지 동일한 방법으로 모델들을 정의할 수 있는데요,  
1. ```Sequelize.Model.init(attributes, options)```로 정의하기.
```
const Model = Sequelize.Model;
class User extends Model {}
User.init({
  // attributes
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull defaults to true
  }
}, {
  sequelize,
  modelName: 'user'
  // options
});
```

2. ```sequelize.define```로 정의하기  
```
const User = sequelize.define('user', {
  // attributes
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull defaults to true
  }
}, {
  // opt
```
내부적으로 ```sequelize.define```은 ```Model.init```을 호출합니다.  

위에있는 코드들은 table 이름을 자동적으로 **users**로 바꿔줍니다.   
자동으로 복수형으로 바꾸는데, 이게 싫으면 옵션을 추가해주면 돼요
```freezeTableName: true```  
자동으로 sequelize는
```id```(primary key),
```createdAt```,
```updatedAt```을 모든 모델에 생성해줍니다.

#### 1. 모델링 옵션 넣기
sequelize 생성자는 define 옵션을 갖고있어서 모든 모델에 옵션을 적용할 수 있습니다.
```
const sequelize = new Sequelize(connectionURI, {
  define: {
    // The `timestamps` field specify whether or not the `createdAt` and `updatedAt` fields will be created.
    // This was true by default, but now is false by default
    timestamps: false
  }
});

// Here `timestamps` will be false, so the `createdAt` and `updatedAt` fields will not be created.
class Foo extends Model {}
Foo.init({ /* ... */ }, { sequelize });

// Here `timestamps` is directly set to true, so the `createdAt` and `updatedAt` fields will be created.
class Bar extends Model {}
Bar.init({ /* ... */ }, { sequelize, timestamps: true });
```
