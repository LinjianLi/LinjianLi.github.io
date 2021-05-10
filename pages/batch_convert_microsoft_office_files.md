# Batch Convert Microsoft Office Files

## PPT to PPTX

时间 2018-12-14，系统 Windows 10，测试成功，未发现转换出来的文件有错误。

### 使用方法

随便打开一个 `.pptx` 文件，然后按 `[ALT]` + `[F11]` ，添加模块，复制粘贴以下 VBA 代码进去，然后运行。在弹出框内输入目标文件夹
（打开目标文件夹然后点击上面的路径然后复制粘贴），然后确定。

### 注意事项

1. 前面打开的 `.pptx` 文件不要与需要转换的文件处于同一文件夹内
2. 目标文件夹里的文件不能是打开状态，否则会出现“路径/文件访问错误 允许时错误75”。
3. 无法支持中文路径

### VBA code

```vba
Sub BatchPPTToPPTX()
' Opens each PPT in the target folder and saves as PPTX format

    Dim folder As String
    Dim ppt_name As String
    Dim old_ppt As Presentation

    ' Get the foldername:
    folder = InputBox("Path to the folder containing PPT files to process", "Folder")

    If folder = "" Then
        Exit Sub
    End If

    ' Make sure the folder name has a trailing backslash
    If Right$(folder, 1) <> "\" Then
        folder = folder & "\"
    End If

    ' Are there PPT files there?
    If Len(Dir$(folder & "*.PPT")) = 0 Then
        MsgBox "Bad folder name or no PPT files in folder."
        Exit Sub
    End If

    ' Open and save the presentations
    ppt_name = Dir$(folder & "*.PPT")
    While ppt_name <> ""
        Set old_ppt = Presentations.Open(folder & ppt_name, , , False)
        Call old_ppt.SaveAs(folder & ppt_name & "x", ppSaveAsDefault)
        old_ppt.Close
        ppt_name = Dir$()
    Wend

    MsgBox "DONE"

End Sub
```

## DOC/DOCX to PDF

时间 2021-05-10，系统 Windows 10，测试成功，未发现转换出来的文件有错误。

### 使用方法

随便打开一个 `.docx` 文件，然后按 `[ALT]` + `[F11]` ，添加模块，复制粘贴以下VBA代码进去，然后运行。在弹出框内输入目标文件夹
（打开目标文件夹然后点击上面的路径然后复制粘贴），然后确定。

### 注意事项

1. 前面打开的 `.docx` 文件不要与需要转换的文件处于同一文件夹内
2. 目标文件夹里的文件不能是打开状态，否则会出现“路径/文件访问错误 允许时错误75”。
3. 这份代码**可以**支持中文路径，暂时还没花时间去弄清楚原因

### VBA code

```vba
Sub ConvertWordsToPdfs()
'Updated by Extendoffice 20181123
    Dim xIndex As String
    Dim xDlg As FileDialog
    Dim xFolder As Variant
    Dim xNewName As String
    Dim xFileName As String
    Set xDlg = Application.FileDialog(msoFileDialogFolderPicker)
    If xDlg.Show <> -1 Then Exit Sub
    xFolder = xDlg.SelectedItems(1) + "\"
    xFileName = Dir(xFolder & "*.*", vbNormal)
    While xFileName <> ""
        If ((Right(xFileName, 4)) <> ".doc" Or Right(xFileName, 4) <> ".docx") Then
            xIndex = InStr(xFileName, ".") + 1
            xNewName = Replace(xFileName, Mid(xFileName, xIndex), "pdf")
            Documents.Open FileName:=xFolder & xFileName, _
                ConfirmConversions:=False, ReadOnly:=False, AddToRecentFiles:=False, _
                PasswordDocument:="", PasswordTemplate:="", Revert:=False, _
                WritePasswordDocument:="", WritePasswordTemplate:="", Format:= _
                wdOpenFormatAuto, XMLTransform:=""
            ActiveDocument.ExportAsFixedFormat OutputFileName:=xFolder & xNewName, _
                ExportFormat:=wdExportFormatPDF, OpenAfterExport:=False, OptimizeFor:= _
                wdExportOptimizeForPrint, Range:=wdExportAllDocument, From:=1, To:=1, _
                Item:=wdExportDocumentContent, IncludeDocProps:=True, KeepIRM:=True, _
                CreateBookmarks:=wdExportCreateNoBookmarks, DocStructureTags:=True, _
                BitmapMissingFonts:=True, UseISO19005_1:=False
            ActiveDocument.Close
        End If
        xFileName = Dir()
    Wend
End Sub
```

Refrence: [ExtendOffice: How To Batch Convert Multiple Word Documents To Pdf Files?](https://www.extendoffice.com/documents/word/5551-word-batch-convert-to-pdf.html)
