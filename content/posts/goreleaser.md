---
title: "利用 goreleaser 发布自己的项目到 github"
date: 2022-10-18T19:12:11+08:00
draft: false
tags: ["goreleaser", "github"]
categories: ["golang", "最佳实践"]
featuredImagePreview: /images/goreleaser.png
---



经常会用 golang 做一些工具并开源给大家用。如果自己发版，太麻烦。 而且还要交叉编译各个平台。 在网上找到了一个golang的发布工具，只需要在本地输入一行命令就能直接编译好所有的二进制文件。并发布到 github。
[goreleaser 官网地址](https://goreleaser.com/quick-start/)


### 最后的效果如下：

![image-20221018105905803](/note/images/goreleaser.png)


### 在你的项目下新建 `.goreleaser.yaml` 文件

```yaml
# This is an example .goreleaser.yml file with some sensible defaults.
# Make sure to check the documentation at https://goreleaser.com
before:
  hooks:
    # You may remove this if you don't use go modules.
    - go mod tidy
    # you may remove this if you don't need go generate
    - go generate ./...
builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - linux
      - windows
      - darwin
    goarch:
      - amd64
      - arm64
    gobinary: "garble"
archives:
  - replacements:
      linux: Linux
      windows: Windows
      amd64: x86_64
      darwin: MacOS
checksum:
  name_template: 'checksums.txt'
snapshot:
  name_template: "{{ incpatch .Version }}-next"
changelog:
  sort: asc
  filters:
    exclude:
      - '^docs:'
      - '^test:'

# modelines, feel free to remove those if you don't want/use them:
# yaml-language-server: $schema=https://goreleaser.com/static/schema.json
# vim: set ts=2 sw=2 tw=0 fo=cnqoj

```



### 在你的项目下新建 `.github/workflows/release.yaml` 文件

```yml
name: goreleaser

on:
  push:
    # run only against tags
    tags:
      - '*'

permissions:
  contents: write
  # packages: write
  # issues: write

jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - run: git fetch --force --tags
      - uses: actions/setup-go@v3
        with:
          go-version: '>=1.19.2'
          cache: true
      # More assembly might be required: Docker logins, GPG, etc. It all depends
      # on your needs.
      - run: go install mvdan.cc/garble@latest
      - uses: goreleaser/goreleaser-action@v2
        with:
          # either 'goreleaser' (default) or 'goreleaser-pro':
          distribution: goreleaser
          version: latest
          args: release --rm-dist
        env:
          GITHUB_TOKEN: ${{ secrets.GORELEASER_GITHUB_TOKEN }}
          # Your GoReleaser Pro key, if you are using the 'goreleaser-pro'
          # distribution:
          # GORELEASER_KEY: ${{ secrets.GORELEASER_GITHUB_TOKEN }}

```

⚠️注意： 这里需要 GITHUB_TOKEN 。需要自己提前配置。