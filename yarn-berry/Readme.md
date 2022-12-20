# yarn-berry으로 모노레포 구성하기

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
  console.log('hello from lib');
  return 'hello from lib';
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

## prettier, eslint 설정

1. root에 prettier, eslint 설치

```shell
yarn add prettier eslint eslint-config-prettier eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks eslint-import-resolver-typescript @typescript-eslint/eslint-plugin @typescript-eslint/parser -D

yarn dlx @yarnpkg/sdks
```

- `.vscode/extensions.json` 에 익스텐션 추가 확인

```json
{
  "recommendations": ["arcanis.vscode-zipfs", "dbaeumer.vscode-eslint", "esbenp.prettier-vscode"]
}
```

2. vscode 익스텐션 설치

- esbenp.prettier-vscode
- dbaeumer.vscode-eslint

3. `.vscode/settings.json` 설정 추가

```json
"editor.defaultFormatter": "esbenp.prettier-vscode",
"editor.formatOnSave": true,
"editor.rulers": [
  120
]
```

4. root에 `.prettierrc` 추가

```json
{
  "arrowParens": "avoid",
  "bracketSameLine": false,
  "bracketSpacing": true,
  "endOfLine": "lf",
  "jsxSingleQuote": false,
  "printWidth": 120,
  "proseWrap": "preserve",
  "quoteProps": "as-needed",
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

5. root에 `.eslintrc.js` 파일 추가

```javascript
module.exports = {
  root: true,

  env: {
    es6: true,
    node: true,
    browser: true,
  },

  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {jsx: true},
  },

  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'import', 'react', 'react-hooks'],
  settings: {'import/resolver': {typescript: {}}, react: {version: 'detect'}},
  rules: {
    'no-implicit-coercion': 'error',
    'no-warning-comments': [
      'warn',
      {
        terms: ['TODO', 'FIXME', 'XXX', 'BUG'],
        location: 'anywhere',
      },
    ],
    curly: ['error', 'all'],
    eqeqeq: ['error', 'always', {null: 'ignore'}],

    // Hoisting을 전략적으로 사용한 경우가 많아서
    '@typescript-eslint/no-use-before-define': 'off',
    // 모델 정의 부분에서 class와 interface를 합치기 위해 사용하는 용법도 잡고 있어서
    '@typescript-eslint/no-empty-interface': 'off',
    // 모델 정의 부분에서 파라미터 프로퍼티를 잘 쓰고 있어서
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-parameter-properties': 'off',
    '@typescript-eslint/no-var-requires': 'warn',
    '@typescript-eslint/no-non-null-asserted-optional-chain': 'warn',
    '@typescript-eslint/no-inferrable-types': 'warn',
    '@typescript-eslint/no-empty-function': 'off',
    '@typescript-eslint/naming-convention': [
      'error',
      {format: ['camelCase', 'UPPER_CASE', 'PascalCase'], selector: 'variable', leadingUnderscore: 'allow'},
      {format: ['camelCase', 'PascalCase'], selector: 'function'},
      {format: ['PascalCase'], selector: 'interface'},
      {format: ['PascalCase'], selector: 'typeAlias'},
    ],
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/array-type': ['error', {default: 'array-simple'}],
    '@typescript-eslint/no-unused-vars': ['error', {ignoreRestSiblings: true}],
    '@typescript-eslint/member-ordering': [
      'error',
      {
        default: [
          'public-static-field',
          'private-static-field',
          'public-instance-field',
          'private-instance-field',
          'public-constructor',
          'private-constructor',
          'public-instance-method',
          'private-instance-method',
        ],
      },
    ],

    'import/order': [
      'error',
      {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index', 'object'],
        alphabetize: {order: 'asc', caseInsensitive: true},
      },
    ],

    'react/prop-types': 'off',
    // React.memo, React.forwardRef에서 사용하는 경우도 막고 있어서
    'react/display-name': 'off',
    'react-hooks/exhaustive-deps': 'error',
    'react/react-in-jsx-scope': 'off',
    'react/no-unknown-property': ['error', {ignore: ['css']}],
  },
};
```

6. `.vscode/settings.json` eslint 설정 추가

```json
"eslint.packageManager": "yarn",
"eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact"],
```

7. apps/rarla/.eslintrc.json 파일 삭제

8. apps/rarla/pages/index.tsx eslint 동작 확인

- eslint가 정상적으로 동작이 안되면 eslint 서버를 재시작 해본다.
  - command + shift + p
  - ESLint: Restart EsLint Server 선택

## react library 패키지 만들기.

1. packages/ui 폴더 생성 및 `package.json` 생성

```shell
cd packages/ui
yarn init
```

- `package.json` 수정

```json
{
  "name": "@rarla/ui",
  "packageManager": "yarn@3.3.0"
}
```

2. react dependency 설치

- root로 이동

```shell
yarn

