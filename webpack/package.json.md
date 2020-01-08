

## 执行npm script

- 串行多个指令，按照顺序执行， **&&**符号把多个`npm script`按照先后顺序串行

```js
{
  "test": "npm run lint && npm run dev && npm run mock"
}
```

- 并行多个指令，**&**符号实现并行，需要`& wait`

```js
{
  "test": "npm run lint & npm run dev & npm run mock & wait"
}
```

- 使用`npm-run-all`实现串行和并行

安装

```shell
npm i -D npm-run-all
```

串行

```js
{
  "test": "npm-run-all lint dev mock"
}
```

并行

```js
{
  "test": "npm-run-all --parallel lint dev mock"
}
```

- 使用`concurrently`

安装

```shell
npm i -D concurrently
```

并行

```js
{
  "test": "concurrently \"npm run dev\" \"npm run mock\""
}
```

## script 增加注释

- `pckage.json`会忽略键为`//`的配置，可以将注释都写到键为`//`对应的值中

```js
{
  "//": "先执行代码检测，再运行项目，最后运行mock服务",
  "test": "npm run lint && npm run dev && npm run mock"
}
```

- 另外一种方式是直接在 script 声明中做手脚，因为命令的本质是 shell 命令（适用于 linux 平台），可以在命令前面加上注释

```js
{
  "test": "# 先执行代码检测，再运行项目，最后运行mock服务 \n  npm run lint && npm run dev && npm run mock"
}
```

注意注释后面的换行符 `\n` 和多余的空格，换行符是用于将注释和命令分隔开，这样命令就相当于微型的 shell 脚本，多余的空格是为了控制缩进，也可以用制表符 `\t` 替代。这种做法能让 npm run 列出来的命令更美观，但是 scripts 声明阅读起来不那么整齐美观

