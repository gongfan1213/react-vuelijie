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