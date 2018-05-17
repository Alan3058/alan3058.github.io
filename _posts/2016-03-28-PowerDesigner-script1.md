---
layout: post
title: [PowerDesigner脚本使用一]
categories: [开发工具]
tags: [powerdesigner,名字备注替换,导出excel,脚本]
id: [18717232594944]
fullview: false
---
这段时间经常使用powerdesigner，期间有遇到导出excel或者转名称为备注等需求。于是网上搜索了下，似乎新版本也只支持部分功能，最后发现工具下有段运行脚本的功能（`PowerDesigner->Tools->Execute Commands->Edit/Run Scripts`）。这应该是设计者留给我们使用者的入口吧![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)，自己搜了几段脚本，越看越眼熟，这尼玛不是蛋疼的VB吗？大学学过的东西还是用上了，当然我也没有完全记住，只能记住大概、能看懂就是了，不过已经足够了。。。![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)。其实程序员哥哥妹妹应该很容易看懂啦。。。

**1. Name信息复制到备注信息中**
```vbscript
'******************************************************************************
'* File:     Name2Comment
'* Title:    pdm export to excel
'* Purpose:  把pdm中name自动添加到comment里面，如果comment为空,
'*           则填入name;如果不为空,则保留不变,这样可以避免已有的注释丢失.
'* Model:    Physical Data Model
'* Objects:  Table, Column, View
'* Author:   雪碧心拔凉
'* Created:  2016-03-28
'* Version:  1.0
'******************************************************************************
Option   Explicit 
ValidationMode   =   True 
InteractiveMode   =   im_Batch 

Dim   mdl 

'获取当前model 
Set   mdl   =   ActiveModel 
If   (mdl   Is   Nothing)   Then 
      MsgBox   "There   is   no   current   Model " 
ElseIf   Not   mdl.IsKindOf(PdPDM.cls_Model)   Then 
      MsgBox   "The   current   model   is   not   an   Physical   Data   model. " 
Else 
      ProcessFolder   mdl 
End   If 

'复制name到comment列（针对table,view,column）
Private   sub   ProcessFolder(folder)    
      Dim   Tab   'running     table    
      for   each   Tab   in   folder.tables    
            if   not   tab.isShortcut then
                     if  trim(tab.comment)="" then'如果有表的注释,则不改变它.如果没有表注释.则把name添加到注释里面.
                        tab.comment   =   tab.name
                     end if  
                  Dim   col   '   running   column    
                  for   each   col   in   tab.columns   
                        if trim(col.comment)="" then '如果col的comment为空,则填入name,如果已有注释,则不添加;这样可以避免已有注释丢失.
                           col.comment=   col.name   
                        end if 
                  next    
            end   if    
      next    
  
      Dim   view   'running   view    
      for   each   view   in   folder.Views    
            if   not   view.isShortcut and trim(view.comment)=""  then    
                  view.comment   =   view.name    
            end   if    
      next    
  
      '   go   into   the   sub-packages    
      Dim   f   '   running   folder    
      For   Each   f   In   folder.Packages    
            if   not   f.IsShortcut   then    
                  ProcessFolder   f    
            end   if    
      Next    
end   sub
```

**2. 导出单个Excel文件例子**
```vbscript
'******************************************************************************
'* File:     Table2Excel
'* Title:    pdm export to excel
'* Purpose:  To export the tables and columns to Excel
'* Model:    Physical Data Model
'* Objects:  Table, Column, View
'* Author:   雪碧心拔凉
'* Created:  2016-03-28
'* Version:  1.0
'******************************************************************************
Option Explicit
   Dim rowsNum
   rowsNum = 0
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
   DIM EXCEL, SHEET
   set EXCEL = CREATEOBJECT("Excel.Application")
   EXCEL.workbooks.add(-4167)'添加工作表
   EXCEL.workbooks(1).sheets(1).name ="test"
   set sheet = EXCEL.workbooks(1).sheets("test")

   ShowProperties Model, SHEET
   EXCEL.visible = true
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
 End If
'-----------------------------------------------------------------------------
' Show properties of tables
'-----------------------------------------------------------------------------
Sub ShowProperties(mdl, sheet)
   ' Show tables of the current model/package
   rowsNum=0
   beginrow = rowsNum+1
   ' For each table
   output "begin"
   Dim tab
   For Each tab In mdl.tables
      ShowTable tab,sheet
   Next
   if mdl.tables.count > 0 then
        sheet.Range("A" & beginrow + 1 & ":A" & rowsNum).Rows.Group
   end if
   output "end"
End Sub
'-----------------------------------------------------------------------------
' Show table properties
'-----------------------------------------------------------------------------
Sub ShowTable(tab, sheet)
   If IsObject(tab) Then
     Dim rangFlag
      rowsNum = rowsNum + 1
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

待续中。。。 [PowerDesigner脚本使用二](/160329/PowerDesigner-script2)
