## Unable to start Teamcenter FSC service ##
### Description ###
Teamcenter FSC service can not start when attempted to start from services 
window. It shows an error 


like: process terminated unexpectedly.


FSC.log indicated below error.


 - 2015/07/10-06:22:54,357 UTC - XXXXXXX- Error during initialization. 
com.teamcenter.fms.servercache.FMSServerCache 
com.teamcenter.fms.config.ConfigurationFileException: Could not find the FSC 
configuration file at './fsc.xml'. 
 at 
com.teamcenter.fms.config.FSCConfiguration.<init>(FSCConfiguration.java:398) 
 at 
com.teamcenter.fms.config.FMSConfigManager.getFSCConfig(FMSConfigManager.java:2 
10) 
 at 
com.teamcenter.fms.config.FMSConfigManager.getFSCConfig(FMSConfigManager.java:1 
43) 
 at 
com.teamcenter.fms.servercache.FSCConfigManager.getConfig(FSCConfigManager.java 
:158) 
 at 
com.teamcenter.fms.servercache.FMSServerCache.configure(FMSServerCache.java:540 
) 
 at 
com.teamcenter.fms.servercache.FMSServerCache.instanceMain(FMSServerCache.java: 
299) 
 at 
com.teamcenter.fms.servercache.FMSServerCache.main(FMSServerCache.java:228) 
### Soalution ###



The above error is the side effect of the change in the JAVA version or the 
directory location of JAVA.


seems to be the new JAVA version was installed and is not recognised 


- update JAVA_HOME and JRE64_HOME pointing to new location. 
- Disabled UAC. 
- Check temcenter.ini and update the JAVA path 
- Run TEM and selected MIGRATE JRE , and select new java version path. 
once TEM is sucessfull


Reboot machine. 

Attempt to start the FSC service, It should now start the FSC service just 
fine.

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

