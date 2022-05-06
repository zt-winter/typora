# go相关网络问题集合

```
// gopls等插件无法下载
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
//就该代理地址，google被墙
```

```
//vim coc-nvim插件中coc-go无法在vim界面安装 CocInstall coc-go
cd ~/.config/coc/extensions
//手动进入插件扩展目录，然后使用代理下载
proxychains yarn add coc-go
```

