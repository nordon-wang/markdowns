# 初始化项目
- npm init -y
    -  如果 main 字段没有在 package.json 中出现，那么 Electron 将会尝试加载 index.js 文件
- npm install electron -g
    - 项目中 npm install electron --save-dev 
    


# 打包-方式1
- 安装
    - npm i -S electron-packager
- package增加
    - "package": "electron-packager . --platform=win32 --arch=ia32 --electron-version=1.4.15 --overwrite --ignore=node_modules --ignore=.gitignore" 