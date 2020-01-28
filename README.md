![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Use View .bak Files Like Sys.Master_Files
**Post Date: October 13, 2016**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some SQL logic that takes a folder path (filled with .bak files), and produces a result set similar to sys.master_files.
Ok; so what does this do. This creates a few variable/temp tables then compiles a list of all database names, logical names, and physical names(paths) including all data file names *.mdf, *.ldf, etc.
It essentially builds the list based on the backup files found under the given path. The script will assume backup files are simply named after the database it's self which is an extremely simplified convention, but change accordingly if you see the need. It then goes about creating a 'better filelistonly' table incorporating the database names (extrapolated from the file names), and presents you with the following:
Database Name
Backup Path
Backup File
Type of file
You can then produce a script in accordance with the type of file that is located. </p>      


## SQL-Logic
```SQL
use master;
set nocount on
 
declare @path       varchar(255)    = '\\MyBackupServerName\E$\MyBackupFolder'
declare @filelist   table
(
    [subdirectory]  varchar(255)
,   [depth]     int
,   [file]      int
)
insert  into        @filelist
exec master..xp_dirtree @path, 1, 1
 
if object_id('tempdb..#backupfiles') is not null
        drop table  #backupfiles
create  table       #backupfiles
(
    [id]        int identity(1,1)
,   [database]  varchar(255)
,   [backup_path]   varchar(255)
,   [backup_file]   varchar(255)
)
 
insert  into #backupfiles ([database], [backup_path], [backup_file])
select  upper(replace([subdirectory], '.bak', '')), @path, [subdirectory]
from    @filelist where [subdirectory] not in ('master.bak', 'msdb.bak', 'model.bak')
 
select * from #backupfiles
 
if object_id('tempdb..#better_filelistonly') is not null
    drop table  #better_filelistonly
create  table   #better_filelistonly
(
    [id]            int identity(1,1)
,   [database]      varchar(255)
,   LogicalName     nvarchar(128)
,   PhysicalName        nvarchar(260)
,   [Type]          char(1)
,   FileGroupName       nvarchar(128)
,   Size            numeric(20,0)
,   MaxSize         numeric(20,0)
,   FileID          bigint
,   CreateLSN       numeric(25,0)
,   DropLSN         numeric(25,0)
,   UniqueID        uniqueidentifier
,   ReadOnlyLSN     numeric(25,0)
,   ReadWriteLSN        numeric(25,0)
,   BackupSizeInBytes   bigint
,   SourceBlockSize     int
,   FileGroupID     int
,   LogGroupGUID        uniqueidentifier
,   DifferentialBaseLSN numeric(25,0)
,   DifferentialBaseGUID    uniqueidentifier
,   IsReadOnly      bit
,   IsPresent       bit
,   TDEThumbprint       varbinary(32) 
)
 
declare @get_logical_names  varchar(max)
set @get_logical_names  =''
select  @get_logical_names  =@get_logical_names +
'declare @baseid    int =(select isnull(max(id), 0) from #better_filelistonly)' + char(10) +
'insert into #better_filelistonly' + char(10) +
'(
    LogicalName
,   PhysicalName
,   [Type]
,   FileGroupName
,   Size
,   MaxSize
,   FileID
,   CreateLSN
,   DropLSN
,   UniqueID
,   ReadOnlyLSN
,   ReadWriteLSN
,   BackupSizeInBytes
,   SourceBlockSize
,   FileGroupID
,   LogGroupGUID
,   DifferentialBaseLSN
,   DifferentialBaseGUID
,   IsReadOnly
,   IsPresent
,   TDEThumbprint
)'  + char(10) + 
 
'exec (''restore filelistonly from disk = ''''' + [backup_path] + [backup_file] + ''''''');' + char(10) +
'declare    @lastid int =   (select (max(id) + 1) from #better_filelistonly)' + char(10) +
'update     #better_filelistonly    set [database] = ''' + [database] + ''' where [id] between @baseid and @lastid' + char(10) + 'go' + char(10) + char(10)
from    #backupfiles
 
exec    (@get_logical_names)
 
select * from #better_filelistonly
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

      
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

