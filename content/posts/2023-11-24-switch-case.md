+++

title = 'switch-case handbook'
date = 2023-11-24T20:22:27+08:00
draft = false

tags = ["switch","case","switch-case"]
categories = ["DevOps"]

+++

# switch-case语法

**shell**

```shell
语法
case 变量 in 
    取值1)
		操作语句
		操作语句
		;;
	取值2)
		操作语句
		操作语句
		;;
	取值3)
		操作语句
		操作语句
		;;
	*)
		操作语句
		操作语句 
		;;
esac 


# example
read -p "Enter string: " str_01
case $str_01 in
   linux|Linux)
     echo "CentOS"
     ;;
   windows|Windows)
     echo "Microsoft"
     ;;
   *)
     echo "Other"
     ;;
esac
```

**go**

```go
func testSwitch3() {
	switch n := 7; n {
	case 1, 3, 5, 7, 9:
		fmt.Println("奇数")
	case 2, 4, 6, 8:
		fmt.Println("偶数")
	default:
		fmt.Println(n)
	}
}

func switchDemo1() {
	finger := 3
	switch finger {
	case 1:
		fmt.Println("大拇指")
	case 4:
		fmt.Println("无名指")
	case 5:
		fmt.Println("小拇指")
	default:
		fmt.Println("无效的输入！")
	}
}
```

**python**

3.10及以后

```python
match term:
    case pattern-1:
         action-1
    case pattern-2:
         action-2
    case pattern-3:
         action-3
    case _:
        action-default


lang = input("What's the programming language you want to learn? ")
match lang:

    case "Python":
        print("You can become a Data Scientist")
    case "go":
        print("You can become a Blockchain developer")
    case "Java":
        print("You can become a mobile app developer")
    case _:
        print("The language doesn't matter, what matters is solving problems.")
```



