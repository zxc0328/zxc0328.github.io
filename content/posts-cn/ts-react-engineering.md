---
title: "React + Typescript 组件的工程化治理实践"
date: 2019-11-15 15:01:03
tags:
categories:
---


最近参与了一个 React + Typescript 组件项目，这个项目后期会开源，对代码的质量和工程化上有比较高的要求，因此需要进行工程化治理。通过这次工程化治理，笔者算是梳理清楚了一个 React + Typescript 第三方组件所需要的一些工程化方面的基础设施，在这里总结并分享给大家。

这次的工程化治理主要分以下几个方面：

- 静态检查：TypeScript + ESLint
- 开发体验：打包工具和 Mono-repo 管理
- 代码质量：测试

<!-- more -->

## 静态检查

TS 和 ESLint 这些工具本质上是对代码做静态检查，尽早发现隐藏的 bug。在 TS 出现之后，TS 有 ESLint 没有的类型检查，并且也具备 ESLint 具有的语法错误检查的能力，所以目前我们用 ESLint 主要是利用社区中数量庞大的 Lint 规则来对代码风格做一个规范，利用工具的方式去推行一些最佳实践。TS 则主要负责对代码语法和语义上的错误进行静态检查。另外，TS 本身是一个全新的语言，使用 TS 可以享受到一些 JS 没有的语言特性。


### 从 AnyScript 到 TypeScript

用 TS，一个很重要的区别就是有没有在配置中打开 strict 选项。如果没有的话，那其实你用的就是 AnyScript，在类型上基本没有约束，和 JS 没有太大的区别。如果是从 JS 迁移到 TS 的项目，这个选项应该关闭，因为老的 JS 代码没有写类型。但如果是全新的纯 TS 项目，strict 是一定要打开的。现在 CRA 这样的脚手架创建的项目也是默认开启了 strict 模式的。

打开 strict 模式其实很简单，难的是如何在 strict 模式下**优雅的写 TS 代码**。下面说说一些 strict 模式下的常见问题以及一些类型的技巧：

#### noImplicitAny 

这个问题出现的最常见的场景就是函数的参数。如果习惯了写 JS，在写函数参数的时候很大可能会忘记写类型。虽然 TS 可以推断出函数的返回值类型，但不能推断出函数的参数类型。如果不写参数类型，那参数的类型默认就是 any，这个时候就会报 noImplictAny 的错误，因为 TS 的 strict 模式下不允许这种隐式 any 的存在。

解决这个问题，就要养成给函数参数加类型的习惯，并且不能直接加个显式的 any 就完事了😂。该定义新类型就定义，如果已经定义的就引用一下。不是非常规的场景，是不应该出现 any 的，这个后面还会再讲到。any 本身是一个绕过类型检查的 escape hatch，用了 any 就会导致这个地方的类型检查被绕过，这样一来使用 TS 的意义就不大了。

话说回来，如果写代码的同学的背景是写静态类型语言的，那是绝对不会忘了加类型的。这个问题在习惯写弱类型的 JS 的前端同学身上比较常见，更多是一个习惯和**类型思维的养成问题**。

让我们再来看一个场景：

```javascript
const props = {
   foo: 'bar'
}

props['foo'] = 'baz' // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type
```

这种场景下也会报 noImplicitAny 的错误。这是因为我们没有显式声明这个对象的 index signiture。解决方案就是：

```typescript
interface Props {
   foo: string;
   [key:string]: Props[keyof Props]
}

const props:Props = {
   foo: 'bar'
}

props['foo'] = 'baz' // ok
props['bar'] = 'baz' // error
```

这里本来可以直接写 `[key:string]: any;`  的，但如果在 key 确定的情况下，可以用 keyof 来获取一个接口的所有 key 组成的联合类型，然后用 index types 获取 value 的类型。这样相比 any 能对 value 的类型有一个限制。

#### strictNullChecks

在默认（非 strict 模式）下，undefined 和 null 可以被赋值给任意的值，所以去调用一个可能为 undefined 的属性上的方法也是被允许的。在 strict 模式下，null 和 undefined 被作为单独的类型处理，不能赋值给其他的类型。因此如果一个值可能是 null 或者 undefined，我们必须采取措施，为类型检查提供信息。

比如有如下的场景：

