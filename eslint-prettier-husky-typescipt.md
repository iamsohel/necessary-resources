# Setup a Node.js project with Typescript, ESLint, Prettier, Husky

> Starting a personal node project could be easy; starting a team node project could be challenging. 


In my experience, common mistakes developer make when starting a projects are:

* No Linting
* Lack of compile-time type checking (not really mistake but less desirable in my preference)
* Inconsistent code styling
* Linter breaking the build

In this article, I am going to not only to share how to address these above problems, but also some of the best practices 

**The article assumes the reader knows the basics about nodes and typescripts**

## Typescript

We can start off by a simple node project consisting only the package.json file

```shell
yarn add typescript --dev
```

After adding the dev dependency, create a `ts.config.json` file under the root of the node project

```json
{
  "ts-node": {
    "compilerOptions": {
      "module": "commonjs"
    }
  },
  "compilerOptions": {
    "target": "ESNext",
    "module": "esnext",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "./src",
    "downlevelIteration": true,
    "types": ["express"]
  },
  "include": [
    "src"
  ],
  "exclude": [
    "**/*.spec.ts",
    "**/*.spec.tsx"
  ]
}

{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "es2017",
    "sourceMap": true,
    "outDir": "./build",
    "baseUrl": "./",
    "incremental": true,
    "rootDir": "./",
    "strict": true,
    "moduleResolution": "node",
    "paths": {
      "*": ["*"]
    },
    "esModuleInterop": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictBindCallApply": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "alwaysStrict": true,
    "allowUnreachableCode": false,
    "noUnusedLocals": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUncheckedIndexedAccess": true
  },
  "watchOptions": {
    "synchronousWatchDirectory": true
  },
  "include": ["src"]
}


```

The above are the minimal settings of typescript compiler `tsc`. 

It tells the compiler to 

* use `es2018` syntax when generating the distributable
* use the `commonjs` module format.
* generate *.js to `/dist` folder
* generate source map
* include all *.ts file inside `/src` folder
* exclude all files inside `/node_modules` folder

**Fancy configuration of `ts.config.json` is beyond the scope of this article**

Adding the following line to package.json

```json
{
  "scripts":{
    "build": "tsc"
  }
}
```

To build, run the following in shell

```shell
yarn build
```

## ESLint

We have a handful choices of linting tools for node development, but the de-factor standard these days for typescript is `ESLint`. Partnering with prettier it really improves consistency and productivity of a development team.

Again we are starting off by adding the dev dependencies

```shell
yarn add eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser --dev
```

ESLint does not support typescript out of the box. Fortunately we are able to add the typescript support by these two packages `@typescript-eslint/eslint-plugin` `@typescript-eslint/parser` thanks to the ESLint team's modular design.


.eslintrc.js for node.js

```
  module.exports = {
    env: {
      commonjs: true,
      node: true,
      browser: true,
      es6: true,
      jest: true,
    },
    parser: '@typescript-eslint/parser',
    plugins: ['prettier', 'jest'],
    extends: [
      "plugin:@typescript-eslint/recommended"
    ],
    parserOptions: { "sourceType": "module", "ecmaVersion": "latest" },
    ignorePatterns: [],
    rules: {
      'prettier/prettier': [
        'warn',
        {
          singleQuote: true,
          trailingComma: 'all',
          printWidth: 120,
          tabWidth: 2,
          semi: true
        },
      ],
      'no-template-curly-in-string': 0,
      'import/no-anonymous-default-export': 0,
      'no-console': 0,
      eqeqeq: ['error', 'always'],
      'jsx-a11y/anchor-is-valid': 0,
      'no-unreachable': 2,
      // 'no-var': 'error',
      'no-import-assign': 0,
      'no-prototype-builtins': 0,
      'object-curly-spacing': ['warn', 'always'],
      '@typescript-eslint/no-redeclare': 0,
      '@typescript-eslint/camelcase': 0,
      '@typescript-eslint/no-var-requires': 0,
      '@typescript-eslint/no-use-before-define': 0,
      '@typescript-eslint/explicit-member-accessibility': 0,
      '@typescript-eslint/no-empty-interface': 0,
      '@typescript-eslint/no-empty-function': 0,
      '@typescript-eslint/explicit-module-boundary-types': 0,
      '@typescript-eslint/ban-ts-comment': 0,
      '@typescript-eslint/no-explicit-any': 0,
      '@typescript-eslint/ban-types': 0,
      '@typescript-eslint/no-unused-vars': 'off',
      '@typescript-eslint/no-unused-vars-experimental': 'off',
      'no-unused-vars': 'off',
      'no-control-regex': 0,
      "import/prefer-default-export": "off",
    },
  };


```

