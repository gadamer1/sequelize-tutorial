# sequelize 튜토리얼 강좌

# 목차

## 1. sequelize란

## 2. sequelize 세팅
    1. [설치](#1.-설치) 
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
```sequelize.close()```를 사용해 종료할 수 있습니다.

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
자세한 옵션들은 [이곳](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-define)에서 확인해주세요.

---

### 4. db와 생성한 모델 동기화 하기
sequelize가 우리가 정의한 모델들을 바로 db와 동기화 시키고 싶으면, ```then()```메소드를 사용합시다!
```
// Note: using `force: true` will drop the table if it already exists
User.sync({ force: true }).then(() => {
  // Now the `users` table in the database corresponds to the model definition
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```
#### 1. 한번에 모든 모델들을 동기화 하기
각 모델마다 ```then()```을 통해 동기화 하지말고, sequelize 객체에 ```sync()```메소드를 사용해 한꺼번에 동기화 시킵시다!
```sequelize.sync()```
#### 2. (추가) 배포할 때 
배포환경에서는 ```sync()```대신에 Migrations를 고려해봄직 합니다.  [이곳](https://sequelize.org/master/manual/migrations.html)에서 확인해보세요.


---

### 5. sql 쿼리 
#### 1. attributes 
attributes 옵션을 통해 원하는 column 값만 가져올 수 있습니다.
```
Model.findAll({
  attributes: ['foo', 'bar']
});
```
```
SELECT foo, bar ...
```
aggregation또한 ```sequelize.fn```을 통해 사용가능합니다.
```
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});
```
```
SELECT COUNT(hats) AS no_hats ...
```
aggregation을 사용하려면 별명을 부여해야 합니다.
위의 코드에서는 ```instance.get('no_hats')를 통해 모자의 개수를 얻을 수 있습니다.  
<br>
<br>
물론 exclude를 통해 제외 시켜버릴 수도 있습니다.
```
Model.findAll({
  attributes: { exclude: ['someColumn'] }
});
```
---

#### 2.Where  
 find/findAll 이나 updates/destroy등의 질의를 할때 ```where```객체를 넣어서 필터링을 할 수 있습니다.
 ```
 const Op = Sequelize.Op;

Post.findAll({
  where: {
    authorId: 2
  }
});
// SELECT * FROM post WHERE authorId = 2

Post.findAll({
  where: {
    authorId: 12,
    status: 'active'
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';

Post.findAll({
  where: {
    [Op.or]: [{authorId: 12}, {authorId: 13}]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.findAll({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.destroy({
  where: {
    status: 'inactive'
  }
});
// DELETE FROM post WHERE status = 'inactive';

Post.update({
  updatedAt: null,
}, {
  where: {
    deletedAt: {
      [Op.ne]: null
    }
  }
});
// UPDATE post SET updatedAt = null WHERE deletedAt NOT NULL;

Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('status')), 6)
});
// SELECT * FROM post WHERE char_length(status) = 6;
 ```
#### operator
```Sequelize.Op```를 통해 더욱 정교한 비교가 가능해집니다.
```
const Op = Sequelize.Op

[Op.and]: {a: 5}           // AND (a = 5)
[Op.or]: [{a: 5}, {a: 6}]  // (a = 5 OR a = 6)
[Op.gt]: 6,                // > 6
[Op.gte]: 6,               // >= 6
[Op.lt]: 10,               // < 10
[Op.lte]: 10,              // <= 10
[Op.ne]: 20,               // != 20
[Op.eq]: 3,                // = 3
[Op.is]: null              // IS NULL
[Op.not]: true,            // IS NOT TRUE
[Op.between]: [6, 10],     // BETWEEN 6 AND 10
[Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
[Op.in]: [1, 2],           // IN [1, 2]
[Op.notIn]: [1, 2],        // NOT IN [1, 2]
[Op.like]: '%hat',         // LIKE '%hat'
[Op.notLike]: '%hat'       // NOT LIKE '%hat'
[Op.iLike]: '%hat'         // ILIKE '%hat' (case insensitive) (PG only)
[Op.notILike]: '%hat'      // NOT ILIKE '%hat'  (PG only)
[Op.startsWith]: 'hat'     // LIKE 'hat%'
[Op.endsWith]: 'hat'       // LIKE '%hat'
[Op.substring]: 'hat'      // LIKE '%hat%'
[Op.regexp]: '^[h|a|t]'    // REGEXP/~ '^[h|a|t]' (MySQL/PG only)
[Op.notRegexp]: '^[h|a|t]' // NOT REGEXP/!~ '^[h|a|t]' (MySQL/PG only)
[Op.iRegexp]: '^[h|a|t]'    // ~* '^[h|a|t]' (PG only)
[Op.notIRegexp]: '^[h|a|t]' // !~* '^[h|a|t]' (PG only)
[Op.like]: { [Op.any]: ['cat', 'hat']}
                           // LIKE ANY ARRAY['cat', 'hat'] - also works for iLike and notLike
[Op.overlap]: [1, 2]       // && [1, 2] (PG array overlap operator)
[Op.contains]: [1, 2]      // @> [1, 2] (PG array contains operator)
[Op.contained]: [1, 2]     // <@ [1, 2] (PG array contained by operator)
[Op.any]: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)

[Op.col]: 'user.organization_id' // = "user"."organization_id", with dialect specific column identifiers, PG in this example
[Op.gt]: { [Op.all]: literal('SELECT 1') }
                          // > ALL (SELECT 1)
```
#### 범위 형식의 operator 
```
// All the above equality and inequality operators plus the following:

[Op.contains]: 2           // @> '2'::integer (PG range contains element operator)
[Op.contains]: [1, 2]      // @> [1, 2) (PG range contains range operator)
[Op.contained]: [1, 2]     // <@ [1, 2) (PG range is contained by operator)
[Op.overlap]: [1, 2]       // && [1, 2) (PG range overlap (have points in common) operator)
[Op.adjacent]: [1, 2]      // -|- [1, 2) (PG range is adjacent to operator)
[Op.strictLeft]: [1, 2]    // << [1, 2) (PG range strictly left of operator)
[Op.strictRight]: [1, 2]   // >> [1, 2) (PG range strictly right of operator)
[Op.noExtendRight]: [1, 2] // &< [1, 2) (PG range does not extend to the right of operator)
[Op.noExtendLeft]: [1, 2]  // &> [1, 2) (PG range does not extend to the left of operator)
```
#### 조합을 통해 복잡한 비교문 간단히 생성!
```
const Op = Sequelize.Op;

{
  rank: {
    [Op.or]: {
      [Op.lt]: 1000,
      [Op.eq]: null
    }
  }
}
// rank < 1000 OR rank IS NULL

{
  createdAt: {
    [Op.lt]: new Date(),
    [Op.gt]: new Date(new Date() - 24 * 60 * 60 * 1000)
  }
}
// createdAt < [timestamp] AND createdAt > [timestamp]

{
  [Op.or]: [
    {
      title: {
        [Op.like]: 'Boat%'
      }
    },
    {
      description: {
        [Op.like]: '%boat%'
      }
    }
  ]
}
// title LIKE 'Boat%' OR description LIKE '%boat%'
```
#### relation 쿼리
```
// Find all projects with a least one task where task.state === project.state
Project.findAll({
    include: [{
        model: Task,
        where: { state: Sequelize.col('project.state') }
    }]
})
```

#### Pagination / Limiting
```
// Fetch 10 instances/rows
Project.findAll({ limit: 10 })

// Skip 8 instances/rows
Project.findAll({ offset: 8 })

// Skip 5 instances and fetch the 5 after that
Project.findAll({ offset: 5, limit: 5 })
```

#### 순서 마음대로 정렬하기(ordering)

```
Subtask.findAll({
  order: [
    // Will escape title and validate DESC against a list of valid direction parameters
    ['title', 'DESC'],

    // Will order by max(age)
    sequelize.fn('max', sequelize.col('age')),

    // Will order by max(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // Will order by  otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // Will order an associated model's created_at using the model name as the association's name.
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's created_at using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using the name of the association.
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using an association object. (preferred method)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using association objects. (preferred method)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at simple association objects.
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
 ]

  // Will order by max age descending
  order: sequelize.literal('max(age) DESC')

  // Will order by max age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.fn('max', sequelize.col('age'))

  // Will order by age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.col('age')

  // Will order randomly based on the dialect (instead of fn('RAND') or fn('RANDOM'))
  order: sequelize.random()
})
```

#### Index Hints
```indexHints```는 부가적으로 query에 index hints를 부여할 수 있습니다.  
hint의 타입은 ```Sequelize.IndexHints```여야 하며, 현재 존재하고 있는 index여야 합니다.
```
Project.findAll({
  indexHints: [
    { type: IndexHints.USE, values: ['index_project_on_name'] }
  ],
  where: {
    id: {
      [Op.gt]: 623
    },
    name: {
      [Op.like]: 'Foo %'
    }
  }
```
이는 아래와 같은 mysql query와 같습니다.
```
SELECT * FROM Project USE INDEX (index_project_on_name) WHERE name LIKE 'FOO %' AND id > 623;
```
```Sequelize.IndexHints```는 ```USE```, ```FORCE```,```IGNORE```를 가지고 있습니다.

###  6. promise와 async/await
sequelize는 promise들을 광범위하게 사용합니다.   
async await도 사용 가능합니다.   
또한 모든 sequelize의 promise들은 ```bluebird``` 프로미스여서, bluebird API를 적용할 수 있습니다. 
