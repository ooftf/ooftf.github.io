
# [Homebrew](https://brew.sh/index_zh-cn)

## 安装 homebrew 命令
```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 安装 homebrew 错误

```
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

### 错误原因：

国内网络不可描述的原因, 锅就在目前 GitHub 的 gist 访问不了，所以获取不到安装的脚本文件

### 解决方案：

```shell
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

## 使用 brew install xxx 报错


```shell
Error: Failure while executing; `tar --extract --no-same-owner --file /Users/Salinger/Library/Caches/Homebrew/downloads/5da338c344047ee06f60495e7def31345483e10f19246aad74dca7f5dcea962d--gdbm-1.20.catalina.bottle.tar.gz --directory /private/tmp/d20210624-13993-hs6cjj` exited with 1. Here's the output:
tar: Error opening archive: Failed to open '/Users/Salinger/Library/Caches/Homebrew/downloads/5da338c344047ee06f60495e7def31345483e10f19246aad74dca7f5dcea962d--gdbm-1.20.catalina.bottle.tar.gz'
```
### 错误原因
Bintray 要关闭了, 所以 Homebrew 的归档之后就没再往Bintray 那边传了, 而新版的 Homebrew 已经去除了Bintray相关，使用 ghcr.io 服务了.
正常情况下通过 Homebrew 官网提供的命令安装的用户是无感的, 但是由于国内特殊网络环境的问题, 我使用的是如上文所说的国内镜像, 而国内的镜像是依然指向 Bintray 的, 所以才会出现无法打开归档的错误. 

### 两个解决方案
#### 方案一
临时修改去掉国内的镜像设置: 在 Terminal 中输入下面的命令即可.
```shell
export HOMEBREW_BOTTLE_DOMAIN=''
```
#### 方案二

通过更新profile文件永久修改设置: zsh是~/.zprofile文件，bash要修改~/.bash_profile文件.
