# module_03 note

## 注意点

- go mod tidy下载的第三方包一般在$GOPATH/pkg/mod下面。如果没有设置GOPATH
  环境变量，其默认值为你的home路径下的go文件夹。这样第三方包就在go文件夹的pkg/mod下
  面。
