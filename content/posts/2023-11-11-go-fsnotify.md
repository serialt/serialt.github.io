+++
title = 'Go fsnotify'
date = 2023-11-11T10:24:42+08:00
draft = false

tags = ["Go","fsnotify"]
categories = ["Go 库文档"]

+++

## Go Fsnotify库

官方仓库：github.com/fsnotify/fsnotify

用于监控文件或目录的改变



### 1、官网示例

```go
package main

import (
	"log"

	"github.com/fsnotify/fsnotify"
)

func main() {
	watcher, err := fsnotify.NewWatcher()
	if err != nil {
		log.Fatal(err)
	}
	defer watcher.Close()

	done := make(chan bool)
	go func() {
		for {
			select {
			case event := <-watcher.Events:
				log.Println("event:", event)
				if event.Op&fsnotify.Write == fsnotify.Write {
					log.Println("modified file:", event.Name)
				}
			case err := <-watcher.Errors:
				log.Println("error:", err)
			}
		}
	}()

	err = watcher.Add("/tmp/foo")
	if err != nil {
		log.Fatal(err)
	}
	<-done
}

```

`fsnotify`的使用比较简单:

* 先条用`NewWatcher`创建一个监听器
* 然后条用监听器的Add监听文件或目录
* 如果目录或文件有事件发生，监听器的通道`Events`可以取出事件。如果出现错误，监听器中的通道`Errors`可以取出错误信息。

其实，重命名时会产生两个事件，一个是原文件的`RENAME`事件，一个是新文件的`CREATE`事件。

注意，`fsnotify`使用了操作系统接口，监听器中保存了系统资源的句柄，所以使用后需要关闭。



### 2、事件

上面示例中的事件是`fsnotify.Event`类型：

```go
// fsnotify/fsnotify.go
type Event struct {
  Name string
  Op   Op
}
```

事件只有两个字段，`Name`表示发生变化的文件或目录名，`Op`表示具体的变化。`Op`有 5 中取值：

```go
// fsnotify/fsnotify.go
type Op uint32

const (
  Create Op = 1 << iota
  Write
  Remove
  Rename
  Chmod
)
```

3、监控目录

参考链接：https://blog.csdn.net/finghting321/article/details/102852746

```go
package main;
 
import (
    "github.com/fsnotify/fsnotify"
    "fmt"
    "path/filepath"
    "os"
)
 
type NotifyFile struct {
	watch *fsnotify.Watcher
}
 
func NewNotifyFile() *NotifyFile {
	w := new(NotifyFile)
	w.watch, _ = fsnotify.NewWatcher()
	return w
}
 
//监控目录
func (this *NotifyFile) WatchDir(dir string) {
	//通过Walk来遍历目录下的所有子目录
	filepath.Walk(dir, func(path string, info os.FileInfo, err error) error {
		//判断是否为目录，监控目录,目录下文件也在监控范围内，不需要加
		if info.IsDir() {
			path, err := filepath.Abs(path)
			if err != nil {
				return err
			}
			err = this.watch.Add(path)
			if err != nil {
				return err
			}
			fmt.Println("监控 : ", path)
		}
		return nil
	})
 
	go this.WatchEvent() //协程
}
 
func (this *NotifyFile) WatchEvent() {
	for {
		select {
		case ev := <-this.watch.Events:
			{
				if ev.Op&fsnotify.Create == fsnotify.Create {
					fmt.Println("创建文件 : ", ev.Name)
					//获取新创建文件的信息，如果是目录，则加入监控中
					file, err := os.Stat(ev.Name)
					if err == nil && file.IsDir() {
						this.watch.Add(ev.Name)
						fmt.Println("添加监控 : ", ev.Name)
					}
				}
 
				if ev.Op&fsnotify.Write == fsnotify.Write {
					//fmt.Println("写入文件 : ", ev.Name)
				}
 
				if ev.Op&fsnotify.Remove == fsnotify.Remove {
					fmt.Println("删除文件 : ", ev.Name)
					//如果删除文件是目录，则移除监控
					fi, err := os.Stat(ev.Name)
					if err == nil && fi.IsDir() {
						this.watch.Remove(ev.Name)
						fmt.Println("删除监控 : ", ev.Name)
					}
				}
 
				if ev.Op&fsnotify.Rename == fsnotify.Rename {
					//如果重命名文件是目录，则移除监控 ,注意这里无法使用os.Stat来判断是否是目录了
					//因为重命名后，go已经无法找到原文件来获取信息了,所以简单粗爆直接remove
					fmt.Println("重命名文件 : ", ev.Name)
					this.watch.Remove(ev.Name)
				}
				if ev.Op&fsnotify.Chmod == fsnotify.Chmod {
					fmt.Println("修改权限 : ", ev.Name)
				}
			}
		case err := <-this.watch.Errors:
			{
				fmt.Println("error : ", err)
				return
			}
		}
	}



func main() {
 
	watch := FSNotify.NewNotifyFile()
	watch.WatchDir("G:\\Ferry")
	select {}
	return

}
```

