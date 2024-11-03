monorepo:协同管理不同模块的不同独立的库的生命周期，相对应的会有更高的操作复杂度的
mono-repo技术选型：
Npm workspace
Yarn workspace
Pnpm workspace
专业工具：nx,bit,turborepo,rush,nk,lerna
pnpm相比其他的打包工具更加有优势的：
依赖安装的更加快一些，更规范，处理幽灵依赖问题，处理不同的依赖是以link形式链接的，没有显示声明但是被安装的依赖
定义开发规范
代码规范的检查流程，eslint代码风格和代码规范的检查
Pnpm i eslint -D -w在根目录下安装依赖
。gitgnore: .node_modules，.history
Pnpm i -w -D @types-eslint /eslint-pluginlastest 
eslint:parser:用什么样的解析器把语法转换成为抽象语法树
parserOptions:ecmaVersion:lastest,sourceType:module
rules:具体的lint的规则,plugins:是规则的合集
extends:继承其他规则的,eslint:recommended;plugin:@typescropt-eslint /recommended 
代码风格prettier
semi用不用加分号,eslint可以做风格检查和pretter冲突
commit规范检查
Git commit husky安装，拦截git命令，
.husky/pre-commit "pnpm lint"
Pnpm lint会对代码进行全量的检查，当项目复杂后执行速度会比较慢，可以考虑使用lint-staged实现对暂存区的代码进行检查，通过commitlint对git提交信息进行检查，首先需要安装必要的库
Pnpm i commitlint @commitlint  
只对暂存区的代码进行检查，对commit信息是否规范进行检查的，新建配置文件commitlints.js
modules.expots = {extends:["commitlint/config-conventional"};指定规范集,.commitlints.js
集成当husky当中的，
conventional规范集的意义：
提交类型:提交信息，type：subject 
常见的type类型如下：feat:添加新功能，fix:修改chore:不影响已有功能修改，docs专指文档的修改,.perf性能方面的优化，refactor:代码重构，test添加一些测试代码等等
tsconfig.jsontypescript基础的入口
scripts放置工具的配置脚本的
# 第二章 jsx转换
react项目的结构：
react.js和宿主环境无关的公用的方法
react-reconciler协调器的实现，宿主环境无关的
各种宿主环境的包
shared公共的辅助方法，宿主环境无关
jsx转换是什么？
main代表入口文件，在react17之前，jsx的转换结果是react.createElement,17之后转换结果是jsx这个方法的调用的
包括两部分：1.编译的时候2.运行的时候：jsx方法或者react.createElement的方法的实现(包括dev,prod环境),
编译的时候的babel编译的时候实现，我们来实现运行的时候，工作量包括：1.jsx方法2.实现打包原理3.实现调试打包结果环境
运行时:jsx方法执行或者React.createElement的方法
实现jsx方法：
包括：1.jsxDev方法（dev环境）2.jsx方法（prod环境）React.creaeElement方法
```js
  //src/jsx.ts 
  import {jsx} from './src/jsx'
  
  import {Type,Key,Ref,Props,ElementType,ReactElement} from 'shared/reactTypes'
export default {
    version:'18.2.0',
    createElement: ,

}

const ReactElement = function (type:Type,key:Key,ref:Ref,props:Props):ReactElement {
    const element = {
      $$typeof:REACT_ELEMENT_TYPE,
      key,
      ref,
      type,
      props，
      _marks:"KaSong"
    }
    return element
  }
  ReactElement和宿主环境无关的数据结构，所以它的类型定义就应该放到shell当中
  shell 
  pnpm init 
  shell不需要入口文件的，因为里面的所有的方法都会被引用
  //react symbols为了防止别人滥用，所以使用symbols 
  const supportSymbol = typeof Symbol === 'function' && Symbol.for;
  export const REACT_ELEMENT_TYPE = supportSymbol ? Symbol.for('react.element') : 0xeac7;
  export const jsx = (type:ElementType,config:any,...maybeChildren: any[])

//shared/reactTypes 
export type Type = any;
export type Key = any ;
export type Ref = any ;
export type Props = any;
export type ElementType =any;

export interface ReactElement {

  $$typeof: symbol | number;
  type: Type;
  type; 
  key: Key;
  ref: Ref;
  props: Props;
  __marks: string;
}
export const jsx = (
    type:ElementType,
    config:any,
   ...maybeChildren: any[]
)=>{
  let key:Key = null;
  let ref:Ref = null;
  const props:Props = {};
  for (const prop in config) {
    const val = config[prop];
    if (prop === 'key') {
      if (val!== undefined) {
        key = '' + val;
      }
      continue;
    }
    if(prop === 'ref') {
        if(val!==undefined) {
            ref=val;
        }
        continue;
    }
    if({}.hasOwnProperty.call(config,prop)) {
        props[prop] = val;
        //是他自己的properyu还是原型上的props属性的
    }
  }
  const maybeChildren =maybeChildren.length;
  if(maybeChildren.length) {
    [child,单独的child,[child,child,child,child]]
    if(maybeChildren.length === 1) {
        props.children = maybeChildren[0];
    }else {
        props.children = maybeChildren;
    }
  }
    return ReactElement(type,key,ref,props);

};
export const jsxDEV = jsx;
//生产环境和开发环境的jsx都是同样的hi实现的,开发环境可以做额外的检查,



```

# 实现打包流程
对应上述三种方法，打包对应的文件：
react/jsx-dev-runtime.jsdev环境
react/jsx-runtime.jsprod环境
React
```js

//react.config.js
//路径的名字 
import path from 'path';
const pkgPath = path.resolve(_dirname,'../../packages');
const disPath = path.resolve(_dirname,'../../dist/node_modules')

export function resolvePkgPath (pkgName,isDist) {
    if(isDist) {
        return `${disPath}/${pkgName}`
    } else {
        return `${pkgPath}/${pkgName}`
    }

}
export function getPackageJSON(pkgName) {
    //包的路径
    const path = `${resolvePkgPath(pkgName)}/package.json`;
    const str = fs.readFileSync(path,{encoding:'utf-8'});
    return JSON.parse(str);

}
import {getPackageJSON,} from './utils'
import ts from 'rollup-plugin-typescript2';
import cjs from '@rollup/plugin-commonjs';
import typescript from 'rollup-plugin-typescript2';

import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
const module ='src/index.ts'
const pkgPath = getPackageJSON('react');
//react包的路径
const pkgDistPath = getPackageJSON('react',true);
//react的产物路径

export default {
    //react包
    //jsx-runtime包
    {
        input:`${pkgPath}/src/jsxt.ts`,
        output: [
            //jsx-rumtime 
            {
                file: `${pkgDistPath}/jsx-runtime.js`,
                format: 'umd',
                name: 'jsx-runtime.js',
                plugins: [...getBaseRollupPlugins(),generatePackageJson({
                    inputFolder: pkgPath,
                    outputFolder: pkgDistPath,
                    baseContents:({name,description,version}) => ({
                        name,
                        description,
                        version,
                        main: 'index.js'})//输出产物的入口

                })]

            },
            {
                file: `${pkgDistPath}/jsx-runtime.js`,
                format: 'umd',
                name: 'jsx-dev-runtime.js'

            },
        ]
    }
    {
        input: `${pkgPath}/${module}`,
        output: {
            file: `${pkgDistPath}/${module}`,
            format: 'umd',
            name: 'index'

        },
        plugins: getBaseRollupPlugins()
        external: (id) => {
            return /^react$/.test(id);
        },
        

    }
}
export function getBaseRollupPlugins({typescript= {}}) {
    return [cjs(),resolve(),typescript({tsconfig:'./tsconfig.json'})]
}

//pnpm install i -D -w rollup-lugin-peer-deps-external
//pnpm install i -D -w @rollup-plugin-commonjs
"scripts" : {
    "build:dev":"rollup --config scripts/rollup/react.config.js"

}
//scripts/react.config.js当中配置包的开发路径和生产环境路径,input,putput,file，plugins
//多了dist的目录pnpm i-D -w rimraf
//dist/node_modules/index.js,jsx-dev-runtime.js,jsx0runtime.js
//pnpm link --global 生成的包指向全局node_modules下的react包
//全局node_modules下的react下有我们生成的包
pnpm link react --global demo项目
依赖的项目变成全局的依赖
big-react->打包react包->pnpm link --global 全局下的依赖
dist/node_modules/react
 全局nodemodule下的react指向打包后的react 
 plugins: [
    ...getBaseRollupPlugins(), 
    generatePackageJson({
        inputFolder: pkgPath,
        outputFolder: pkgDistPath,
        baseContents:({name,description,version}) => ({
            name,
            description,
            version,
            main: 'index.js'})//输出产物的入口
    })]

 ]
npx creat-react-app react-demo 
pnpm link react --global 
npm start 启动后的效果r
react-dom包改一下项目的初始化的代码
不必要的代码给删除

```
模拟实际项目引用react项目的情况，热更新的调试方式
三种调试方式
scripts目录下都是我们打包的脚本
react.config.js
```js
export default [
    {
        input:,
        output:,
        plugins:,
    }
]
function getPackageJSON(pkgName,isDist){
    if(isDist){
        return  `${distPath}/${pkgName}}`
    }else{
        return `${pkgName}/${pkgName}.js`
    }
    const path = `${resolvePkgPath(pkgName,isDist)}.package.json`;
    const str = fs.readFileSync(path,{encoding:'utf-8'});
    return JSON.parse(str)

}
function resolvePkgPath(pkgName,isDist) {
    //源码路径和打包后的文件的路径

}
//utils工具放置我们打包后的方法
const pkgPath = path.resolve(__dirname,'../../packages')
const pkgDistPath = path.resolve(pkgPath,'dist');
const distPath = path.resolve(__dirname,'../../dist/m')
const pkgDir = fs.readdirSync(pkgPath)
//react.config.js
import {getPackageJSON,} from './utils' 
const {name,version} =getPackageJSON('react');
const pkgPath = resolvePkgPath('react');
const pkgDistPath = resolvePkgPath(pkgPath,'dist');

scripts: {
    lint: eslint --ext .js,.ts,.jsx .tsx --fix --quiet 
    build:dev :rollup --bundleConfigAsCjs --config scripts/rollup/react.js'
    //尝试将js文件以js文件加载,

}

export default [
    {
        input:`${pkgPath}/${module}`,
        output: {
            file: `${pkgDistPath}/index.js`,
            name:'index.js',
            format:'umd'
        },
        plugins:[]
    }
]
pnpm install -D -webkitURL
export function getBaseRollupPlugins(params) {
    return [
        resolve(),
        commonjs(),
        babel({
            exclude:'/node_modules/'
        })
    ]
}
export defualt getBaseRollupPlugins({

})
scripts: {
    build:dev :rollup --config scripts/rollup/react.dom.js'
    //尝试将js文件以js文件加载,

}
export default [ 
    //react包
    {
        input:`${pkgPath}/${module}`,
        output: {
            file: `${pkgDistPath}/index.js`,
            name:'index.js',
            format:'umd'
        },
        plugins:[...getBaseRollupPlugins(),generatePacgageJson({
        inputFolder:pkgPath,
        outputFolder:pkgDistPath,
        baseContents:({}name,description,version))=>({
        name,descript,
    }
    //react-jsxj-runtime 
    {
        input:`${pkgPath}/jsx-runtime`,
        output :[
            {file:`${pkgDistPath}/jsx-runtime.js`,
            name:'jsx-runtime.js',
        formate:'umd'},
            {file:`${pkgDistPath}/jsx-dev-runtime.js}`}
            //jsx-runtime 
            //jsx-dev-runtime
//file,name,formate,file,name,formate配置好对应的文件的内容
        ]

    }
]
//dist/mode_modules/index.js,jsx0dev0runtime.js,jsx0runtime.js 



```

big-rewact打包-react包
有一个这个包以后pnpm link--global,
生成的react包指向全局模式下的node_modules下的包，
Pnpm link react --global把项目依赖的react变成全局项目下的依赖
，全局项目下的react是打包好的包
Npx start 
React dom包，改一下项目的初始化的代码 
改变react的指向
Pnpm lin react --global 
Npm start 
第一个打印的是react,