```jsx
class Component extends React.Component<{}, {}> {
    graph: Graph?;

    componentDidMount() {
      this.graph = new Graph()
      this.init(this.graph) 
    }
    
    init() {
      this.graph.on("click", () => {})  // Object is possibly 'undefined'
    }

    render() {
        return <div>foo</div>
    }
}

```

在 GUI 的场景中，很多的成员变量是会在组件初始化之后才有值的，比如这个场景下的 `this.graph` 。在变量参数或者属性的类型后面加一个 ? 是 [Optional](https://www.typescriptlang.org/docs/handbook/advanced-types.html#optional-parameters-and-properties) 的意思。表名这个属性或者参数有可能是 undefined。

要想绕过这个报错，我们需要用[ type guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#optional-parameters-and-properties)：

```jsx
init() {
    if (this.graph) this.graph.on("click", () => {})  // ok
}
```
#### 
但如果调用 `this.graph`  的地方比较多，写 if 就很麻烦，而且影响代码的阅读。这个时候在确保 `this.graph`  一定有值的情况下，我们可以用 [type assertion](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-type-assertions):

```jsx
init() {
    this.graph!.on("click", () => {})  // ok
}
```

在近期的 TS 3.7 版本中新推出的 [Optional Chaining](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining)，是更好的解决方案：

```jsx
init() {
   this.graph?.on("click", () => {})  // ok
}
```

> ! 和 ? 这些操作符的用法，写过 Swift 的同学应该会很熟悉。现在 TS 里面基本有了一整套的 Nullable Value 的操作方案。


总结一下，对于 strict 模式下的 strictNullChecks，我们可以用 type guards，type assertion，optional chaining 三种办法去告诉编译器，这里的操作是安全的。

关键是，Optional 的值在 GUI 编程中是很正常的，我们要学会去处理和面对这些情况，把 undefined 和 null 作为一个单独的类型来对待。

#### 第三方库的类型定义

在引用第三方库时，我们需要注意一点，就是这个库有没有提供类型定义文件。类型定义文件一般在项目 package.json 的 types 字段中有说明。对于 JS 写的库，类型定义也可能是一个单独的包，比如 [@types/react](#) 这样的。

有类型定义文件，那我们在调用 API 的时候就可以引入对应的类型定义，调用的时候也会有类型检查和代码提示，来提升我们使用第三方库的效率，提早发现可能的 bug。

如果没有，就要考虑是否自行维护一个定义文件，但这样的成本是很大的。所以，一个没有定义文件的第三方库，我们要仔细的考虑是否要在 TS 项目中使用这个库。

#### 高级类型

TS 文档中有一章叫 “[高级类型](https://www.typescriptlang.org/docs/handbook/advanced-types.html)”。里面提到的都是一些高级的类型特性。除了 Type Guard，交叉类型，联合类型和上文提到的可以为 null 的类型之外，最关键的是

- Mapped types（映射类型）
- Conditional Types（条件类型）
- Index types（索引类型）

在使用泛型时，这些技巧可以让我们**对类型进行“编程”**，想象一下对类型变量使用 ？三元表达式或者 `Array.prototype.map`  这样的方法。

举个例子，条件类型的用法是这样的：

```typescript
T extends U ? X : Y
```

如果 T 和 U 兼容（T 包含 U 有的所有属性，T 可以被赋值给 U），这个类型就是 X，否则就是 Y。

看一下条件类型的实际用途。比如有如下的函数，可能返回 string，也可能是 null：

```typescript
function process(text: string | null): string | null {
  return text && text.replace(/f/g, "p")
}
```

但这样的类型写法是有问题的，因为返回值有可能是 null，没有 toUpperCase 这个方法。

```typescript
//            ⌄ Type Error! :(
process("foo").toUpperCase()
```

这个时候我们可以用条件类型来解决：

```typescript
function process<T extends string | null>(
 text: T
): T extends string ? string : null {
 ...
}
 
process("foo").toUpperCase() // ok
process().toUpperCase() // error
```

在写 TS 代码时，我们在掌握了这些高级类型技巧时，就可以适时的去用这些技巧来让代码的类型检查变的更健壮，避免重复定义类型，写更优雅的代码。

> 因为本文不是 TS 的专题文章，上文中没有提到的 TS 使用技巧，比如映射类型等等，还可以参考：[巧用 TypeScript](https://zhuanlan.zhihu.com/p/39620591) 和 [巧用 Typescript (二)](https://zhuanlan.zhihu.com/p/64423022) 以及 [TS 学习总结：编译选项 && 类型相关技巧](http://zxc0328.github.io/diary/2019/10/2019-10-05.html)

### 
### ESLint 和 Prettier

ESLint 和 Prettier 是更流行，普及程度更高的工具。这里就不讲太多细节，主要说的是 ESLint 如何支持 TypeScript。

#### ESLint + TypeScript：取代 TSLint 的新方案

TSLint 在 2019 年宣布未来项目将会废弃。TS 官方推荐 ESLint 作为 Linter。我们可以通过 [@typescript-eslint/parser](https://www.npmjs.com/package/@typescript-eslint/parser) 让 ESLint 支持解析 TS 文件。配套的还有  [@typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin) 作为 ESLint 下针对 TS 订制的 Lint 规则。

ESLint + TS 这方面的资料很多，[Using ESLint and Prettier in a TypeScript Project](https://dev.to/robertcoopercode/using-eslint-and-prettier-in-a-typescript-project-53jb)  这篇文章讲了如何从 TSLint 迁移到 ESLint。

还有以下的文章，都讲解了相关的配置（ESLint + TS）：

- [Integrating Prettier + ESLint + Airbnb Style Guide in VSCode](https://blog.echobind.com/integrating-prettier-eslint-airbnb-style-guide-in-vscode-47f07b5d7d6a)
- [Setting up ESLint with Prettier, TypeScript, and Visual Studio Code](https://levelup.gitconnected.com/setting-up-eslint-with-prettier-typescript-and-visual-studio-code-d113bbec9857)
- [From ESLint to TSLint and Back Again](https://codeburst.io/from-eslint-to-tslint-and-back-again-bf259c2e7437)

> 关于 TSLint 到 ESLint 的切换的背景，可以看 [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint) 这个项目的 README，讲的非常详细


使用 ESLint 的好处就是：可以**背靠 ESLint 的生态**，像 Airbnb 这样的规则集就可以直接用于 TS 项目。上面列举的博客就有讲如何配置 Airbnb + typescript-eslint + prettier 三种规则集。让项目可以用 typescript-eslint 来规范 TS 代码（TS 特有的 Lint 规则），用 Airbnb 来规范 React 和 JS 代码（TS 是 JS 的超集），用 Prettier 相关规则来关闭前两个规则中和 Prettier 代码风格冲突的规则。三者集合就是目前比较完善，好用的 Lint 规则了。

> Airbnb 中有一些规则，比如要求 React 组件声明 PropTypes，是不适用于 TS 项目的，所以需要在 ESLint 配置文件里关掉。其他类似的配置有很多，我们不用死板的遵守 Lint 规则，而是关闭不合适的规则，只取其精华。


#### JS + TS 混合项目 ESLint 配置

在又有 JS 又有 TS 文件的情况下，ESLint 需要只在 TS 文件上，执行 TS 相关规则的校验，不然在校验 JS 时很多 TS 规则也会生效，这样就造成了困扰。

解决方案就是使用 ESLint 的 override：

```javascript
"overrides": [
    {
      "files": "**/*.ts",
      "extends": [
        "eslint-config-airbnb",
        "plugin:@typescript-eslint/recommended",
        "prettier/@typescript-eslint",
        "prettier",
        "prettier/react"
      ],
    }
  ],
```

只在处理 TS 文件时才加入 TS 的相关规则。

与这个问题有关的 [issue](https://github.com/typescript-eslint/typescript-eslint/issues/109)。

另外，有一些 JS 规则在 TS 文件上使用时也会出现问题，比如: [https://github.com/eslint/eslint/issues/8813](https://github.com/eslint/eslint/issues/8813)。解决方案也是使用 override。

#### Pre commit hook

Pre commit hook 是指设置一个 Git hook，在提交之前运行。前端项目一般利用这个机会运行静态代码检查和代码格式化，比如 ESLint，Prettier。也可以运行测试或者 TS 编译等等检查。

这里特别提到 Commit hook 是因为这个环节是**必不可少**的。如果没有 commit hook，那 ESLint 和 Prettier 等于是**形同虚设**了。

具体设置的流程可以参考：[Configuring Pre-commit Hooks for Prettier and Linting on a TypeScript Project](https://justinnoel.dev/2019/07/13/configuring-precommit-hooks-for-eslint-and-prettier/)。

> commit hook 也可以用 -n 跳过，所以还应该在 CI 时加上 ESLint，来保证不规范的代码提交被立刻发现。 


---


## 开发体验

打包工具也好，mono-repo 也好，这些基础设施其实提升的是开发人员的体验。开发的时候省心省力，方便快捷，一键配置，一键升级，这是现在前端开发体验升级的方向。在选择一个 React 组件的构建工具链时，开发体验是值得我们关注的一个重要要素。

### 打包工具

关于模块的格式，我们听过 AMD，CommonJS，UMD，ES Module 等等。由于模块标准一开始大规模应用是 Nodejs 的 CommonJS，所以几年前我们写的 JS 模块都是以 CommonJS 的格式。Webpack 这样的打包工具也只兼容 CommonJS 模块。后来 ES 2015 中提出了 ES Module，这个标准是未来浏览器支持的标准，Nodejs 也会支持。并且从功能上来说，ES Module 语法上更简洁，支持 multiple exports，并且可以让构建工具进行静态的依赖分析，让 Tree-shaking 成为可能。

目前的构建工具都支持原生的 ES Module 格式（之前需要用 babel 转为 CommonJS）。我们写的组件源代码就是 ES Module。在输出方面，现在的 JS 库一般都会提供 ES Module 版本。所以我们需要寻找合适的打包方式。

#### 构建 ES module：Rollup/Babel

我们只需要把 package.json 的 [module](https://github.com/rollup/rollup/wiki/pkg.module) 字段指向打包出的 es module 格式的文件，构建工具就会使用 module 字段而不是 main 字段进行构建了。

接下来我们要选择打包工具，Webpack 目前不支持输出 ES module，[可能在 Webpack 5 会支持](https://github.com/webpack/webpack/issues/2933)。所以先排除 Webpack。

Rollup 和 Babel 是可行的两种方案。

Rollup 是目前最流行 JS 库打包工具，React，Vue 之类的开源项目都在使用 Rollup。Rollup 支持输出 CommonJS，UMD，ES Module 在内的主流格式，并可以通过插件支持 CSS 等静态资源的处理。Rollup 和 Webpack 的主要区别就是 Rollup 是以构建 JS 为核心的，并且从一开始就是基于 ES Module 的，如果要兼容 CommonJS 代码，需要引入额外的插件。Webpack 更关注的是所有资源的构建，并且强调 Code Splitting 的能力，专注于 Web 应用的打包。Rollup 更轻量和专注，而且支持 ES Module 的输出，所有在 JS 库打包这个方面 Rollup 是首选。

Babel 其实本身只是一个转译工具。但 Babel 可以通过插件支持 TS 代码的转译，还有 JSX 的转译（老本行），所以如果是简单的 TS 库，可以直接用 Babel 进行转译，输出的就是原汁原味的 ES Module（因为 Babel 压根没有去解析模块，只是单纯的转译代码）。需要注意的是 Babel 的 TS 转译只是转译，不是编译，所以类型错误是不会报出的，需要额外跑 tsc 来对 TS 代码进行类型校验。其他的静态资源也是一样的，需要单独跑 task。

#### 专注与 JS 库打包的 Father 

上面两种工具都可以用，但这里不打算讲如何配置，因为现在的趋势就是构建工具链下沉，封装为一个统一的入口。只要跑一个命令就可以构建，并且只需要配置一些简单的必要的参数。底下的工具链升级也只要更新一个入口工具就行，不用花时间去维护整个构建体系。Umijs，Create React App，和 Vue-cli 都是这样的例子。

这里向大家安利一款专注于 JS 库打包的工具：[Father](https://github.com/umijs/father)。Father 可以简单理解为是 JS 库领域的 CRA 或者 Umi。Father 封装了 Rollup 和 Babel 两套工具链。

在最简单的情况下：我们只需要告诉 Father 需要什么格式的输出就可以构建成功，比如：

```shell
father build --esm --cjs --umd --file bar src/foo.js
```

因此笔者在项目中就使用了 Father 来对 React 组件进行打包。如果对 Rollup 和 Babel 构建流程有兴趣的同学，可以看一下 Father 的源码，还是很容易看懂的。

### mono-repo 管理：Lerna

[Lerna](https://github.com/lerna/lerna) 是用于管理拥有多个 npm package 的 mono repo 的工具。mono repo 就是指多个项目的源码放在同一个仓库下进行管理。

简单的说，Lerna 的功能就是一键在多个 package 中同时运行一些命令。而且运行的时候还会根据 package 之间的依赖拓扑关系，对命令的启动顺序进行编排。同时 Lerna 的 bootstrap 命令可以把 package 之间相互的依赖，自动 link 到 package 自己的 node_modules 里面。这可以说是最大的一个卖点。Lerna 之前如果要在本地开发多个相互依赖的 npm 包，那就要敲一堆的 npm link，而且还容易出问题。

mono-repo 这种方式本身也是为了提升多个 npm package 的情况下，管理源代码的效率，以及共享基础设施。因此 Lerna 其实是提升了开发者开发基于 mono repo 的前端项目的体验。

前端的组件库一类的项目，用 Lerna 是非常合适的。


---


## 代码质量

其实之前提到的静态检查也是用于保证代码质量的，这里的代码质量主要是指测试。

### React 组件测试技术选型

React 组件的测试框架有很多，我选的是 Jest。因为这是 FB 自家的工具，也是一个很流行的测试框架。除了测试框架，我们还需要一个 DOM Util，用于组件渲染和 DOM 的操作。比较流行的就是 [Enzyme](https://github.com/airbnb/enzyme) 和 [react-testing-library](https://github.com/testing-library/react-testing-library)。

在 React 16 下，Enzyme 有一些问题，比如 shallow 模式下不支持 useEffect。详见：[https://github.com/airbnb/enzyme/issues/2086](https://github.com/airbnb/enzyme/issues/2086)。react testing library 是在 React 官方的 test util 基础上包装的，要更轻量一些。他的 FAQ 中写了对于 Enzyme 的看法：

> What about enzyme is "bloated with complexity and features" and "encourage poor testing practices"?
> Most of the damaging features have to do with encouraging testing implementation details. Primarily, these are [shallow rendering](http://airbnb.io/enzyme/docs/api/shallow.html), APIs which allow selecting rendered elements by component constructors, and APIs which allow you to get and interact with component instances (and their state/properties) (most of enzyme's wrapper APIs allow this).
> The guiding principle for this library is:
> 
> The more your tests resemble the way your software is used, the more confidence they can give you. - [17 Feb 2018](https://twitter.com/kentcdodds/status/977018512689455106)


作者觉得测试应该模仿用户使用你的产品时的操作，而不应该鼓励对实现细节进行测试。

综合各种因素，笔者选用了 react testing library。总的来说其实这种 Util 库，选用哪一个的差别不大。写起来更方便的就是好的。

如果对 react testing library 不熟悉，可以看官网和[这篇]()教程。

> [https://testingjavascript.com/](https://testingjavascript.com/) 这个测试教程网站，可以了解到测试相关技术的大图，如果对测试的分类和作用不太清楚可以看一下这个网站。


### 常见的测试技巧

#### react testing library 的测试套路

使用 react testing library 测试是很简单的，我们只需要调用 `render` ，把组件渲染出来就行了：

```jsx
const { asFragment, queryByText, rerender } = render(
  <Graphin data={data} layout={layout}>
    <div>foo</div>
  </Graphin>,
);
expect(queryByText(/foo/)).toBeTruthy();
```

比较有意思的就是 `render`  之后会返回一个 render result，里面是一些 DOM query util 和一些其他的 util。比如 `queryByText`  就是根据元素里的文本作为选择器来获取 DOM 元素。其他的 DOM query API 可以从[这里](https://testing-library.com/docs/dom-testing-library/api-queries)看到。其中用的比较多的一个是 [queryByTestId](https://testing-library.com/docs/dom-testing-library/api-queries#bytestid)，在 React 元素上加 `data-test-id`  之后就可以直接通过 `queryByTextId`  获取到这个元素。

可以看出，react testing library 鼓励的是根据元素的 text 这样的属性来获取元素，进行断言。这就是这个库的哲学，希望开发者从用户怎么使用产品的角度去测试。而不是通过 DOM 结构之类实现细节的来判断。

除了对渲染的 UI 进行测试，我们还需要触发事件，这个过程中需要用 `act`  和 `fireEvent`  这样的 API：

```jsx
act(() => {
  fireEvent.click(getByText(/Click Me/), {});
});
```

需要用 [act](https://reactjs.org/docs/test-utils.html#act) 包装的原因是，浏览器中 React 的渲染是有一定的周期的，会有 batch update。因此把会修改 state 的调用写在 act 中可以保证这个调用会完整的走完渲染周期。

如果想对组件的 props 进行更新，我们需要使用 `render`  结果里返回的 `rerender` ：

```jsx
data = { id: "1" } // update props.data
rerender(<Graphin data={data} layout={layout}>/Graphin>)
```

之后就可以继续使用第一次调用返回的那几个函数进行断言。

最后一个函数是 `asFragment` ，调用 `asFragment`  可以返回组件的 DOM 结构。这让我们可以使用 Jest 的 Snapshot 对组件进行测试：

```jsx
expect(asFragment()).toMatchSnapShot()
```

#### Mock 浏览器事件

测试中，经常会遇到需要 mock 函数或者其他对象的情况。mock 函数可以用 Jest 的 [Mock Functions](https://jestjs.io/docs/en/mock-functions.html)。比较麻烦的是一些浏览器事件的 mock。因为 Jest 的 DOM 实现使用的是 JSDom，并不是真实的浏览器环境。这里举一个例子，如果需要模拟浏览器的 resize 事件，可以这么做：

```typescript
act(() => {
  // Change the viewport to 500px.
  (window as any).innerWidth = 500;
  (window as any).innerHeight = 500;
});
fireEvent(window, new Event('resize')); 
```

#### Canvas 测试

如果测试的目标中有 Canvas，情况分两种：

- Canvas 上的内容是组件用到的图表库一类的渲染结果，和组件本身的正确性无关
- Canvas 上的内容就是测试的目标，比如给图表库写测试

如果是前者，我们可以 Mock 掉 Canvas，使用 [jest-canvas-mock](https://www.npmjs.com/package/jest-canvas-mock) 可以很方便的一键 Mock。

如果是后者，我们可以用 [jest-electron](https://github.com/hustcc/jest-electron) 去运行一个真实的浏览器，来测试 Canvas 的绘制结果。

使用 jest-canvas-mock 的时候，我们还可以通过 Mock 的 Canvas 对象上附加的 API 来获取 Canvas 上的绘制调用的信息：

```typescript
let canvas = getByTestId('custom-element').firstChild as HTMLCanvasElement;
let ctx = canvas.getContext('2d') as any;
ctx.__getPath() // 获取路径信息
ctx.__getEvents() // 获取事件记录
ctx.__getDrawCalls() // 获取绘制调用信息
```

这样我们就可以通过这些信息来看图表的绘制接口是否有被调用，从而看出调用了图表渲染 API 的 React 组件本身的逻辑是否正确。

### 覆盖率

Jest 配置了 `collectCoverage: true`  之后就会在本地生成测试覆盖率报表。用 http-server 起一个本地服务器就可以看到，类似如下的表格：

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/1573285869183-01c84b6e-ffc5-4ba8-8f9d-f37b1ef7b440.png)

覆盖率分为语句，行，分支，函数四个部分。我们一般说的覆盖率一般是指行覆盖率，就是代码本身有百分之几是被测试跑到的。但分支覆盖率也很重要，这意味着我们有没有把所有的 case 都测试到。覆盖率是不是要 100% 要看情况，如果是 lodash 这样的工具库，那就要有这样的指标。如果是比较复杂的 React 组件，那主要先保证核心链路是被覆盖的。

如果测试本身写的不好，覆盖率很高其实也没有用。比如只是把代码跑一遍但没有对结果做任何验证的话，就算代码逻辑出现了问题，测试也是 pass，覆盖率也很高，但这样的测试是没有用的。

总结一下，就是不能一味的追求数字的好看。覆盖率报表是帮助我们看测试是否漏掉了应该测试的函数，分支等等，起一个辅助的作用。评价测试的标准还是看测试能不能帮助我们在之后每一次提交代码时发现是否有 regression 的情况。

---


## 结语


本文是对一次 React + TypeScript 组件的工程化治理过程所做的总结。如果你的项目也是 React + TypeScript 组件，并且会发布为 NPM package 给其他人使用，那本文应该可以为工程化方面的建设提供一些参考。

因为篇幅原因，里面一些具体的流程需要读者自行看链接中的教程和博客，那些文章更专注，更有深度。本文主要介绍的还是 React + TypeScript 组件工程化的主要几个方向（静态检查，开发体验和代码质量）和其中一些需要解决的问题。

