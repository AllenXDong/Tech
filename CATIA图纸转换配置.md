## CATIA图纸转换配置

### Dispatcher处理程序

- TaskPrep

```java
package com.qzql.teamcenter.dispatcher.translators.catiadrawingtopdf;

import com.qzql.teamcenter.dispatcher.common.FileExtension;
import com.qzql.teamcenter.dispatcher.common.TaskPrepItemRevisionDataset;
import com.teamcenter.translationservice.task.TranslationTask;

public class TaskPrep extends TaskPrepItemRevisionDataset {
  public TranslationTask prepareTask() throws Exception {
    try {
      init(false);
      this.subjectPre = "CATDrawing to PDF \"" + this.sourceItemRev.get_object_string() + " " + this.sourceDataset.get_object_name() + "\"";
      FileExtension inputFile = new FileExtension(getFileToStaging());

      addOptionTaskId();
      addOptionObjectName(inputFile.getNameWithoutSuffix());
      addOptionInputFile(inputFile);
      addOptionOutputDir();
      logEnd();
      return this.translationTask;
    } catch (Exception exception) {
      failed(exception);
      throw exception;
    } 
  }
}
```



- DatabaseOperation

```java
package com.qzql.teamcenter.dispatcher.translators.catiadrawingtopdf;

import com.qzql.teamcenter.dispatcher.common.DatabaseOperationItemRevisionDataset;
import com.qzql.teamcenter.dispatcher.common.EnTcSoaErrorStackHandler;
import com.teamcenter.services.strong.core._2007_01.DataManagement;
import com.teamcenter.soa.client.model.ModelObject;
import com.teamcenter.soa.client.model.strong.Dataset;
import com.teamcenter.soa.client.model.strong.WorkspaceObject;
import java.util.HashMap;
import java.util.Map;

public class DatabaseOperation extends DatabaseOperationItemRevisionDataset {
  public void load() throws Exception {
    try {
    	
    	
      init();
      this.subjectPre = "CATDrawing to PDF \"" + this.sourceItemRev.get_object_string() + " " + this.sourceDataset.get_object_name() + "\"";
      if (isSuccessful()) {
        Dataset convertedDataset = updateOrCreateDataset("IMAN_manifestation", "PDF", "PDF_Reference", "pdf", "PDF");
        setReleaseStatusFromSourceItemRev((WorkspaceObject)convertedDataset);
        setOwnershipFromSourceDataset((WorkspaceObject)convertedDataset);
        checkInSourceDataset();
        successful();
      } else {
        failed(getLogContent());
      } 
    } catch (Exception exception) {
      failed(exception);
    } 
    processExceptionToThrow();
  }
}
```



### Properties配置文件

```properties
Translator.QZQL.catiaparttostep.Prepare=com.qzql.teamcenter.dispatcher.translators.catiaparttostep.TaskPrep
Translator.QZQL.catiaparttostep.Load=com.qzql.teamcenter.dispatcher.translators.catiaparttostep.DatabaseOperation
```

![image-20231219135401017](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20231219135401017.png)

### 首选项设置

- 提供商

![image-20231219134443297](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20231219134443297.png)

- 转换服务

![image-20231219134724004](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20231219134724004.png)

- 支持数据集类型

![image-20231219134702531](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20231219134702531.png)

- 设置完成后可在界面上手动创建转换请求

![1702966957179](C:\Users\Administrator\Documents\WeChat Files\a316739861\FileStorage\Temp\1702966957179.png)

### 图纸转换程序

- Python及转换插件安装

