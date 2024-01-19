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
### Solution ###
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