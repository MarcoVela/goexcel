﻿# GoExcel

Manipulate excel files using all the features of golang

Fork from [github.com/maxim-morozenko/excel]


# dependency

[github.com/go-ole/go-ole][ole]

# install

go get github.com/MarcoVela/goexcel

# example
``` go
package main

import (
	"runtime"
	"fmt"
	"time"
	"github.com/MarcoVela/goexcel"
)

func main() {
	runtime.GOMAXPROCS(1)
	option := excel.Option{"Visible": true, "DisplayAlerts": true, "ScreenUpdating": true}
	xl, _ := excel.New(option)
	defer xl.Quit()

	sheet, _ := xl.Sheet(1)
	defer sheet.Release()
	sheet.Cells(1, 1, "hello")
	sheet.PutCell(1, 2, 2006)
	sheet.MustCells(1, 3, 3.14159)

	cell := sheet.Cell(5, 6)
	defer cell.Release()
	cell.Put("go")
	cell.Put("font", map[string]interface{}{"name": "Arial", "size": 26, "bold": true})
	cell.Put("interior", "colorindex", 6)

	sheet.PutRange("a3:c3", []string {"@1", "@2", "@3"})
	rg := sheet.Range("d3:f3")
	defer rg.Release()
	rg.Put([]string {"~4", "~5", "~6"})

	urc := sheet.MustGet("UsedRange", "Rows", "Count").(int32)
	println("str:"+sheet.MustCells(1, 2), sheet.MustGetCell(1, 2).(float64), cell.MustGet().(string), urc)

	cnt := 0
	sheet.ReadRow("A", 1, "F", 9, func(row []interface{}) (rc int) {    //"A", 1 or 1, 9 or 1 or nothing
		cnt ++
		fmt.Println(cnt, row)
		return                                                                   //-1: break
	})

	time.Sleep(2000000000)

	//Sort
	cells := excel.GetIDispatch(sheet, "Cells")
	cells.CallMethod("UnMerge")
	sort := excel.GetIDispatch(sheet, "Sort")
	sortfields := excel.GetIDispatch(sort, "SortFields")
	sortfields.CallMethod("Clear")
	sortfields.CallMethod("Add", sheet.Range("f:f").IDispatch, 0, 2)
	sort.CallMethod("SetRange", cells)
	sort.PutProperty("Header", 1)
	sort.CallMethod("Apply")

	//Chart
	shapes := excel.GetIDispatch(sheet, "Shapes")
	_chart, _ := shapes.CallMethod("AddChart", 65)
	chart := _chart.ToIDispatch()
	chart.CallMethod("SetSourceData", sheet.Range("a1:c3").IDispatch)

	//AutoFilter
	cells.CallMethod("AutoFilter")
	excel.Release(sortfields, sort, cells, chart, shapes)

	time.Sleep(3000000000)
	xl.SaveAs("test_excel.xls")    //xl.SaveAs("test_excel", "html")
}

```

# license

The [BSD 3-Clause license][bsd]

[ole]: http://github.com/go-ole/go-ole
[bsd]: http://opensource.org/licenses/BSD-3-Clause




