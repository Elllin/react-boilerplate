# Webpack 적용기

## Webpack은 왜 사용할까?

webpack을 사용하기전 궁금했던 것은 왜 webpack을 사용할지에대한 궁금증이었다. 왜 webpack을 복잡하게 사용하며 왜 파일을 하나로 합쳐야 할까?

> 1. http 요청이 비효율적이기 때문이다.

다운로드 할 파일들이 많이지면 네트워크 커넥션이 많아진다. 네트워크에 부하가 많아지면서 느려질 수 밖에 없다.

웹페이지는 html, css, js 등 수 많은 구성요소로 이루어져 있다. 이 외에도 이미지, json 데이터 등등 수 많은 파일들을 받아와야 하는데 http/1.1에서는 커넥션 하나를 열어 하나씩 요청을 보내야한다. 하나의 요청이 끝나야 다음 요청을 보낼 수 있기 때문에 요청이 많을수록 비효율적이다. 때문에 빠른 요청을 하기위해 webpack이 필요한 것이다!

하나의 파일로 합치기에 파일이 너무 크다면 여러 개로 나눌 수도 있다. 라이브러리들은 자주 수정되지 않기 때문에 라이브러리만 모아둔 파일과 코드 수정이 자주되는 핵심 페이지를 분리하여 두개의 JS로 만들 수도 있다.

아래 그림과 같이 webpack은 여러 파일들을 하나로 합쳐주면서 크로스 브라우징 이슈도 대응해준다.