yarn workspace @rarla/ui add typescript react react-dom @types/node @types/react @types/react-dom -D
```

3.  ui 패키지 설정

- `packages/ui/tsconfig.json` 설정

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "baseUrl": "./src",
    "target": "esnext",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "jsx": "react-jsx",
    "noEmit": false,
    "incremental": true
  },
  "exclude": ["**/node_modules", "**/.*/", "dist", "build"]
}
```

4. `packages/ui/src/index.ts`, `packages/ui/src/Button.tsx` 파일 생성

- `packages/ui/src/Button.tsx`

```typescript
import {ButtonHTMLAttributes, MouseEventHandler, ReactNode} from 'react';

export type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement> & {
  children: ReactNode;
  onClick?: MouseEventHandler<HTMLButtonElement>;
};

const Button = (props: ButtonProps) => {
  const {children, onClick, ...other} = props;

  return (
    <button type="button" onClick={onClick} {...other}>
      {children}
    </button>
  );
};

export default Button;
```

- `packages/ui/src/index.ts`

```typescript
export {default as Button} from './Button';
```

5. `packages/ui/package.json` main 추가

```json
{
  "name": "@rarla/ui",
  "packageManager": "yarn@3.3.0",
  "main": "src/index.ts",
  "devDependencies": {
    "@types/node": "^18.11.16",
    "@types/react": "^18.0.26",
    "@types/react-dom": "^18.0.9",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^4.9.4"
  }
}
```

6. apps/rarla 에서 packages/ui 사용

- root에서 실행

```shell
yarn workspace @rarla/web add @rarla/ui
```

- `rarla/pages/index.tsx`

```typescript
import {Button} from '@rarla/ui';
...
<Button>Hello ui button</Button>
```

7. 구동 후 확인

```shell
yarn workspace @rarla/web dev
```

- 실행 시 브라우저에서 typescript 문법을 해석하지 못해서 아래와 같은 오류 발생

```
../../packages/ui/src/Button.tsx
Module parse failed: Unexpected token (3:7)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
| import {ButtonHTMLAttributes, MouseEventHandler, ReactNode} from 'react';
|
> export type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement> & {
|   children: ReactNode;
|   variant: 'contained' | 'outlined';
```

- @rarla/web에서 javascript로 변환(transpile) 해줘야 한다.

```shell
# next-transpile-modules 설치
yarn workspace @rarla/web add next-transpile-modules
```

- `apps/rarla/next.config.js` 파일 수정

```javascript
// @rarla/ui 패키지를 tranpile 시킨다.
const withTM = require('next-transpile-modules')(['@rarla/ui']);

/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
};

module.exports = withTM(nextConfig);
```

## typecheck 넣어보기

1. typescript package.json에 script추가

- 추가해야 할 파일
  apps/rarla/package.json
  packages/lib/package.json
  packages/ui/package.json

```json
"scripts": {
  "typecheck": "tsc --project ./tsconfig.json --noEmit"
},
```

2. `yarn workspace @rarla/web typecheck`

3. 모든 프로젝트를 typecheck 하는 scripts 만들기

- workspace-tools plugin 설치하기

```shell
yarn plugin import workspace-tools
```

- root package.json 수정하기

```json
"scripts": {
  "g:typecheck": "yarn workspaces foreach -pv run typecheck"
},
```

- 실행

```shell
yarn g:typecheck
```

---
