# yarn-berry을 사용해 모노레포 구성하기

- yarn-berry는 yarn 2.0 이상 버전을 말함
- 모듈 설치 시 node_modules 디렉토리가 아닌 .pnp.cjs 파일을 생성함
- .pnp.cjs 파일에는 모듈의 버전과 저장된 위치, 참조하고 있는 다른 모듈들을 참조 테이블에 기록함

## 순서

1. yarn berry 버전 변경

```shell
yarn set version berry
yarn -v
```

2. yarn workspace 패키지 만들기

- packages 디렉토리 만들기 / 루트 초기화 작업

```shell
yarn init -w
```

3. root package.json 파일 수정

```json
{
  "name": "yarn-berry",
  "packageManager": "yarn@3.3.0",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

4. apps 폴더 생성 후 create next-app 프로젝트 추가

```shell
mkdir apps
cd apps
yarn create next-app rarla
```

5. rarla 폴더 내 package.json name 수정

```json
{
  "name": "@rarla/web"
}
```

6. root 경로에서 `yarn`

- 폴더 이름 등 설정 변경 시 root 에서 yarn을 해줘야 pnp 파일에 동기화됨

```shell
yarn # root로 가서 상태 갱신
yarn workspace @rarla/web run dev # 실행 확인
```

7. typescript error 해결

- yarn berry는 npm과 모듈을 불러오는 방식이 다르기 때문에 발생하는 문제

```shell
yarn add -D typescript
yarn dlx @yarnpkg/sdks vscode
```

- yarn PnP 사용을 위한 vscode extension arcanis.vscode-zipfs 설치

- `.vscode/extensions.json` 추가

```json
{
  "recommendations": ["arcanis.vscode-zipfs"]
}
```

7. packages/lib 공통 패키지를 만들어보기

```shell
cd packages/lib
yarn init
```

8. packages/lib/package.json 파일 수정

```json
{
  "name": "@rarla/lib",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "dependencies": {
    "typescript": "^4.9.3"
  }
}
```

8. packages/lib/tsconfig.json 파일 생성 후 설정값 넣기

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "strict": true,
    "useUnknownInCatchVariables": true,
    "allowJs": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "newLine": "lf",
    "module": "ESNext",
    "moduleResolution": "node",
    "target": "ESNext",
    "lib": ["ESNext", "dom"],
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "baseUrl": "./src",
    "noEmit": false,
    "incremental": true,
    "resolveJsonModule": true,
    "paths": {}
  },
  "exclude": ["**/node_modules", "**/.*/", "./dist", "./coverage"],
  "include": ["**/*.ts", "**/*.js", "**/.cjs", "**/*.mjs", "**/*.json"]
}
```

9. packages/lib/src/index.ts 생성

```typescript
export const sayHello = () => {
  console.log("hello from lib");
  return "hello from lib";
};
```

10. apps/rarla에서 packages/lib 의존해보기

- root 경로로 이동

```shell
yarn workspace @rarla/web add @rarla/lib
```

- apps/rarla/package.json에 `"@rarla/lib": "workspace:^"` 추가 확인

11. apps/rarla/pages/index.tsx 파일에서 @rarla/lib 사용해보기

```typescript
import { sayHello } from "@rarla/lib";
...
<h2>{sayHello()}</h2>
```

12. @rarla/web 구동하기

```shell
yarn workspace @rarla/web run dev
```

- 페이지에 hello from lib가 노출된다면 성공
