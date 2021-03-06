# Managing Applications 

 - Backup and Restore

 - Deployment History and Rollbacks

 - Migrating an Application to another Gear

--

## Backup and Restore#
- ###Introduction to Snapshots##
   - #####Application snapshots are used to back up and restore applications.
   - #####Snapshots are stored in **tar.gz** files, which contain the application and all local files, including log files
   - #####A binary deployment file is a snapshot of an application used for deployment without using Git. 
	- #####Each snapshot has the same top-level structure:
   		- build-dependencies/ ------> cartridge-specific content determined by the contents of the managed_files.yml file
   		- dependencies/ ------------> cartridge-specific content determined by the contents of the managed_files.yml file
   		- repo/ --------------------> the files that make up the application source code

Notes:
Application backups and user data are not stored on OpenShift Enterprise servers. These files are only stored on the local system

--
## Backing up an application ##


	$ rhc snapshot save firstphp
	Pulling down a snapshot to firstphp.tar.gz...
	Waiting for stop to finish
	Done
	Creating and sending tar.gz
	Done
	

- #####This command creates a gzipped tar file of your application and of any locally-created log and data files
- #####This snapshot is downloaded to your local machine
- #####The default filename for the snapshot is **Application_Name.tar.gz**
- #####You can override this path and filename with the **-f** or **--filepath** option

Notes:
- The directory structure that exists on the server is maintained in the downloaded archive
- This command will stop your application for the duration of the backup process.

--
## Restoring a backup#

- ####To restore an application snapshot, run the following command####

```
$ rhc snapshot restore firstphp -f firstphp.tar.gz
Restoring from snapshot firstphp.tar.gz
Removing old data dir: ~/app-root/data/*
Restoring ~/app-root/data>
```

####Warnings

- #####The **rhc snapshot restore** command overwrites the remote **Git** repository#

- #####This command will stop your application for the duration of the restore process#

Notes:
- Restoring from an application snapshot restores the Git repository, the application data directories, and the log files found in the specified archive. When the restoration is complete, the deployment script is run on the restored repository as though git push was run.

- Importing snapshot data into a local environment can delete local content, for example a user table in a database. 
If you are unsure of the effect a snapshot import could have on local data, use SSH to access an application and create the backup directly.

--
## Deployment History and Rollbacks#

- ####OpenShift supports deployment history complete with the ability to rollback to a previous version of your application. 

- ####Specifying the number of deployments to keep ( By default, will only track the last deployment)
```
$ cd APP_NAME
$ rhc app-configure --keep-deployments 10
```
### Rolling back#

```
$ rhc deployment-list
3:58 PM, deployment 9d8f96ac
4:04 PM, deployment cc200f43
4:07 PM, deployment a9b1f5bf
```
```
$ rhc deployment-activate cc200f43
Activating deployment 'cc200f43' on application prodapp ...
Stopping PHP 5.4 cartridge (Apache+mod_php)
Starting PHP 5.4 cartridge (Apache+mod_php)
Application directory "/" selected as DocumentRoot
Success
```

--
## Migrating an Application to another Gear

#####Create a snapshot of an existing application

    $ rhc snapshot save App_Name

#####Verify that the App_Name.tar.gz file has been created in the working directory

    $ rhc app delete App_Name

#####Create a new application using the same cartridges, but with the correct gear size

    $ rhc app create App_Name Cart_Name -g gear_size

#####Finally, restore the previously saved application snapshot to the newly created application

    $ rhc snapshot restore App_Name -f App_Name.tar.gz