![](https://images.velog.io/images/lllen/post/062e8fe4-d62d-40ac-b1cf-e2d53f282f0a/image.png)

## 설정 파일 구분

webpack 4가 나오면서 개발시 사용하는 `Development` 모드와 파일을 압축하여 변수 이름 등을 읽기 어렵게 축약해 놓은 `Production` 모드가 생겼다.

한 개의 설정파일을 생성하고 공유하면서 커맨드라인에서 주어지는 mode 옵션을 받아서(두번째 인자) 각 빌드 별로 분기를 만들 어 설정해줄 수도 있지만 파일을 아예 개발용과 배포용으로 나눠놓는게 더 권장하는 방식이기 때문에 **Webpack Merge**를 사용해 개발용과 배포용 설정 파일에서 공통으로 쓰이는 부분을 webpack.common.js로 분리하여 사용할 것이다.
파일 체계는 아래와 같다.

```
webpack.common.js
webpack.dev.js
webpack.prod.js
```

아래와 같이 설치를 진행해준다.

```
npm i webpack-merge
```

## 설치

### 노드 프로젝트 생성

```
npm init
```

계속 엔터를 누르면 package.json파일이 생성되는 것을 알 수있다.

### 웹팩 설치

웹팩4 부터는 webpack core와 webpack-cli 패키지가 분리되었기 때문에 두 패키지를 각각 설치해야 한다. webpack-cli webpack을 터미널에서 실행하기 위한 툴이다.

```
npm install --save-dev webpack webpack-cli
```

## entry, output 생성

### entry

entry에 경우 두개의 js파일을 생성하고 싶으면 아래와같이 두개의 파일을 명시해준다

```
entry: {


   app: './src/index.js',
       issue: './src/issue.js'

}

```

하나의 entry에 여러 파일을 넣고싶을 경우 배열로 처리한다.
아래의 경우는 a.js랑 b.js가 한 파일로 엮여 app.js라는 결과물로 나온다. 이렇게 웹팩은 entry의 js 파일부터 시작해서 import, require 관계로 묶여진 다른 js까지 알아서 파악한 뒤 모두 entry에 기재된 키 개수만큼으로 묶어준다.

```
 entry: {
   app: ['a.js', 'b.js'],
 },

```

#### 추가로 알게된 점

처음에는 entry파일에 polyfill을 적용해주기 위해 배열로 파일경로를 넣어주었다.
(배열에 넣고싶은 파일경로를 넣을 수도 있지만 npm 모듈들을 넣을 수도 있다.)

참고로 babel을 사용하면 최신문법을 브라우저 호환성에 맞춰 예전문법으로 바꿔주지만 Map, Set같은 새로나온 기능들을 사용하려면 polyfill을 적용해야 브라우저에 적용이 가능하다.

[babel 공식문서](https://babeljs.io/docs/en/babel-polyfill)
위 공식문서를 보면 Babel 7.4.0부터 더이상 사용하지 않고 `core-js`와 `regenerator-runtime`을 직접 사용하는 방식을 제안하고 있다.
또한 전역을 오염시키지 않고 웹팩 번들에 포함하여 번들 내부에 가두는 방법이 있어 그방법을 사용하기로 했다. 이러한 방식은 코드에서 사용한 폴리필 메서드만 번들에 포함된다. 방법은 아래와 같다.

우선 아래와 같이 설치를 해준다.

```
npm install --save-dev @babel/plugin-transform-runtime
npm install --save @babel/runtime @babel/runtime-corejs3
```

module의 rules아래 plugins에 아래 코드를 추가해준다.

```
 module: {

   rules: [
     {
        test: /\.jsx?$/,
       loader: "babel-loader",
       plugins: [
                 [
                  "@babel/plugin-transform-runtime",
                {
                 corejs: 3,
                },
             ],
          ],
         ...

```

### output

```
output: {
 filename: "[name].bundle.js",
 path: path.resolve(__dirname, "dist"),
 publicPath: '/',
},
```

**filename**에 [name]은 entry에서 설정해 주었던 app인 파일이름이 [name]으로 들어가 app.bundle.js로 번들 파일을 생성해준다.

**publicPath**는 파일들이 위치할 서버 상의 경로이다.

#### 추가로 알게된 점

HtmlWebpackPlugin을 사용하면 html 파일을 자동으로 생성해 주기 때문에 output을 입력하지 않아도 번들 파일이 dist에 생성된다. 파일의 이름이나 경로를 따로 지정해 줄 필요가 없으면 입력하지 않아도 된다. 이번 프로젝트 경우는 지정이 필요 없어서 생략했다.

## reslove

resolve는 웹팩이 알아서 경로나 확장자를 처리할 수 있게 도와주는 옵션이다.

**modules**에 node_modules를 넣어야 디렉토리의 node_modules를 인식할 수 있다.

### 확장자

**extensions**에 넣은 확장자들은 웹팩에서 알아서 처리해주기 때문에 파일에 저 확장자들을 입력할 필요가 없다. (같은 이름의 파일일 경우 배열에 먼저 넣은 확장자가 적용됨)

```
entry: {...},
output: {...},
resolve: {

   modules: ['node_modules'],
   extensions: ['.js','.jsx'],

},

```

### 경로 줄이기

**resolve.alias**에 이름과 경로를 넣어주면 빌드할 때 Key의 이름을 해당 key에 매칭된 path로 바꿔서 빌드 해준다. 예를들어

```
import main from '../../../main';
import Utility from '../../../utilities/utility';
```

위와 같은 경로를 설정해준 아래처럼 간단히 작성 가능하다.

```
 import main from '@/main';
 import Utility from 'Utilities/utility';  //Utilities가 key값
```

적용 코드는 아래와 같다.

```
module.exports = {
  ...
 resolve: {
  ...
   alias: {
      Utilities: path.resolve(__dirname, 'src/utilities/'),
     '@': path.resolve(__dirname, 'src/')
  }
 }
};
```

## loader

loader는 babel로더를 사용할것이며 아래 링크에 따로 정리했기 때문에 여기서 언급하지는 않도록 하겠다.

[babel 정리내용](https://velog.io/@lllen/babel)

```
@babel/core, @babel/cli, @babel/preset-env,
@babel/preset-react, babel-loader
```

## optimization

성능 최적화에 관련된 것이라 webpack.prod.js 에만 추가해주면 된다. 파일을 나누는 것은 아래를 참고하자

```
{
  optimization: {
     minimize: true/false, //UglifyJsPlugin을 계승
     minimizer: [], //TerserWebpackPlugin 추가 예정
     splitChunks: {}, //CommonsChunkPlugin을 계승
     concatenateModules: true,
     //ModuleConcatenationPlugin을 계승
}
}
```

위 설정들은 Production 모드에서는 이미 활성화되어있기 때문에 따로 설정해 줄 필요는 없을 것 같고 `splitChunks`와`TerserWebpackPlugin` 만 추가해 주면 될 것 같다.

`TerserWebpackPlugin` : 자바스크립트 코드를 난독화하고 debugger 구문을 제거한다. Production 모드일때 역시 기본 활성화가 되어있지만 기본 외에도 콘솔 로그를 제거하는 옵션도 있는데 배포 버전에는 로그를 감추는 것이 좋을 수도 있기 때문에 추가해 주었다.

아래와 같이 설치 후

```
  npm i -D terser-webpack-plugin
```

optimization에 아래와 같이 추가한다.

```
  minimizer: new TerserPlugin({
        terserOptions: {
           compress: {
            drop_console: true, // 콘솔 로그를 제거한다
          }
        }
      })
```

`SplitChunksPlugin` : 코드를 분리할때 중복을 없앤다.

코드를 압축하는 것 외에 entry 파일을 여러개로 나누면 브라우저 다운로드 속도를 높일 수 있다. 큰 파일 하나를 다운로드 하는것 보다 작은 파일 여러개를 동시에 다운로드하는 것이 더 빠르기 때문이다.

optimization에 아래와 같이 추가한다.

```
  splitChunks: {

     cacheGroups: {
       commons: {
         test: /[\\/]node_modules[\\/]/,
         name: "vendors",
         chunks: "all",
       },
     },
   },

```

`cacheGroups` : 특정 파일들을 청크로 분리할 때 사용. 여기서는 common 이랑 청크를 분리한다.
`test` : 분리할 대상이 되는 파일
`name` : 청크로 분리할 때 이름으로 사용될 파일명이다. output filename 옵션에 [name] 에 대치될 내용이기도 하다.
`chunks` : 모듈의 종류에 따라 청크에 포함할지 말지를 결정하는 옵션이다 initial 과 async 그리고 all 이 있다. 여기서는 all 을 사용하는데 말 그대로 test 조건에 포함되는 모든 것을 분리하겠다는 뜻이다. initial 은 초기 로딩에 필요한 경우, async 은 import() 를 이용해 다이나믹하게 사용되는 경우에 분리한다.

분리된 파일들은 서버가 열리면 HtmlWebpackPlugin 이 알아서 index.html에 주입해준다. 물론 production 빌드를 하면 분리된 번들 파일 두개가 생성된다

참고 : [김정환 블로그 - 프론트엔드 개발환경의 이해: 웹팩(심화)](https://jeonghwan-kim.github.io/series/2020/01/02/frontend-dev-env-webpack-intermediate.html)

그 외..
써드파티 라이브러리를 사용한다면 externals을 사용할 수도 있다.
패키지로 제공될 때 이미 빌드 과정을 거쳤기 때문에 빌드 프로세스에서 제외하는 것이 좋다. 이번 프로젝트에서는 적용할 필요가 없을 것 같아서 생략했다.

## plugin 및 코드분리

여기서 Production 모드에서 사용할 plugin과 Development 모드에서 사용할 plugin이 나뉘기 때문에 파일을 분리해야 한다.

우선 공동으로 필요한 webpack.common.js부터 생성해보자.

`webpack.common.js`

`htmlWebpackPlugin` : 서버를 띄울 때마다 임시 index.html 파일을 만들어 사용한다.
`dotenv-webpack` : .env 파일로 간단하게 노드의 환경 변수를 설정

htmlWebpackPlugin을 사용해 output파일을 생성하기위해 공통으로 사용했다.
dotenv는 개발 모드와 배포 모드를 따로 만들고 경로도 따로 줄 수 있지만 우리 프로젝트에서는 하나의 파일이기 때문에 따로 경로 지정은 생략하고 공통 파일에 넣었다.

아래와 같이 설치 후 적용하자

```
npm i --save-dev html-webpack-plugin dotenv-webpack
```

**최종파일은 아래와 같다.**

```
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const Dotenv = require("dotenv-webpack");

module.exports = {
  entry: "./src/index.js",

  module: {
    rules: [
      {
         test: /\.jsx?$/,
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-react", "@babel/preset-env"],
          plugins: [
            [
              "@babel/plugin-transform-runtime",
              {
                corejs: 3,
              },
            ],
          ],
        },
        exclude: /node_module/,
      },
    ],
  },

  resolve: {
    alias: {
      Components: path.resolve(__dirname, "./src/components/"),
    },
    extensions: [".js", ".jsx"],
  },

  plugins: [
    new HtmlWebpackPlugin({
      template: "./public/index.html",
      // public/index.html 파일을 읽는다.
      filename: "index.html",
      // output으로 출력할 파일은 index.html 이다
      showErrors: true, // 에러 발생시 메세지가 브라우저 화면에 노출 된다.
    }),
    new Dotenv(),
  ],
};

```

**주의할 점**

처음에 오류가 났는데 이유를보니 .env파일이 없어서 난 오류였다.
.env 빈 파일이라도 생성 후 번들링 해야한다.

### Development 모드

Development에서는 webpack-dev-server, Hot Module Replacement(HMR), 소스맵을 사용할 것이다.

`webpack-dev-server` : express로 만들어진 간단한 web server이다. 변경 사항을 감지해 리컴파일을 한뒤 브라우저를 빠르게 실시간 리로드한다. 디스크에 저장되지 않는 메모리 컴파일을 사용하기 때문에 컴파일 속도가 빨라진다.

`htmlWebpackPlugin` : 서버를 띄울 때마다 임시 index.html 파일을 만들어 사용한다. //동영상강의본 뒤 내용추가

`Hot Module Replacement(HMR)` : 코드에 변경이 생겨 다시 빌드할 때 매번 브라우저를 리로드 할 필요 없이 변경된 모듈만 바로 교체하는 기능이다. 그래서 현재 테스트 중인 스테이트가 계속 유지된다는 장점도 있다.

`소스맵` : 배포시 파일들을 압축하여 코드를 알아보기 힘들기 때문에 에러가 난다면 디버깅이 힘들다. 소스맵은 배포용으로 빌드한 파일과 원본 파일을 연결시켜주기 때문에 배포용 파일의 특정 부분이 원본 소스의 어떤 부분인지 쉽게 확인가능하다.

`inline-source-map` : 개발용에서는 로그, 디버깅, 번들링 타임을 고려하여 옵션 선택

우선 webpack-dev-server 를 설치한다.

```
npm i --save-dev webpack-dev-server react-hot-loader
```

아래와같이 설정한다.

```
devServer = {
         hot: true, //HMR을 사용한다는 의미로 따로 HMR를 추가하지 않아도 적용된다.
         //host: '0.0.0.0' // 디폴트로는 "localhost" 로 잡혀있다.
     외부에서 개발 서버에 접속해서 테스트하기 위해서는 '0.0.0.0'으로 설정해야 한다.
       contentBase: path.join(__dirname, 'dist'),
       port: 3000,
       inline: true,
       historyApiFallback: true,
```

`inline` : 컴파일된 코드를 일반적인 template html에 삽입하는 inline 모드와 iframe에 넣어 업데이트하는 iframe 모드가 있는데 HMR이 inline 모드에서 지원이 되므로 inline 모드를 사용했다.

`historyApiFallback`: HTML5의 History API를 사용하는 경우에 설정해놓은 url 이외의 url 경로로 접근했을때 404 responses를 받게 되는데 이때도 index.html을 서빙할지 결정하는 옵션이다. React와 react-router-dom을 사용해 프로젝트를 만들때도 react-router-dom이 내부적으로 HTML5 History API를 사용하므로 미지정 경로로 이동했을때, 있는 경로지만 그 상태에서 refresh를 했을때와 같은 경우에도 애플리케이션이 적절히 서빙될 수 있어서 유용한 옵션이다.

**최종 코드는 아래와 같다**

`webpack.dev.js`

```
const merge = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "development",

  devtool: "inline-source-map",

  devServer: {
    historyApiFallback: true,
    inline: true,
    port: 3000,
    hot: true,
    publicPath: "/",
  },
});



```

### Production 모드

프로덕션 모드에서 고려해야 할 것은 코드 미니파이지만 위에서 보다시피 웹팩 4에서는 UglifyWebpackPlugin 이 내장되어 추가 설치나 설정 없이 미니파이가 가능하다.

- 만약 Uglify를 필요한 사항에서만 적용하려면 optimization의 minimize를 false로 주고 `terser-webpack-plugin`을 활용하면 된다.

때문에 Production 모드에서는 빌드마다 기존 dist 디렉터리를 초기화 해주는 플러그인과 위에서 언급한 optimization 정도만 적용했다.

`clean-webpack-plugin` : build시 폴더를 초기화
`cheap-module-source-map` : 배포용 소스맵은 용량이 가장 작은 옵션으로 선택

아래와 같이 설치를 해준다.

```
 npm i --save-dev clean-webpack-plugin
```

**최종 코드는 아래와 같다**

`webpack.prod.js`

```

const merge = require("webpack-merge");
const TerserPlugin = require("terser-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "production",

  devtool: "cheap-module-source-map",

  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
    minimize: true,
    minimizer: [
      new TerserPlugin({
        sourceMap: true,
        terserOptions: {
          compress: {
            drop_console: true,
          },
        },
      }),
    ],
  },

  plugins: [new CleanWebpackPlugin()],
});


```

**주의할 점**
CleanWebpackPlugin 사용법에 주의하자! 예전 블로그에는 잘못 된 방법으로 사용된 내용이 간혹 있어서 나도 오류를 경험했다.

const { CleanWebpackPlugin } = require("clean-webpack-plugin"); 이렇게 import 해야하며 사용시 아래와 같이 사용한다
plugins: [new CleanWebpackPlugin()],

## package.json

> ```
> "scripts": {
> "start": "webpack-dev-server --open --config webpack.dev.js", //dev-server 적용
>   "build": "webpack --config webpack.prod.js"
> },
> ```

```

### 참고

[webpack-dev-server](https://brightparagon.wordpress.com/2018/06/27/webpack-v4-development-configuration/)
[소스맵](https://perfectacle.github.io/2016/11/14/Webpack-devtool-option-Performance/#%EC%86%8C%EC%8A%A4%EB%A7%B5)
[babel폴리필](https://okchangwon.tistory.com/3)
[개발환경 최적화](https://jeonghwan-kim.github.io/series/2020/01/02/frontend-dev-env-webpack-intermediate.html)
[webpack4 설정](https://www.zerocho.com/category/Webpack/post/58aa916d745ca90018e5301d)
```