.eslintrc.js for reactjs

```
module.exports = {
  settings: {
    react: {
      pragma: 'React',
      version: '17.0.2',
    },
  },
  env: {
    browser: true,
    es6: true,
    jest: true,
    node: true,
    webextensions: true,
  },
  parser: '@typescript-eslint/parser',
  plugins: ['react', 'react-hooks', 'prettier', 'jest'],
  extends: [
    'react-app',
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  ignorePatterns: [],
  rules: {
    'prettier/prettier': [
      'warn',
      {
        singleQuote: true,
        trailingComma: 'all',
        printWidth: 120,
        tabWidth: 2,
        semi: true
      },
    ],
    'react/jsx-uses-vars': 2,
    'react/no-children-prop': 0,
    'react/prop-types': 0,
    'react/display-name': 0,
    'react/no-unknown-property': 0,
    'react/no-deprecated': 0,
    'react/jsx-key': 0,
    'react-hooks/exhaustive-deps': 0,
    'react/no-unescaped-entities': 0,
    'no-template-curly-in-string': 0,
    'import/no-anonymous-default-export': 0,
    'no-console': 0,
    eqeqeq: ['error', 'always'],
    'jsx-a11y/anchor-is-valid': 0,
    'no-unreachable': 2,
    // 'no-var': 'error',
    'no-import-assign': 0,
    'no-prototype-builtins': 0,
    'object-curly-spacing': ['warn', 'always'],
    '@typescript-eslint/no-redeclare': 0,
    '@typescript-eslint/camelcase': 0,
    '@typescript-eslint/no-var-requires': 0,
    '@typescript-eslint/no-use-before-define': 0,
    '@typescript-eslint/explicit-member-accessibility': 0,
    '@typescript-eslint/no-empty-interface': 0,
    '@typescript-eslint/no-empty-function': 0,
    '@typescript-eslint/explicit-module-boundary-types': 0,
    '@typescript-eslint/ban-ts-comment': 0,
    '@typescript-eslint/no-explicit-any': 2,
    '@typescript-eslint/ban-types': 0,
    '@typescript-eslint/no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars-experimental': 'warn',
    'no-unused-vars': 'off',
    'no-control-regex': 0,
  },
};

```

It tells the ESLint linter to:

* use Typescript parser
* use `Recommended Typescript preset` for linting
* use `ES2018` and  `module` sourceType to match ts.config settings

add .eslintignore

  ```json
        node_modules
        dist
        logs
  ```
Add the following lines in `package.json`:

```json
{
  "scripts": {
    "lint": "eslint --max-warnings=0 --cache --cache-location ./.eslintcache ./src --ext .js,.ts -c ./.eslintrc.js",
    "lint:fix": "eslint --max-warnings=0 --cache --cache-location ./.eslintcache --fix ./src --ext .js,.ts -c ./.eslintrc.js",
    "format": "eslint src/**/*.ts --fix",
    "pretty": "prettier --write \"src/**/*.ts\""
  }
}
```

To lint, run the following in shell

```shell
yarn lint
```

To format the code to comply with linting rules, run the following in shell

```
yarn format
```

## Prettier

Prettier drastically improves team consistency by automatically formatting the code. To enable prettier, we first install it by running:

```shell
yarn add prettier --dev
```

Configure prettier by adding a `.prettierrc` to the root of the project with the following content

```json
{
    "semi": true,
    "trailingComma": "all",
    "singleQuote": true,
    "printWidth": 120,
    "tabWidth": 2
}
```

The setting let Prettier to

* Ensure semi colon at the end of each statement
* Ensure trailing comma at the end of each statement
* Convert all double quotes to single quotes where applicable
* Break into new lines for all lines greater than 120 characters wide
* Ensure tab width is 2 characters

In Visual Studio Code, `Ctrl (CMD) + P` then select `Format Code`, or enable `Format on Save` in settings for best result.

## Husky

No matter how careful I am, I always endup with situations where I changed and committed the code to Github without linting, and that can lead to failure CI builds. 

A good pratcice is to lint before commit. `Husky` is a very popular plugin to achieve so.

Install Husky by

```
yarn add husky --save-dev
```

Husky does not have its own configuration files. Instead we will add its config into package.json:

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "yarn lint",
      "pre-push": "yarn test"
    }
  },
}
```

Next time you commit, husky would exit the `git commit` when the code does not pass linting.

## The End

There are so many more things you could do to your project to ensure productivity, consistency and coding styles, but I think this is a good start. This article will be subject to improvements to the latest changes and practices.

If you find this article useful please let me know.