[pycatia转换插件](https://github.com/AllenXDong/pycatia)

- CatiaDrawing to PDF

```python
#! /usr/bin/python3.9
# author: evereux
# contact: evereux@gmail.com

"""
    Save Drawings To PDF

    Description
    ===========
    Loops through all the files (.CATDrawing) of a given directory and saves to
    PDF.

    For CATDrawings the Document.export_data() method exports each sheet to a
    single PDF. This script uses pypdf to merge these single sheets into a
    single pdf for each drawing.

    Requirements
    ============
    python >= 3.9
    pycatia >= 0.6.4
    pypdf
    CATIA V5 running
    A network accessible folder that contain your CATDrawings.

    Documentation
    =============
    https://pycatia.readthedocs.io

    More examples and user scripts can be found at:
    https://github.com/evereux/pycatia/tree/master/examples
    https://github.com/evereux/pycatia/tree/master/user_scripts
"""

##########################################################
# insert syspath to project folder so examples can be run.
# for development purposes.
import os
import sys

sys.path.insert(0, os.path.abspath("..\\pycatia"))
##########################################################
from pathlib import Path

from pypdf import PdfWriter

from pycatia import CATIADocHandler
from pycatia.drafting_interfaces.drawing_document import DrawingDocument

def export_pdf(inputFile,outputFile):
    with CATIADocHandler(file_name=inputFile) as handler:
        caa = handler.catia
        part_document = handler.document
        drawing = DrawingDocument(part_document.com_object)
        drawing.export_data(outputFile, 'pdf', overwrite=True)

    # close document
    part_document.close()
    

if ( __name__ == '__main__' )  or  ( __name__ == 'main' ) :
    inputFile = sys.argv[1];
    outputFile = sys.argv[2];
    print("inputFile="+inputFile);
    print("outputFile="+outputFile);
    export_pdf(inputFile,outputFile);


```

- python生成exe可执行程序

> ```
> #透過pip安裝pyinstaller
> pip install pyinstaller#如果失敗可以使用以下的方法進行安裝
> pip install https://github.com/pyinstaller/pyinstaller/archive/develop.tar.gz#目前可支援Python 2.7 and 3.3—3.6
> ```

```bash
pyinstaller -F hello.py
```

[【Python】使用 PyInstaller 將 Python打包成 exe 檔](https://medium.com/pyladies-taiwan/python-%E5%B0%87python%E6%89%93%E5%8C%85%E6%88%90exe%E6%AA%94-32a4bacbe351)

### Translator配置

1. translator.xml(D:\Siemens\DC\Module\conf\translator.xml)增加

![image-20231219141640740](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20231219141640740.png)

```xml
<CatiaDrawingToPDF provider="QZQL" maxlimit="1" service="catiadrawingtopdf" isactive="true" OutputNeeded="false" wrapperclass="com.teamcenter.tstk.translator.DefaultTranslator">
    <TransExecutable name="catiadrawingtopdf.bat" dir="&MODULEBASE;/Translators/catiadrawingtopdf"/>
    <Options>
      <Option name="clientoption" optionkey="taskId" string="" value="" description="Task Id."/>
      <Option name="clientoption" optionkey="objectName" string="" value="" description="Dataset object name."/>
      <Option name="clientoption" optionkey="inputPart"  string="" value="" description="NX Database Input Path."/>
      <Option name="clientoption" optionkey="outputDir"  string="" value="" description="Full path to the output directory."/>
    </Options>
     <TransErrorStrings>
      <TransInputStream string="ERROR"/>
      <TransInputStream string="error"/>
      <TransErrorStream string="ERROR"/>
      <TransErrorStream string="error"/>
     </TransErrorStrings>
  </CatiaDrawingToPDF>
```

2. Tranlator文件夹（D:\Siemens\DC\Module\Translators）增加

```bash
@echo off
set TRANSLATOR_NAME=%~n0
:: # ==================================================================== #
::   SCRIPT TO RUN THE TRANSLATOR
:: # ==================================================================== #

title EISENMANN.%TRANSLATOR_NAME% translator

:: Read the parameters ----------------------------------------------------
set TASK_ID=%~1
set OBJECT_NAME=%~2
set INPUT_PART=%~3
set OUTPUT_DIR=%~4
set THIS_CALL="%~f0" %*

set LOG_FILE=%OUTPUT_DIR%\en_translator.log
set WORKING_DIR=%TEMP%\%TASK_ID%
set TRANSLATOR_HOME=%~dp0

echo %DATE% %TIME% Teamcenter Dispatcher Client - Provider: EISENMANN - Service: %TRANSLATOR_NAME% > "%LOG_FILE%"
echo DispatcherModule-Server: %COMPUTERNAME% >> "%LOG_FILE%"
echo Call: %THIS_CALL% >> "%LOG_FILE%"
echo. >> "%LOG_FILE%"


:: Display the parameters -------------------------------------------------
echo.
echo ===============================================================================
echo %0
echo 1. parameter: taskId=%TASK_ID%
echo 2. parameter: objectName=%OBJECT_NAME%
echo 3. parameter: inputPart=%INPUT_PART%
echo 4. parameter: outputDir=%OUTPUT_DIR%
echo current working directory: %cd%
echo ===============================================================================


echo.
echo Execute ----------------------------------------------------------------
set ACT_CMD=%~dp0CatiaDrawingToPDF.exe %INPUT_PART% %OUTPUT_DIR%\%OBJECT_NAME%.pdf
echo %ACT_CMD%
echo. >> "%LOG_FILE%"
echo %ACT_CMD% >> "%LOG_FILE%"
echo. >> "%LOG_FILE%"
%ACT_CMD%
set EXITVALUE=%ERRORLEVEL%
if "%EXITVALUE%" neq "0" (
 echo EN_TRANSLATOR_ERROR
 echo failed: %ACT_CMD%
 echo. >> "%LOG_FILE%"
 echo ERRORLEVEL %EXITVALUE% >> "%LOG_FILE%"
 goto end
)



:end

:: Display if successful or not -------------------------------------------
echo.
echo.
if "%EXITVALUE%" equ "0" (
 echo return %EXITVALUE% - successful
 echo Result: successful >> "%LOG_FILE%"
) else (
 echo EN_TRANSLATOR_ERROR
 echo.
 type "%LOG_FILE%"
 echo.
 echo return %EXITVALUE% - failed
 echo Result: failed >> "%LOG_FILE%"
)
echo.

exit %EXITVALUE%
```

### 注意事项

如果Dispatcher同时发起catiatopdf和catiatostp两个请求，服务器会发生错误，需修改配置文件将<b>MaximumTasks</b>改为<b>1</b>

> Module
>
> Edit the **transmodule.properties** file located in the *DISP_ROOT***\Module\conf** directory.

```properties
#============================================================================
# Description:
#   MaximumTasks property controls the maximum number of tasks allowed to 
#   in the module at a particular instant
#
#   Use this property to control the maximum number of services/translations
#   running on the machine
#
# Options:
#   3 -  (Default)
#   Or any other non-zero number depending on the hardware configuration 
#   of the local machine
#============================================================================
#
MaximumTasks=1
```

