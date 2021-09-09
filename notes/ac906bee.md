# Power Assertを使ったユニットテストをTypeScriptで書きそれをKarmaからMochaで実行する

Power Assertを使ったユニットテストをTypeScriptで書き、それをnode.jsからMochaで実行する場合はespower-typescriptを使えば簡単にできる。

これがKarmaから実行するとなると設定が複雑になるのだけど、この度やっと設定できたのでこれに関してのメモを残す。

ビルドツールにはwebpackを使用したがrollupなどといった他のツールでも代用できると思う。

## 必要なモジュール

以下のモジュールを `yarn add -D` または `npm install -D` などでインストールする。

- @tsconfig/node14
- @types/karma
- @types/karma-mocha
- @types/karma-webpack
- @types/mocha
- @types/node
- @types/webpack
- buffer
- cross-env
- karma
- karma-mocha
- karma-webpack
- mocha
- power-assert
- ts-loader
- ts-node
- ts-node
- typescript
- util
- webpack
- webpack-espower-loader

## karma.conf.ts

```ts
import * as webpack from 'webpack';

import type { Config, ConfigOptions } from 'karma';

function setConfig(config: Config): void {
  // NOTE: @types/karma-webpackがwebpackとwebpackMiddlewareを必須としている対策
  const co: Partial<ConfigOptions> = {};

  co.basePath = '.';
  co.client = {
    mocha: {
      reporter: 'html',
      ui: 'bdd'
    }
  };
  co.files = [
    {
      pattern: 'src/**/*.ts',
      type: 'js'
    }
  ];
  co.frameworks = ['mocha'];
  co.mime = {
    'text/x-typescript': ['ts']
  };
  co.preprocessors = {
    'src/**/*.ts': ['webpack']
  };
  co.reporters = ['dots'];

  co.webpack = {};
  co.webpack.mode = 'development';
  co.webpack.module = {
    rules: []
  };
  co.webpack.module.rules.push({
    test: /\.ts$/i,
    use: [
      { loader: 'webpack-espower-loader' },
      {
        loader: 'ts-loader',
        options: {
          compilerOptions: {
            // NOTE: 実行環境(e.g. Chrome, Firefox)向けに変換する
            target: 'ES2015'
          }
        }
      }
    ]
  });

  co.webpack.resolve = {
    alias: {
      // NOTE: クライアント向けにビルドされたpower-assertを使う
      'power-assert-formatter': require.resolve(
        'power-assert-formatter/build/power-assert-formatter'
      ),
      assert: require.resolve('power-assert/build/power-assert'),
      // NOTE: power-assert系の一連のモジュールが依存している
      buffer: require.resolve('buffer/'),
      util: require.resolve('util/')
    },
    extensions: ['.ts', '...']
  };
  co.webpack.plugins = [
    // NOTE: utilがprocess.env.NODE_DEBUGを内部で持っているため変換する
    new webpack.DefinePlugin({
      'process.env.NODE_DEBUG': false
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
    }) as any
  ];

  config.set(co as ConfigOptions);
}

export default setConfig;
```

## karma.tsconfig.json

```json
{
  "extends": "@tsconfig/node14/tsconfig.json",
  "ts-node": {
    "files": true,
    "transpileOnly": true
  }
}
```

この設定ファイルはKarmaの実行に対しての設定。要は `karma.conf.ts` を解釈するためのもの。

## package.json

```json
{
  "scripts": {
    "karma": "cross-env TS_NODE_PROJECT=karma.tsconfig.json karma"
  }
}
```

npm-scriptには上記のように記述しておき、 `yarn karma start` のようにすると実行できる。
