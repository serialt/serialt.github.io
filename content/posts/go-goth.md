+++
title = 'Go goph'
date = 2023-11-10T21:20:42+08:00
draft = false

tags = ["Go","goph","ssh","sftp"]
categories = ["Go 库文档"]

+++

# Go goph库

链接地址：

* github.com/melbahja/goph
* github.com/serialt/goph (方便指定端口)

## 特性

* 易于使用，简化 ssh api。
* 支持ssh 密码、私钥，带密码的私钥。
* 支持从本地上传文件和从远程下载文件。
* 支持 ssh 使用 ssh-agent 连接会话。
* 支持增加主机到 known_hosts 文件。



## 安装

``` sh
go get github.com/serialt/goph
```



## 使用示例

### 1、使用执行命令

```go
package main

import (
	"fmt"
	"log"

	"github.com/serialt/goph"
)

var Client *goph.Client

func init() {
	auth, err := goph.Key(os.Getenv("HOME")+"/.ssh/id_rsa", "")
	if err != nil {
		log.Fatalf("cat read the ssh private key: %v", err)
	}

	client, err := goph.New("root", "10.10.16.10", 22, auth)
	if err != nil {
		log.Fatalf("ssh link faild: %v", err)
	}

	Client = client
}

func main() {
	result, err := Client.Run("date")
	fmt.Printf("result: %v\nerr: %v", string(result), err)
    defer Client.Close()
}

```



###  2、从字符串中读取私钥

```go
package main

import (
	"fmt"
	"log"

	"github.com/serialt/goph"
)

var Client *goph.Client
var sshkey = `
-----BEGIN OPENSSH PRIVATE KEY-----
NeY6rItzuvwtiPa5etNiAAAAGnNlcmlhbHQgdHNlcmlhbHRAZ21haWwuY29tAQID
NeY6rItzuvwtiPa5etNiAAAAGnNlcmlhbHQgdHNlcmlhbHRAZ21haWwuY29tAQID
NeY6rItzuvwtiPa5etNiAAAAGnNlcmlhbHQgdHNlcmlhbHRAZ21haWwuY29tAQID
AAAECptYaTKFA2Omsb67+FN2SPr3daBAA0IxpVwv5KYJ1QKWR98JYVP7/WAqffRO
NeY6rItzuvwtiPa5etNiAAAAGnNlcmlhbHQgdHNlcmlhbHRAZ21haWwuY29tAQID
-----END OPENSSH PRIVATE KEY-----
`

func init() {
	auth, err := goph.RawKey(sshkey, "")
	if err != nil {
		log.Fatalf("cat read the ssh private key: %v", err)
	}

	client, err := goph.New("root", "10.10.16.10", 22, auth)
	if err != nil {
		log.Fatalf("ssh link faild: %v", err)
	}

	Client = client
}


func main() {
	result, err := Client.Run("date")
	fmt.Printf("result: %v\nerr: %v", string(result), err)
    defer Client.Close()
}

```

### 3、密钥带密码

```go
auth, err := goph.Key(os.Getenv("HOME")+"/.ssh/id_rsa", "you_passphrase_here")
if err != nil {
	// handle error
}

client, err := goph.New("root", "192.1.1.3", auth)
```



### 4、使用密码

```go
auth, err := goph.UseAgent()
if err != nil {
	// handle error
}

client, err := goph.New("root", "192.1.1.1", auth)
```



### 5、上传和下载文件

```go
// upload local file to remote 
err := client.Upload("/path/to/local/file", "/path/to/remote/file")

// download remote file to local
err := client.Download("/path/to/remote/file", "/path/to/local/file")
```

### 6、执行shell命令

```go
// execute bash commands
out, err := client.Run("bash -c 'printenv'")


// execute bash command whith timeout
context, cancel := context.WithTimeout(ctx, time.Second)
defer cancel()
// will send SIGINT and return error after 1 second
out, err := client.RunContext(ctx, "sleep 5")


// execute bash command whith env variables
out, err := client.Run(`env MYVAR="MY VALUE" bash -c 'echo $MYVAR;'`)

```

### 7、使用goph cmd

`Goph.Cmd` struct is like the Go standard `os/exec.Cmd`.

