

> vite属于比较新的开发工具，在我们实际项目过程中需要一些plugin来做项目服务，往往不能像webpack一样有很多开源的工具库来去选择，虽然有些rollup的plugin可以用到vite上面，但是我们需要赋能自己的项目，就需要开发一些新的工具库，下面我们就会从零构建一个工具库开发的全过程。



<a name="HAWv1"></a>
### 一、构建项目基础内容（代码标准）


1. 构建空项目；
> npm init
> tsc --init

2. 我们可以通过workspace 这个功能集成过个项目或者包来进行开发，然后通用依赖放在根package.json中，集体介绍有可以参考知乎上面的一篇文章[https://zhuanlan.zhihu.com/p/381794854](https://zhuanlan.zhihu.com/p/381794854)
2. 将我们的多个子项目都放在packages目录下，同时我们需要主要root目录下package.json尽量放一些公共的库，例如typescript、eslint规范相关可以根目录放置一份;
2. 安装项目语法检测规范，其中包括（eslint、prettier）；
> npm i eslint -g
> eslint --init
> yarn add prettier -D -W

5. 项目中我们应该设定 peerDependencie，去限定库的使用的一些依赖库的版本限制；
> 注意：我们在安装eslint后，发现会有tsconfig.json报错，提示缺少node依赖，我们需要安装 @type/node 依赖在根目录下

6. 项目中接入jest单元测试工具，jest使用办法可以参考[https://www.jestjs.cn/docs/getting-started](https://www.jestjs.cn/docs/getting-started) 中文官方文档；
> yarn global add jest
> jest --init

通过上面命令在项目根目录下创建jest.config.ts配置文件，同时我们需要配置jest相关的配置项（匹配的文件，覆盖的目录等等），我的配置内容具体如下可做参考，最后我们需要在root目录下的package.json中增加相应的test执行命令
```typescript
// jest.config.ts配置文件
export default {
  // An array of file extensions your modules use
  moduleFileExtensions: [
    "js",
    "jsx",
    "ts",
    "tsx",
    "json",
    "node"
  ],
  preset: 'ts-jest',
  testPathIgnorePatterns: [
    "/node_modules/",
    "dist",
    "types"
  ],
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
  testRegex: '(/__tests__/.*|(\\.|/)(test|spec))\\.(ts|js)x?$',
};
```

7. 增加commitizen、conventional-changelog-cli提交代码规范工具，这个工具是让我们提交代码备注方面更加标准一点，具体用法如下，当然你也可以定制自己的提交格式，因人而异；
> npm i commitizen -g
> commitizen init cz-conventional-changelog --save-dev --save-exact

当前root目录下的package.json的内容如下，可以做个阶段的参考
```json
{
  "name": "vite-plugin-parse-html",
  "version": "1.0.0",
  "description": "its a html parse plugin",
  "main": "index.js",
  "private": "true",
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "test": "jest",
    "genlog": "conventional-changelog -p angular -i CHANGELOG.md -s"
  },
  "keywords": [
    "vite",
    "vite-plugin",
    "vite-plugin-parse-html"
  ],
  "author": "kanade",
  "license": "ISC",
  "devDependencies": {
    "@types/jest": "^27.0.3",
    "@types/node": "^16.11.10",
    "@typescript-eslint/eslint-plugin": "^5.4.0",
    "@typescript-eslint/parser": "^5.4.0",
    "commitizen": "^4.2.4",
    "conventional-changelog-cli": "^2.1.1",
    "cz-conventional-changelog": "^3.3.0",
    "eslint": "^8.3.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-plugin-import": "^2.25.3",
    "eslint-plugin-jest": "^25.3.0",
    "husky": "^7.0.4",
    "jest": "^27.3.1",
    "prettier": "^2.5.0",
    "ts-jest": "^27.0.7",
    "ts-node": "^10.4.0",
    "typescript": "^4.5.2",
    "vite": "^2.6.14"
  },
  "peerDependencies": {
    "vite": ">=2.0.0"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}

```
同时可以参考一下eslintrc的配置文件
```javascript
// .eslintrc.js文件
module.exports = {
    "root": true,
    "env": {
        "browser": true,
        "es2021": true,
        "es6": true,
        "node": true
    },
    "extends": [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
        // 'plugin:jest/recommended',
        'prettier',
    ],
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
        "ecmaVersion": 13,
        "sourceType": "module"
    },
    "plugins": [
        "@typescript-eslint",
        "jest"
    ],
    "rules": {
        "no-console": 1,
        "@typescript-eslint/no-non-null-assertion": "off",
        "@typescript-eslint/ban-ts-comment": "off",
        "@typescript-eslint/no-explicit-any": "off",
    }
};

```

8. 项目中增加代码提交前置的格式化工具，做一个开源库或者多人协作项目统一代码风格方便阅读与长久维护，具体办法如下：
   - 安装基本库
> yarn add lint-staged husky -W -D

   - 在package.json中增加prepare命令，prepare命令会在npm install 或 yarn之后执行，会在项目目录下面生成husky脚本目录
> "prepare": "husky install"

   - 生成pre-commit hooks和commit message内容
> npx husky add .husky/pre-commit "npx lint-staged"
> npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'

   - 添加commitlint工具，git commit时候进行代码格式化，我们需要在项目根目录下创建commitlint.config.js配置文件；
>      yarn add  @commitlint/cli @commitlint/config-conventional -W -D

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
}
```

   - 我们需要创建lint-staged对应的配置文件，可以是.lintstagedrc.json，也可以是js文件的方式进行配置，如果是js文件方式我们需要创建相应的script脚本命令，指定使用的配置文件，当我们执行git commit 时候回触发husky 设定好的lint校验与格式化，这样我们可以很好的保证和限定代码的规范化，以下是配置文件
```json
// .lintstagedrc.json文件
{
  "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
  "!(package)*.json'": ["prettier --write--parser json"],
  "*.md": ["prettier --write"]
}
```
通过上面的步骤，我们算是基本完成了一个vite plugin开发的基础环境，这部分主要包括了一些eslint标准、commit提交标准部分、还有我们的单元测试部分，接下来我们将会进入到的我们核心插件开发环节，跟着节奏一步一步实现一个插件的开发与提交，下面附上部分主要的项目结构目录及package.json文件；
```json
{
  "name": "vite-plugin-parse-html",
  "version": "1.0.0",
  "description": "its a html parse plugin",
  "main": "index.js",
  "private": "true",
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "test": "jest",
    "genlog": "conventional-changelog -p angular -i CHANGELOG.md -s",
    "prepare": "husky install"
  },
  "keywords": [
    "vite",
    "vite-plugin",
    "vite-plugin-parse-html"
  ],
  "author": "kanade",
  "license": "ISC",
  "devDependencies": {
    "@commitlint/cli": "^15.0.0",
    "@commitlint/config-conventional": "^15.0.0",
    "@types/jest": "^27.0.3",
    "@types/node": "^16.11.10",
    "@typescript-eslint/eslint-plugin": "^5.4.0",
    "@typescript-eslint/parser": "^5.4.0",
    "commitizen": "^4.2.4",
    "conventional-changelog-cli": "^2.1.1",
    "cz-conventional-changelog": "^3.3.0",
    "eslint": "^8.3.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-import": "^2.25.3",
    "eslint-plugin-jest": "^25.3.0",
    "husky": "^7.0.4",
    "jest": "^27.3.1",
    "lint-staged": "^12.1.2",
    "prettier": "^2.5.0",
    "ts-jest": "^27.0.7",
    "ts-node": "^10.4.0",
    "typescript": "^4.5.2",
    "vite": "^2.6.14"
  },
  "peerDependencies": {
    "node": ">=12.22.0",
    "vite": ">=2.0.0"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```


