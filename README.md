![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 创建数据库还原模板语法
#### Create Database Restore Template Syntax
**发布-日期: 2016年10月05日 (评论)**

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
下面是一些快速SQL逻辑（logic），它将所有数据库备份到一个文件夹，然后为每个创建的备份文件创建数据库还原模板语法。
下面是过程：
1. 将所有数据库备份到特定文件夹。
2. 为该文件夹中找到的所有备份文件创建一个列表。
3. 根据找到的每个文件生成还原逻辑（logic）。
结果作为单个XML字符串输出，一旦单击，它将向你显示找到的每个文件中的所有还原语法。
此逻辑（logic）依赖于xp_dirtree和一个简单的变量表。

## English
Here’s some quick SQL logic that will backup all databases to a folder, then create database restore template syntax for each backup file that was created.
This is the process..
1. Backup all databases to a particular folder.
2. Create a list of all backup files found in that folder.
3. Produce restore logic based on each file that is found.


---
## Logic
```SQL
use master;
set nocount on
 
--	backup all databases
--	备份所有的数据库
exec master..sp_msforeachdb
'if (''?'') not in (''tempdb'')
            begin
                        backup database [?] to disk = ''F:\MyBackupFolder\?.bak'' with format, checksum;
            end'
 
--	use backup path and create variable table to store list of backup files
--	使用备份路径并创建变量表来存储备份文件列表
declare @path               varchar(255) = 'F:\MyBackupFolder\'
declare @restoreall       varchar(max)
declare @filetable         table
(
            subdirectory      varchar(255)
,           depth                int
,           [file]               int
)
 
insert    into @filetable
exec master..xp_dirtree @path, 1, 1
 
--	create logic for xml output
--	为xml输出创建逻辑（logic）
set        @restoreall      = ''
select    @restoreall       = @restoreall + 
            'restore database [' + upper(replace(subdirectory, '.bak', '')) + '] 
                        from disk = ''F:\MyBackupFolder\' + subdirectory + '''' + char(10) + 
            'with replace, recovery,' + char(10) +
            ',           move '''' to '''''    + char(10) +
            ',           move '''' to '''''    + char(10) +
            ',           stats = 10'''        + char(10)
from     @filetable
where   subdirectory not in ('master.bak', 'msdb.bak', 'model.bak')  order by subdirectory asc
 
--	produce complete restore syntax for all backup files
--	为所有备份文件生成完整的还原语法
select    @restoreall for xml path (''), type


```
![#](images/create-database-restore-syntax-template.png?raw=true "#")


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

