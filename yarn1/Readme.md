# yarn1을 사용해 모노레포 구성하기

- yarn1이란 버전을 의미하는데, `yarn -v`를 하면 버전을 확인할 수 있음
- yarn1이 일반적으로 npm 대신 사용하는 패키지 매니저

## 순서

1. `yarn init`
2. packages 폴더 생성
3. root package.json 내 workspaces 설정

```json
  "private": true,
  "workspaces": [
    "packages/*"
  ]
```

4. packages 폴더 하위에 workspace 생성

```shell
mkdir packages
mkdir ./packages/common
mkdir ./packages/server
```

- 생성한 각각의 폴더에서 `yarn init`
- 각 폴더의 package.json 내 name 값을 각각 "@study/common" "@study/server"로 변경

5. 테스트를 위한 각 폴더별 index.js 파일 생성

`common/index.js`

```javascript
module.exports = () => {
  console.log("yarn1 study");
};
```

`server/index.js`

```javascript
const log = require("@study/common");
console.log("hello from server");
log();
```

6. 의존성 주입

- server 폴더에서 common 프로젝트를 의존성을 주입하려면?

`yarn workspace @study/server add @study/common@1.0.0`

- 실행 확인
  `server/package.json` 내 run script 추가

```json
  "scripts": {
    "dev": "node index.js"
  },
```

`yarn workspace @study/server dev`
