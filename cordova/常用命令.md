## 常用命令

- 全局安装 cordova

```
npm i -g cordova
```

- 查看版本

```
cordova --version
```

- 创建 cordova 项目

```
cordova create <path>
// cordova create MyApp 创建一个名称为 MyApp 的 cordova 项目

// 查看可以使用的相关参数
cordova create --help
```

- 查看平台

```
cordova platform ls // 查看所有的平台，可以安装和已安装的, ls 可以省略
```

- 添加平台

```
cordova platform add <platform name> // 增加某个platform
cordova platform add browser // 增加 browser
cordova platform add ios --nosave // --nosave 表示阻止向 package.json 文件添加和删除指定的平台，防止保存平台
cordova platform add ios@5.0.1 // @5.0.1表示添加制定版本的平台
```

- 更新平台

```
cordova platform update <platform[@version] | directory | fir_url>
```

- 删除平台

```
cordova platform remove | rm <platform>
```

- 运行

```
cordova run <platform name> // 运行在某个平台
cordova run browser // 运行在 browser
```

- 检查预打包环境，若是打包对应的环境时缺少什么配置会提示

```
cordova requirements
```

- 恢复，可以将项目下的platforms、plugins等删除的文件快速还原，前提是 package.json中保存了

```
cordova prepare
```

## 插件命令

- 保存插件

> 可以在项目中增加插件，会保存在 package.json 文件中，增加 —nosave 防止保存插件

```
cordova plugin add <plugin[@<version>] | directory | git_url> --nosave
```

- 保存现有插件

> 若是有一个预先存在的项目，并且想要在项目中保存所有当前添加的插件

```
cordova plugin save
```

- 删除、更新

> 由于plugin并没有提供更新的命令，可以先删除差价再添加

```
cordova plugin remove | rm <plugin>
```

## 打包命令

- 命令

```
cordova build // 打包全部平台
cordova build ios // 打包ios平台
```

