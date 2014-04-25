## Dr Elephant

### Compiling & testing locally

* To be able to build & run the application, download and install [Play framework 2.2.2](http://downloads.typesafe.com/play/2.2.2/play-2.2.2.zip).
* To build and run the application in dev mode, run from command line "play run" in the project directory.
* There is need to investigate the framework to see how one can add parameters to the classpath in dev mode.

### Deployment

* To create a deployment package, use "play dist" to create a zip package, or use "play universal:package-zip-tarball" to create a tarball
* To run the deployed package with Hadoop properly, some changes needs to be added to the startup script located at ./bin/dr-elephant

* in the classpath ("declare -r app\_classpath=...") , add to the end of the string, before the end quotes

            :$HADOOP_HOME/*:$HADOOP_HOME/lib/*:$HADOOP_HOME/conf

* after the next line ("addJava ... ;"), add new line

            addJava "-Djava.library.path=$HADOOP_HOME/lib/native/Linux-amd64-64"

### DB Schema evolutions

When the schema in the model package changes, play will need to be ran to automatically apply the evolution.

* There is a problem with Ebean where it does not support something like @Index to generate indices for columns of interest
* So what we did to work around this is to manually add indices into the sql script.
* To do this, we needed to prevent the automatically generated sql to overwrite our modified sql.
* The evolution sql file must be changed (by moving or removing the header "To stop Ebean DDL generation, remove this comment and start using Evolutions") to make sure it does not automatically generate new sql.
* To re-create the sql file from a new schema in code:
	* Backup the file at ./conf/evolutions/default/1.sql
	* Remove the file
	* Run play in debug mode and browse the page. This causes EBean to generate the new sql file, and automatically apply the evolution.
	* Copy over the indices from the old 1.sql file
	* Remove the header in the sql file so it does not get overwritten
	* Browse the page again to refresh the schema to add the indices.

### Running on the cluster

* SSH into the machine
* sudo as elephant
* go to /export/apps/elephant/
* To start: ./run.sh
* To kill: ./kill.sh
* To deploy new version:
	* scp machine:location-to-drelephant.zip /export/apps/elephant/
	* ./kill.sh
	* unzip dr-elephant-0.1-SNAPSHOT.zip
	* ./run.sh
