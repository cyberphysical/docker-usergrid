docker-usergrid
===================

## Getting started (mac)

Install [boot2docker](https://docs.docker.com/installation/mac/)
Install [fig](http://www.fig.sh/install.html)

I got some errors with fig and needed to use the python installation method.

Run the following if boot2docker if it is not already running
```sh
boot2docker up
```

#Get latest code

```sh
$ git clone https://github.com/cyberphysical/docker-usergrid.git
$ cd mysplitsecnd
$ docker-compose up
```
Cassandra is configured to use a data-only container. Data will persist between docker-compose up's.


##Usergrid GUI

Fig will deploy the GUI on port 3000. You can connect to the dev API as follows:

    http://usergrid.dev:3000?api_url=http://usergrid.dev:8080

You must run the install flow first:

    curl http://192.168.59.103:8080/system/database/setup

The default username and password for development is superuser/password.

Once you're logged in, create a new organization called splitsecnd, and add to that organization a new app called dashboard.

Customization
--------------

You can edit the usergrid-custom.properties file. Private/secret data can be injected with env vars.


Old readme
================

docker-usergrid
===================
Base docker image to run a Apache usergrid_ BaaS.

Used elements of [tutum-docker-tomcat](https://github.com/tutumcloud/tutum-docker-tomcat) as Tomcat build manifest.


Image tags
----------
```
    gaborwnuk/usergrid:latest
```


Usage
-----
First of all - You need Cassandra. If You don't have working instance already, installation using docker is pretty straightforward.

	docker run -p 9042:9042 -d --name cass1 poklet/cassandra

*Installation from repository only*: to create the image `gaborwnuk/usergrid`, execute the following command on the docker-usergrid folder:

    docker build -t gaborwnuk/usergrid .

To run the image and bind to port :

    docker run -d -p 8080:8080 --link cass1:cassandra gaborwnuk/usergrid


The first time that you run your container, a new user `admin` with all privileges 
will be created in Tomcat with a random password. To get the password, check the logs
of the container by running:

    docker logs <CONTAINER_ID>

You will see an output like the following:

	=> Creating and admin user with a random password in Tomcat ...
	=> Done!
	=> Creating Apache usergrid_ properties ...
	=> Done!
	========================================================================
	You can now configure to this Tomcat server using:
	
	    admin:rGlsLYtiPce2

In this case, `rGlsLYtiPce2` is the password allocated to the `admin` user for Tomcat admin console.

**Important**: default username and password for usergrid_ is `superuser` and `VDprvB6bt7ebDW`. This can be changed by customising Dockerfile to suit Your needs and add custom `usergrid-*.properties` file. **Without modifications this should not be used on production.**

**New**: You can now set certain properties using ENV vars. I.e. --env SYSADMIN_PASSWORD=mypassword.

	docker run -d -p 8080:8080 --env SYSADMIN_PASSWORD=mypassword --link cass1:cassandra gaborwnuk/usergrid

For the full list of ENV vars see usergrid-custom.properties. See the [envplate](https://github.com/kreuzwerker/envplate) library for more details on syntax
and variable replacement rules.

You can now check usergrid_ status with:

	curl http://127.0.0.1:8080/status
    
Probably the most important value here is:

	"cassandraAvailable" : true
	
If Your value is `false` - usergrid_ won't work as Cassandra instance is required as storage database. Check [docker-cassandra](https://github.com/nicolasff/docker-cassandra).

To fully configure Your Cassandra/usergrid_ , just:

	curl http://superuser:VDprvB6bt7ebDW@127.0.0.1:8080/system/database/setup
	
After few moments You should receive following response:

	{
	  "action" : "cassandra setup",
	  "status" : "ok",
	  "timestamp" : 1409052433807,
	  "duration" : 7442
	}

Now You're good to go - create Your account:

	curl -X POST  \
	     -d 'organization=gwp&username=admin&name=Admin&email=admin@example.com&password=password' \
	     http://127.0.0.1:8080/management/organizations
	     
Authenticate and get token:

	curl "http://127.0.0.1:8080/management/token?grant_type=password&username=admin&password=password"
	
You should receive authentication feedback with token used in all following REST requests, ie:

	curl -H "Authorization: Bearer YWMtA6C8Ti0bEeSpsz3neosJlAAAAUg2TNNzocqs39RLNbd_V5gr2jaoUacix0E" \
	     -H "Content-Type: application/json" \
	     -X POST -d '{ "name":"myapp" }' \
	     http://127.0.0.1:8080/management/orgs/gwp/apps

And so on.

From now on, if You're interested on how to secure Your usergrid_ instance, refer to [https://github.com/apache/incubator-usergrid/tree/master/stack](https://github.com/apache/incubator-usergrid/tree/master/stack) and [https://usergrid.incubator.apache.org/docs/](https://usergrid.incubator.apache.org/docs/).
	

Customising usergrid_ docker container
-------------------------------------------------

If you want to use a preset password instead of a random generated one, you can
set the environment variable `TOMCAT_PASS` to your specific password when running the container:

    docker run -d -p 8080:8080 -e TOMCAT_PASS="mypass" gaborwnuk/usergrid

Following environmental variables are available:

* `TOMCAT_PASS` (**Optional**, default: random generated) - Tomcat instance password, use if required. You won't probably need this.

## Running on Deis

First login and create an app if not already created. `usergrid-prod` should already exist.

### CDP US
If the app doesnt' exist:

    DEIS_PROFILE=cdp-us deis create usergrid-prod --no-remote
    DEIS_PROFILE=cdp-us deis git:remote -r deis-prod-us -a usergrid-prod

Update configs:

    DEIS_PROFILE=cdp-us deis config:push -p ./.cdp-us.env

Deploy:

    git push deis-prod-us master


### CDP EU
If the app doesnt' exist:

    DEIS_PROFILE=cdp-eu deis create usergrid-prod --no-remote
    DEIS_PROFILE=cdp-eu deis git:remote -r deis-prod-eu -a usergrid-prod

Update configs:

    DEIS_PROFILE=cdp-eu deis config:push -p ./.cdp-eu.env

Deploy:

    git push deis-prod-eu master