```go
// Get new `Goph.Cmd`
cmd, err := client.Command("ls", "-alh", "/tmp")

// or with context:
// cmd, err := client.CommandContext(ctx, "ls", "-alh", "/tmp")

if err != nil {
	// handle the error!
}

// You can set env vars, but the server must be configured to `AcceptEnv line`.
cmd.Env = []string{"MY_VAR=MYVALUE"}

// Run you command.
err = cmd.Run()
```

ust like `os/exec.Cmd` you can run `CombinedOutput, Output, Start, Wait`, and [`ssh.Session`](https://pkg.go.dev/golang.org/x/crypto/ssh#Session) methods like `Signal`...



### 8、使用sftp操作文件系统

```go
sftp, err := client.NewSftp()

if err != nil {
	// handle the error!
}

file, err := sftp.Create("/tmp/remote_file")

file.Write([]byte(`Hello world`))
file.Close()

```

For more file operations see [SFTP Docs](https://github.com/pkg/sftp).



## 官方示例

```go
package main

import (
	"bufio"
	"context"
	"errors"
	"flag"
	"fmt"
	"log"
	"net"
	"os"
	osuser "os/user"
	"path/filepath"
	"strings"
	"time"

	"github.com/pkg/sftp"
	"github.com/serialt/goph"
	"golang.org/x/crypto/ssh"
	"golang.org/x/crypto/ssh/terminal"
)

//
// Run command and auth via password:
// > go run main.go --ip 192.168.122.102 --pass --cmd ls
//
// Run command and auth via private key:
// > go run main.go --ip 192.168.122.102 --cmd ls
// Or:
// > go run main.go --ip 192.168.122.102 --key /path/to/private_key --cmd ls
//
// Run command and auth with private key and passphrase:
// > go run main.go --ip 192.168.122.102 --passphrase --cmd ls
//
// Run a command and interrupt it after 1 second:
// > go run main.go --ip 192.168.122.102 --cmd "sleep 10" --timeout=1s
//
// You can test with the interactive mode without passing --cmd flag.
//

var (
	err        error
	auth       goph.Auth
	client     *goph.Client
	addr       string
	user       string
	port       uint
	key        string
	cmd        string
	pass       bool
	passphrase bool
	timeout    time.Duration
	agent      bool
	sftpc      *sftp.Client
)

func init() {
	usr, err := osuser.Current()
	if err != nil {
		fmt.Println("couldn't determine current user.  defaulting to 'root'")
		usr.Username = "root"
	}

	flag.StringVar(&addr, "ip", "127.0.0.1", "machine ip address.")
	flag.StringVar(&user, "user", usr.Username, "ssh user.")
	flag.UintVar(&port, "port", 22, "ssh port number.")
	flag.StringVar(&key, "key", filepath.Join(os.Getenv("HOME"), ".ssh", "id_rsa"), "private key path.")
	flag.StringVar(&cmd, "cmd", "", "command to run.")
	flag.BoolVar(&pass, "pass", false, "ask for ssh password instead of private key.")
	flag.BoolVar(&agent, "agent", false, "use ssh agent for authentication (unix systems only).")
	flag.BoolVar(&passphrase, "passphrase", false, "ask for private key passphrase.")
	flag.DurationVar(&timeout, "timeout", 0, "interrupt a command with SIGINT after a given timeout (0 means no timeout)")
}

func VerifyHost(host string, remote net.Addr, key ssh.PublicKey) error {

	//
	// If you want to connect to new hosts.
	// here your should check new connections public keys
	// if the key not trusted you shuld return an error
	//

	// hostFound: is host in known hosts file.
	// err: error if key not in known hosts file OR host in known hosts file but key changed!
	hostFound, err := goph.CheckKnownHost(host, remote, key, "")

	// Host in known hosts but key mismatch!
	// Maybe because of MAN IN THE MIDDLE ATTACK!
	if hostFound && err != nil {

		return err
	}

	// handshake because public key already exists.
	if hostFound && err == nil {

		return nil
	}

	// Ask user to check if he trust the host public key.
	if askIsHostTrusted(host, key) == false {

		// Make sure to return error on non trusted keys.
		return errors.New("you typed no, aborted!")
	}

	// Add the new host to known hosts file.
	return goph.AddKnownHost(host, remote, key, "")
}

func main() {

	flag.Parse()

	var err error

	if agent || goph.HasAgent() {

		auth, err = goph.UseAgent()

	} else if pass {

		auth = goph.Password(askPass("Enter SSH Password: "))

	} else {

		auth, err = goph.Key(key, getPassphrase(passphrase))
	}

	if err != nil {
		panic(err)
	}

	client, err = goph.NewConn(&goph.Config{
		User:     user,
		Addr:     addr,
		Port:     port,
		Auth:     auth,
		Callback: VerifyHost,
	})

	if err != nil {
		panic(err)
	}

	// Close client net connection
	defer client.Close()

	// If the cmd flag exists
	if cmd != "" {
		ctx := context.Background()
		// create a context with timeout, if supplied in the argumetns
		if timeout > 0 {
			var cancel context.CancelFunc
			ctx, cancel = context.WithTimeout(ctx, timeout)
			defer cancel()
		}

		out, err := client.RunContext(ctx, cmd)

		fmt.Println(string(out), err)
		return
	}

	// else open interactive mode.
	playWithSSHJustForTestingThisProgram(client)
}

func askPass(msg string) string {

	fmt.Print(msg)

	pass, err := terminal.ReadPassword(0)

	if err != nil {
		panic(err)
	}

	fmt.Println("")

	return strings.TrimSpace(string(pass))
}

func getPassphrase(ask bool) string {

	if ask {

		return askPass("Enter Private Key Passphrase: ")
	}

	return ""
}

func askIsHostTrusted(host string, key ssh.PublicKey) bool {

	reader := bufio.NewReader(os.Stdin)

	fmt.Printf("Unknown Host: %s \nFingerprint: %s \n", host, ssh.FingerprintSHA256(key))
	fmt.Print("Would you like to add it? type yes or no: ")

	a, err := reader.ReadString('\n')

	if err != nil {
		log.Fatal(err)
	}

	return strings.ToLower(strings.TrimSpace(a)) == "yes"
}

func getSftp(client *goph.Client) *sftp.Client {

	var err error

	if sftpc == nil {

		sftpc, err = client.NewSftp()

		if err != nil {
			panic(err)
		}
	}

	return sftpc
}

func playWithSSHJustForTestingThisProgram(client *goph.Client) {

	fmt.Println("Welcome To Goph :D")
	fmt.Printf("Connected to %s\n", client.Config.Addr)
	fmt.Println("Type your shell command and enter.")
	fmt.Println("To download file from remote type: download remote/path local/path")
	fmt.Println("To upload file to remote type: upload local/path remote/path")
	fmt.Println("To create a remote dir type: mkdirall /path/to/remote/newdir")
	fmt.Println("To exit type: exit")

	scanner := bufio.NewScanner(os.Stdin)

	fmt.Print("> ")

	var (
		out   []byte
		err   error
		cmd   string
		parts []string
	)

loop:
	for scanner.Scan() {

		err = nil
		cmd = scanner.Text()
		parts = strings.Split(cmd, " ")

		if len(parts) < 1 {
			continue
		}

		switch parts[0] {

		case "exit":
			fmt.Println("goph bye!")
			break loop

		case "download":

			if len(parts) != 3 {
				fmt.Println("please type valid download command!")
				continue loop
			}

			err = client.Download(parts[1], parts[2])

			fmt.Println("download err: ", err)
			break

		case "upload":

			if len(parts) != 3 {
				fmt.Println("please type valid upload command!")
				continue loop
			}

			err = client.Upload(parts[1], parts[2])

			fmt.Println("upload err: ", err)
			break

		case "mkdirall":

			if len(parts) != 2 {
				fmt.Println("please type valid mkdirall command!")
				continue loop
			}

			ftp := getSftp(client)

			err = ftp.MkdirAll(parts[1])
			fmt.Printf("mkdirall err(%v) you can check via: stat %s\n", err, parts[1])

		default:

			command, err := client.Command(parts[0], parts[1:]...)
			if err != nil {
				panic(err)
			}
			out, err = command.CombinedOutput()
			fmt.Println(string(out), err)
		}

		fmt.Print("> ")
	}
}

```

