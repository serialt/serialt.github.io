+++
title = 'Go excelize'
date = 2023-11-11T10:21:42+08:00
draft = false

tags = ["Go","excelize","xls","xlsx"]
categories = ["Go 库文档"]

+++

## Excel库excelize

参考链接：[Go 语言读写 Excel - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/33417413#:~:text=Excelize 是 Go 语言编写的用于操作 Office Excel 文档类库，基于 ECMA-376,标准。 可以使用它来读取、写入由 Microsoft Excel™ 2007 及以上版本创建的 XLSX 文档。)

[Excelize](https://link.zhihu.com/?target=https%3A//github.com/360EntSecGroup-Skylar/excelize) 是 Go 语言编写的用于操作 Office Excel 文档类库，基于 ECMA-376 Office OpenXML 标准。可以使用它来读取、写入由 Microsoft Excel™ 2007 及以上版本创建的 XLSX 文档。相比较其他的开源类库，Excelize 支持写入原本带有图片(表)、透视表和切片器等复杂样式的文档，还支持向 Excel 文档中插入图片与图表，并且在保存后不会丢失文档原有样式，可以应用于各类报表系统中。使用本类库要求使用的 Go 语言为 1.8 或更高版本，完整的 API 使用文档请访问 [godoc.org](https://link.zhihu.com/?target=https%3A//godoc.org/github.com/360EntSecGroup-Skylar/excelize) 或查看[参考文档](https://link.zhihu.com/?target=https%3A//xuri.me/excelize/)。

**GitHub**: [excelize](https://github.com/xuri/excelize)

官方文档：https://xuri.me/excelize

### 1、简单使用

安装

```
go get github.com/xuri/excelize/v2
```

#### 1）创建XLSX

```go
package main

import "github.com/360EntSecGroup-Skylar/excelize"

func main() {
	f := excelize.NewFile()
	// 创建一个工作表
	index := f.NewSheet("Sheet2")
	// 设置单元格的值
	f.SetCellValue("Sheet1", "A2", "Hello world.")
	f.SetCellValue("Sheet1", "B2", 100)
	f.SetCellValue("Sheet1", "B3", "github")
	// 设置工作簿的默认工作表
	f.SetActiveSheet(index)
	// 根据指定路径保存文件
	if err := f.SaveAs("Book1.xlsx"); err != nil {
		println(err.Error())
	}
}

```

结果：

<img src="images/image-20210815152330455.png" alt="image-20210815152330455" style="zoom: 33%;" />



`SetCellValue`函数解释

```go
f.SetCellValue(sheet string, axis string, value interface{})
f.SetCellValue("Sheet1", "B3", "github")
```

* sheet：是sheet表格的的名字。
* axis：包含两部分，字母表示列，数字表示行。
* value：表示值。



#### 2）读取xlsx

```go
func ReadXlsx() {
	filename := "Book1.xlsx"
	f, err := excelize.OpenFile(filename)
	if err != nil {
		fmt.Printf("无法打开xlsx：%v   错误原因：%v", filename, err)
		return
	}

	// 获取工作表中指定单元格的值
	cell, err := f.GetCellValue("Sheet1", "B3")
	if err != nil {
		fmt.Printf("获取 %v 的%v的 值 失败,失败原因：%v", "Sheet1", "B3", err)
	}
	fmt.Printf("读取的内容为：%v", cell)

	// 获取 Sheet1 上所有单元格
	rows, err := f.GetRows("Sheet1")
	if err != nil {
		fmt.Printf("获取 %v的单元格失败,失败原因：%v", "Sheet1", err)
	}
	for _, row := range rows {
		for _, colCell := range row {
			print(colCell, "\t")
		}
		println()
	}
}

```

3）插入一行数据

```go
func (exa *ExcelService) ParseInfoList2Excel(infoList []system.SysBaseMenu, filePath string) error {
	excel := excelize.NewFile()
	excel.SetSheetRow("Sheet1", "A1", &[]string{"ID", "路由Name", "路由Path", "是否隐藏", "父节点", "排序", "文件名称"})
	for i, menu := range infoList {
		axis := fmt.Sprintf("A%d", i+2)
		excel.SetSheetRow("Sheet1", axis, &[]interface{}{
			menu.ID,
			menu.Name,
			menu.Path,
			menu.Hidden,
			menu.ParentId,
			menu.Sort,
			menu.Component,
		})
	}
	err := excel.SaveAs(filePath)
	return err
}
```

4）从表格中导入数据

```go
func PasreExcel2List(filepath string, fileHeader []string, skipHeader bool) (data [][]string, err error) {

	file, err := excelize.OpenFile(filepath)
	if err != nil {
		return
	}
	defer file.Close()

	rows, err := file.GetRows("Sheet1")
	if err != nil {
		return
	}
	if skipHeader {
		if ForEqualStringSlice(rows[0], fileHeader) {
			rows = rows[1:]
		} else {
			err = errors.New("FileHeader not Equal.")
			return
		}
	}
	data = rows
	return
}
```



```go
func (exa *ExcelService) ParseExcel2InfoList() ([]system.SysBaseMenu, error) {
	skipHeader := true
	fixedHeader := []string{"ID", "路由Name", "路由Path", "是否隐藏", "父节点", "排序", "文件名称"}
	file, err := excelize.OpenFile(global.GVA_CONFIG.Excel.Dir + "ExcelImport.xlsx")
	if err != nil {
		return nil, err
	}
	menus := make([]system.SysBaseMenu, 0)
	rows, err := file.Rows("Sheet1")
	if err != nil {
		return nil, err
	}
	for rows.Next() {
		row, err := rows.Columns()
		if err != nil {
			return nil, err
		}
		if skipHeader {
			if exa.compareStrSlice(row, fixedHeader) {
				skipHeader = false
				continue
			} else {
				return nil, errors.New("Excel格式错误")
			}
		}
		if len(row) != len(fixedHeader) {
			continue
		}
		id, _ := strconv.Atoi(row[0])
		hidden, _ := strconv.ParseBool(row[3])
		sort, _ := strconv.Atoi(row[5])
		menu := system.SysBaseMenu{
			GVA_MODEL: global.GVA_MODEL{
				ID: uint(id),
			},
			Name:      row[1],
			Path:      row[2],
			Hidden:    hidden,
			ParentId:  row[4],
			Sort:      sort,
			Component: row[6],
		}
		menus = append(menus, menu)
	}
	return menus, nil
}

func (exa *ExcelService) compareStrSlice(a, b []string) bool {
	if len(a) != len(b) {
		return false
	}
	if (b == nil) != (a == nil) {
		return false
	}
	for key, value := range a {
		if value != b[key] {
			return false
		}
	}
	return true
}
```

