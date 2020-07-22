---
title:  "sequelize init의 models/index.js 설정이 작동하지 않을 때 해결법"
excerpt: "sequelize init의 models/index.js 설정이 작동하지 않을 때 해결한 방법입니다."
toc: true
toc_sticky: true

categories:
  - blog
tags:
  - [back-end, django]
last_modified_at: 2019-07-23T00:24:00
author_profile: true
---


## 문제 상황
{{ page.excerpt }}

```bash
sequelize init
```

sequelize-cli를 통해 sequelize의 기본 구조를 생성하고, models 폴더에 모델을 정의하면 sequelize가 내역을 Database에 반영해줍니다.  

그런데 models/index.js 설정이 작동하지 않아 models 폴더에 작성한 모델이 Database에 반영되지 않을 때, 이를 해결하기 위해 작성한 글입니다.


## 목차
- 모델을 직접 등록하는 방법
- sequelize의 버전을 낮추는 방법

---

### 모델을 직접 등록하는 방법
- `fs`를 사용하는 대신 models/index.js의 설정을 변경하여 모델을 `require()(sequelize, Sequelize)`로 직접 등록합니다.

```js
const Sequelize = require('sequelize');
const env = process.env.NODE_ENV || 'development';
const config = require('../config/config.json')[env];
const db = {};

const sequelize = new Sequelize(
  config.database,
  config.username,
  config.password,
  config,
);
// Model
db.User = require('./user')(sequelize, Sequelize)

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```

### sequelize의 버전을 낮추는 방법
- models 폴더의 모든 파일들을 불러와서 db객체에 모델을 정의하기 위해 `import 메서드`가 models 폴더의 파일을 순회하며 Model을 취합니다. 
- 해당 구문을 사용하기 위해 버전을 낮추니 정상적으로 작동했습니다.

```js
fs.readdirSync(__dirname)
  .filter(file => {
    return (
    file.indexOf('.') !== 0 && file !== basename && file.slice(-3) === '.js'
  );
  })
  .forEach(file => {
  const model = sequelize['import'](path.join(__dirname, file));
  db[model.name] = model;
});
```

```json
{
  "sequelize": "^4.41.0",
  "sequelize-cli": "^5.2.0"
}
```


## 참조
- [victolee님 - Sequelize(1) - sequelize와 sequelize-cli](https://victorydntmd.tistory.com/26)
- [김정환님 - Sequelize Modeling](http://jeonghwan-kim.github.io/sequelize-model/)


최종 수정 시간: {{ page.last_modified_at }}
