# npm 版本号控制


## npm 版本号控制

### 版本运算符

版本运算符指定了一定范围的版本。
主要有**~**、**^**、**-**、**<**、**<=**、**>**、**>=**、**=**版本运算符。

### ~ 版本号 ----- 指定主版本号或者次版本号相同

~ + **只含主版本** --- 主版本相同；
~ + **含有次版本** --- 主版本和次版本号相同。

| 版本范围 | 匹配版本                     |
| :------- | :--------------------------- |
| ~3       | 3.x 或者 3.0.0 <= v < 4.0.0  |
| ~3.1     | 3.1.x 或者 3.1.0 <= v <3.2.0 |
| ~3.1.2   | 3.1.2 < v < 3.2.0            |

指定的版本范围含有预发布版本，只会匹配和完整版本号相同的预发布版本。
~3.1.3-beta.2 匹配 3.1.3-beat.3 不匹配 3.1.4-beat-2

```
npm i lodash@~3 # 安装 3.10.1
npm i lodash@~3.9 # 安装 3.9.3
npm i lodash@~3.9.1 # 安装 3.9.3
npm i lodash@~3.8.0 # 安装 3.8.0
```

### ^ 版本号 --- 第一个*非零* 版本号相同

| 版本范围 | 匹配版本           | 补充                          |
| :------- | :----------------- | :---------------------------- |
| ^3.1.5   | 3.1.5 <= v < 4.0.0 |                               |
| ^0.3.6   | 0.3.6 <= v < 0.4.0 |                               |
| ^0.0.2   | 0.0.2 <= v < 0.0.3 |                               |
| ^3.x.x   | 3.0.0 <= v < 4.0.0 | 版本号缺少的位置，会被 0 填充 |
| ^4.2.x   | 4.2.0 <= v < 4.3.0 |                               |

npm 安装包时，默认使用 ^ 匹配版本。

安装主版本号为 3 的最新版本：

```
npm i lodash@^3 # 安装 3.10.1
npm i lodash@^3.9 # 安装 3.10.1
npm i lodash@^3.8.0 # 安装 3.10.1
```

### ~ vs ^

| 版本范围 | 含义          | 匹配的版本         | 说明               |
| :------- | :------------ | :----------------- | :----------------- |
| ~3.3.0   | 与 3.3.0 相似 | 3.3.0 <= v < 3.4.0 | 主版本和次版本相同 |
| ^3.3.0   | 与 3.3.0 兼容 | 3.3.0 <= v < 4     | 主版本相同         |

同一个版本号，^ 能匹配的范围大些，更加激进。

例子

```
npm i lodash@^3.3.0 # 安装 3.10.1
npm i lodash@~3.3.0 # 安装 3.3.1
```

**~** 和 ≈ 差不多，可将 ~ 理解成**相似**，这样就分辨和理解了，~指定的是**相似版本**。
**^** 可理解成**兼容版本**。

### - 指定精确范围

| 版本范围      | 匹配版本            | 补充                    |
| :------------ | :------------------ | :---------------------- |
| 2.0.0 - 3.2.7 | 2.0.0 <= v <= 3.2.7 | - 前后有空格            |
| 0.4 - 3       | 0.4.0 <= v <= 3.0.0 | 缺少的版本号，被 0 填充 |

```
npm i vue@"1 - 1.9" # 安装 1.0.28
```

### 版本号比较器

| 版本范围 | 匹配版本              | 补充 |
| :------- | :-------------------- | :--- |
| <2.2.0   | 小于 2.2.0 的版本     |      |
| <=2.0.0  | 小于等于 2.0.0 的版本 |      |
| >4.2.0   | 大于 4.2.0 的版本     |      |
| >=4.2.0  | 大于等于 4.2.0 的版本 |      |
| =4.3.0   | 等于 4.3.0 的版本     |      |

\ 是转义字符。

```
npm i lodash@\<3.5 # 安装 3.4.0
npm i lodash@\<=3.5 # 安装 3.5.0
npm i lodash@\>3.5 # 安装 4.17.11
npm i lodash@\>=3.5 # 安装 4.17.11
npm i vue@">1 <2.3" # 安装 2.2.6
```

## package-lock.json

在直接更新 package.json 和 package-loc.json 这两个文件后，npm install 是可以直接覆盖掉原先的版本的，所以在协作开发时，这两个文件如果有更新，你的开发环境应该 npm install 一下才对。

### 运行原理:

如果改了 package.json，且 package.json 和 lock 文件不同，那么执行`npm i`时 npm 会根据 package 中的版本号以及语义含义去下载最新的包，并更新至 lock。如果两者是同一状态，那么执行`npm i`都会根据 lock 下载，不会理会 package 实际包的版本是否有新。

### npm-shrinkwrap.json

该文件是通过运行`npm shrinkwrap`命令产生的

### npm-shrinkwrap.json 与 package-lock.json 的区别与联系

#### 从 npm 版本看

package-lock.json 是 npm5 的新特性，也不向前兼容，如果 npm 版本是 4 或以下，那还是使用 npm-shrinkwrap.json 吧

#### 从 npm 处理机制来看

1. 在一个项目里，如果本身不存在这两个文件，那么在运行 npm install 时，会自动生成一个 package-lock.json，或者在初始化一个项目 npm init 时，也会生成 package-lock.json，安装信息会依据该文件进行，而不是单纯按照 package.json，这两个文件的优先级都比 package.json 高
2. 如果项目两个文件都存在，那么安装的依赖是依据 npm-shrinkwrap.json 来的，而忽略 package-lock.json
3. 运行命令 npm shrinkwrap 后，如果项目里不存在 package-lock.json，那么会新建一个 npm-shrinkwrap.json 文件，如果存在 package-lock.json，那么会把 package-lock.json 重命名为 npm-shrinkwrap.json

## 版本控制最佳实践

1. package.json 包的更新策略不要太激进, 也不要太保守, 可以锁定前两位版本, 最后一位版本可变, 因为最后一位版本往往是修复 BUG
2. package-lock.json 应该加入版本仓库, 这样可以保证测试的依赖库和上线时候的依赖库一致

## 全局 node_modules 权限问题

first check who owns the directory

```
ls -la /usr/local/lib/node_modules
```

it is denying access because the node_module folder is owned by root

```
drwxr-xr-x   3 root    wheel  102 Jun 24 23:24 node_modules
```

so this needs to be changed by changing root to your user but first run command below to check your current user How do I get the name of the active user via the command line in OS X?

```
id -un
```

OR

```
whoami
```

then change owner

```
sudo chown -R [owner]:[owner] /usr/local/lib/node_modules
```

OR

```
sudo chown -R ownerName: /usr/local/lib/node_modules
```

OR

```
sudo chown -R $USER /usr/local/lib/node_modules
```

