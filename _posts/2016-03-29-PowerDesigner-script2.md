---
layout: post
title: [PowerDesigner脚本使用二]
categories: [devtool]
tags: [powerdesigner,名字备注替换,导出excel,脚本]
id: [18719434594944]
fullview: false
---
继第一篇[PowerDesigner脚本使用一](/160328/PowerDesigner-script1)，继续写通过PowerDesigner脚本导出多个Excel文件。如下

**3. 导出多个Excel文件**
```vbscript
'******************************************************************************
'* File:     Table2MutilExcel
'* Title:    pdm export to Mutil excel
'* Purpose:  To export the tables and columns to Excel
'* Model:    Physical Data Model
'* Objects:  Table, Column, View
'* Author:   雪碧心拔凉
'* Created:  2016-03-28
'* Version:  1.0
'******************************************************************************
Option Explicit

'-----------------------------------------------------------------------------
' Main function
'-----------------------------------------------------------------------------
' Get the current active model
Dim Model
Set Model = ActiveModel
If (Model Is Nothing) Or (Not Model.IsKindOf(PdPDM.cls_Model)) Then
   MsgBox "The current model is not an PDM model."
Else
   ' Get the tables collection
   '创建EXCEL APP
   dim beginrow
   DIM excel ,workbook
   set excel = CREATEOBJECT("Excel.Application")
   set workbook = excel.workbooks.add()'添加工作表
   '删除自动生成的三个sheet1，2，3
   workbook.sheets(workbook.sheets.count).delete
   workbook.sheets(workbook.sheets.count).delete
   'workbook.sheets(workbook.sheets.count).delete
   
   ShowProperties Model, workbook
   EXCEL.visible = true
   
       
 End If
'-----------------------------------------------------------------------------
' Show properties of tables
'-----------------------------------------------------------------------------
Sub ShowProperties(mdl, workbook)
   ' Show tables of the current model/package
   
   ' For each table
   output "begin"
   Dim tab,sheet,sumSheet
   set sumSheet = workbook.sheets(workbook.sheets.count)
   dim m
      m=1
      sumSheet.cells(m,1) = "表清单"      
      sumSheet.cells(m, 1).font.bold = 1
      sumSheet.cells(m, 1).horizontalAlignment=3
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).Merge
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).Borders.LineStyle = "1"
      m=m+1
      sumSheet.cells(m,1) = "中文名"
      sumSheet.cells(m,2) = "英文名"
      sumSheet.cells(m,3) = "备注"
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).font.bold = 1
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).Borders.LineStyle = "1"
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).Interior.Color = vbYellow
      sumSheet.Columns(1).ColumnWidth = 20 
      sumSheet.Columns(2).ColumnWidth = 30 
      sumSheet.Columns(3).ColumnWidth = 40 
      sumSheet.name = "表清单"
      
   For Each tab In mdl.tables
      set sheet = workbook.sheets.add
      sheet.name = tab.name
      ShowTable tab,sheet
      
      '设置列宽和自动换行
      sheet.Columns(1).ColumnWidth = 20 
      sheet.Columns(2).ColumnWidth = 30 
      sheet.Columns(3).ColumnWidth = 15 
      sheet.Columns(4).ColumnWidth = 10 
      sheet.Columns(5).ColumnWidth = 10 
      sheet.Columns(6).ColumnWidth = 30
      sheet.Columns(1).WrapText =true
      sheet.Columns(2).WrapText =true
      sheet.Columns(4).WrapText =true
      
      m=m+1
      sumSheet.cells(m,1) = tab.name
      sumSheet.cells(m,2) = tab.code
      sumSheet.cells(m,3) = tab.comment
      sumSheet.Range(sumSheet.cells(m, 1),sumSheet.cells(m, 3)).Borders.LineStyle = "1"      
   Next
   
   output "end"
End Sub
'-----------------------------------------------------------------------------
' Show table properties
'-----------------------------------------------------------------------------
Sub ShowTable(tab, sheet)
   If IsObject(tab) Then
      Dim rangFlag,rowsNum
      rowsNum = 1
      ' Show properties
      Output "================================"
      sheet.cells(rowsNum, 1) = "中文名"
      sheet.cells(rowsNum, 1).font.bold = 1
      sheet.cells(rowsNum, 2) =tab.name
      sheet.Range(sheet.cells(rowsNum, 2),sheet.cells(rowsNum, 6)).Merge
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Borders.LineStyle = "1"
      
      rowsNum = rowsNum + 1
      sheet.cells(rowsNum, 1) = "英文名"
      sheet.cells(rowsNum, 1).font.bold = 1
      sheet.cells(rowsNum, 2) = tab.code
      sheet.Range(sheet.cells(rowsNum, 2),sheet.cells(rowsNum, 6)).Merge
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Borders.LineStyle = "1"
      
      rowsNum = rowsNum + 1
      sheet.cells(rowsNum, 1) = "描述"
      sheet.cells(rowsNum, 1).font.bold = 1
      sheet.cells(rowsNum, 2) = tab.comment
      sheet.Range(sheet.cells(rowsNum, 2),sheet.cells(rowsNum, 6)).Merge
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Borders.LineStyle = "1"
      
      rowsNum = rowsNum + 1
      sheet.cells(rowsNum, 1) = "业务属性集"
      sheet.cells(rowsNum, 1).font.bold = 1
      sheet.cells(rowsNum, 1).horizontalAlignment=3
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Merge
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Interior.Color = vbCyan
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Borders.LineStyle = "1"
      
      
      rowsNum = rowsNum + 1 
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Interior.Color = vbYellow
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).font.bold = 1
      sheet.cells(rowsNum, 1) = "属性名称"
      sheet.cells(rowsNum, 2) = "英文名称"
      sheet.cells(rowsNum, 3) = "数据类型"
      sheet.cells(rowsNum, 4) = "是否必填"
      sheet.cells(rowsNum, 5) = "默认值"
      sheet.cells(rowsNum, 6) = "备注"
      '设置边框
      sheet.Range(sheet.cells(rowsNum, 1),sheet.cells(rowsNum, 6)).Borders.LineStyle = "1"
Dim col ' running column
Dim colsNum
colsNum = 0
      for each col in tab.columns
         rowsNum = rowsNum + 1
         colsNum = colsNum + 1
         sheet.cells(rowsNum, 1) = col.name
         sheet.cells(rowsNum, 2) = col.code
         sheet.cells(rowsNum, 3) = col.datatype
         if(col.mandatory) then
            sheet.cells(rowsNum, 4) = "是"
         else 
            sheet.cells(rowsNum, 4) = "否"
         end if
         sheet.cells(rowsNum, 5) = col.default
         sheet.cells(rowsNum, 6) = col.comment
      next
      sheet.Range(sheet.cells(rowsNum-tab.columns.count,1),sheet.cells(rowsNum,6)).Borders.LineStyle = "1"
      rowsNum = rowsNum + 1
      
      Output "FullDescription: " + tab.Name
   End If
End Sub

```
VB函数

拆分字符:strArr = Split("asdfs(dddsf)","(")

strArr(0)为asdfs
