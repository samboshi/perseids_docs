Estimated install time: 4 hours

## Base Environment
* Ubuntu 14.04

## Security Group
Make sure your instance's security-group allows TCP traffic over ports 3000 and 8080

* sosol-rails

# Install system dependencies with apt-get
	sudo apt-get update
	sudo apt-get install build-essential
	sudo apt-get install unzip
	sudo apt-get install zlib1g-dev
	sudo apt-get install git
	sudo apt-get install subversion
	sudo apt-get install openjdk-7-jre
	sudo apt-get install curl
	supo apt-get install wget
	sudo apt-get install tomcat6
	sudo apt-get install tomcat6-admin
	sudo apt-get install postgresql
	sudo apt-get install apache2
	sudo a2enmod proxy
	sudo a2enmod proxy_http
	sudo a2enmod proxy_https
	sudo a2enmod headers
	
	
	

# Install Ruby and Dependencies
## Install rvm
	gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
	\curl -sSL https://get.rvm.io | bash -s stable
	source ~/.rvm/scripts/rvm

## Install jruby-1.7.19
	rvm install jruby-1.7.19
	rvm --default use jruby-1.7.19
	
## Install Bundler
        gem install bundler
        
## Install rails
	gem install rails -v 3.2.3

# Create sosol user and make it owner of /usr/local
        addgrp sosol
        useradd sosol -m -s /bin/bash -g sosol
        chown -R sosol:sosol /usr/local
        mkdir /usr/local/sosol
        sudo chown -R sosol:sosol /usr/local/sosol
        su - sosol

Everything from here on out should be as sosol user

# Clone sosol project and switch to the perseids-production
	git clone https://github.com/sosol/sosol /usr/local/sosol
	cd /usr/local/sosol
	git checkout -b perseids-production origin/perseids-production

# Install eXist db 
Download

	cd ~
	wget http://downloads.sourceforge.net/project/exist/Stable/1.4.1/eXist-setup-1.4.1-rev15155.jar
	wget http://downloads.sourceforge.net/project/exist/Stable/2.2/eXist-db-setup-2.2.jar

Install

        mkdir /usr/local/eXist-1.4.1
        mkdir /usr/local/eXist-2.2
	java -jar eXist-setup-1.4.1-rev15155.jar -p /usr/local/eXist-1.4.1
	cd /usr/local/eXist-2.2
	java -jar eXist-db-setup-2.2.jar
	cp /usr/local/eXist-2.2/tools/wrapper/bin/wrapper-linux-x86-65 /usr/local/eXist1.4.1/tools/wrapper/bin/wrapper

Edit /usr/local/eXist-1.4.1/tools/wrapper/conf/wrapper.conf to set jetty.port = 8800	

TODO eXist 2.2 install fails, requires Java 7

## Install CTS API and Tools into eXist

TODO replace all this with Capitains Toolkit setup...

Start the eXist server

## Install the Alpheios CTS API into eXist

Download the code as a restorable zip from https://github.com/alpheios-project/cts-api/archive/master.zip

Restore from this zip using the eXist client Tools/Restore  (or use command line restore option)

Verify that the basics of the CTS API are working via the following URLS:

http://localhost:8080/exist/rest/db/xq/CTS.xq?request=GetCapabilities
(should return inventory file)

http://localhost:8080/exist/rest/db/xq/CTS.xq?request=GetValidReff&urn=urn:cts:greekLit:tlg0012.tlg001.alpheios-text-grc1
(should return list of valid refs for books in the Iliad)

http://localhost:8080/exist/rest/db/xq/CTS.xq?request=GetPassage&urn=urn:cts:greekLit:tlg0012.tlg001.alpheios-text-grc1:1.1
(should return TEI xml for line 1 of the Iliad)

TODO provide sample output for testing against.

## Install the Alignment Editor into eXist
Download the code as a restorable zip from https://github.com/alpheios-project/alignment-editor/archive/master.zip

Edit the following files in the zip to replace instances of localhost with the server name of your deployment environment:

/db/app/align-editsentence-perseids-test.xml  

Restore from this zip using the eXist client Tools/Restore (or use command line restore option  I think usr/local/exist/bin/backup.sh -u admin -r master.zip should work but didn't test)

TODO Verify that the basics of the editor are working via the following URLS:


## Setup Annotation Sources

TODO (this isn't specific to the rails 3 setup so for now just use the annot sources from the live environment see below under application config).  Will need to be addressed for the production setup.

# Setup LLT segtok and tokenize services

The test and dev environments can use the default llt tokenize and segtok services but not if the sosol can only be reached via localhost (the sosol environment passes uris to these services and if sosol is on localhost then the segtok and tokenize services must be as well in order for requests from them to localhost to succeed).

See deploy instructions for these services at :

https://github.com/latin-language-toolkit/llt/blob/master/DEPLOY.md

(Note they depend also on postgresql)

These services are used by the OA annotation (the treebank editor uses them too, but doesn't have the same issue with localhost as that uses a POST rather than feeding a URI to the service)

# Setup cite_mapper service

TODO

# Setup image services

TODO 

# Setup image collection services

TODO

# Install Client-Side Tools

## Annotation Editor

        cd /usr/local
        sudo git clone https://github.com/PerseusDL/annotation-editor

## Arethusa

### Install System Dependencies

        curl -sL https://deb.nodesource.com/setup | sudo bash -
        sudo apt-get install -y nodejs

### Get Code

        cd /usr/local
        sudo git clone https://github.com/latin-language-toolkit/arethusa

### Install

Follow install instructions in Arethusa's README.md

        grunt minify:all
        
## Perseids Client Apps

### System Dependecies
        sudo apt-get install python-software-properites
        sudo apt-get install python-software-properties
        sudo apt-get install apt-file
        sudo apt-file update
        sudo apt-file search add-apt-repository
        sudo add-apt-repository ppa:fkrull/deadsnakes
        sudo apt-get update
        sudo apt-get install python3.4
        sudo apt-get install python3.4-dev
        wget http://peak.telecommunity.com/dist/ez_setup.py; 
        sudo python ez_setup.py
        sudo easy_install pip
        sudo pip install virtualenv
        # apt-get install libapache2-mod-wsgi

### Code
       cd /usr/local
       sudo git clone https://github.com/PerseusDL/perseids-client-apps
       cd perseids-client-apps
       virtualenv -p /usr/bin/python3.4 flask
       flask/bin/pip install -r requirements.txt
       source flask/bin/activate
       cd app
       bower install
       
# Prepare the canonical.git repo aka "read-write" data

Note that the default installation instructions for sosol have this as a rake tasks which will install the default papyri.info db (        bundle exec rake git:db:canonical:clone) -- we will skip that step in favor of these for now -- eventually should have a perseids version of this task.

	cd /usr/local/gitrepos
	git clone --bare  https://github.com/PerseusDL/perseids_canonical_dev.git canonical.git
	
	cd /usr/local/sosol/db/test/git
	git clone --bare https://github.com/PerseusDL/perseids_canonical_test.git canonical.git

## get additional data for tests
        cd /var/www
        mkdir tests
        cd tests
        git clone https://github.com/PerseusDL/treebank_data
        
## create a local clone of the bare repo

It's useful to have a local clone of the canonical repo for file maintenance.

        cd ~
        git clone /usr/local/gitrepos/canonical.git

# Prepare the inventory files
You have to load inventory files and their indices for the Persieds CTS selector to work properly.
The perseids-dev.xml inventory file can be found in the CTS_XML_TextInventory directory of the perseids_canonical_dev repo.  This needs to be uploaded to the eXist db/repository/inventory directory.

The perseids-test.xml inventory file can be found in the CTS_XML_TextInventory directory of the perseids_canonical_test repo.  This needs to be uploaded to the eXist db/repository/inventory directory.


## Setup Apache2 Proxies for Tools

TODO Note:  for production environment replace 3000 with 8080/sosol for tomcat deployment, 8080 with 8800 for eXist, and drop the proxies to /cts and /publications and /dmm_api

Add the following in a conf file in /etc/apache2/conf.d and then restart apache

        <IfModule mod_proxy.c>

            ProxyPass /dmm_api http://localhost:3000/dmm_api
            ProxyPassReverse /dmm_api http://localhost:3000/dmm_api

            ProxyPass /cts http://localhost:3000/cts
            ProxyPassReverse /cts http://localhost:3000/cts
            
            ProxyPass /publications http://localhost:3000/publications
            ProxyPassReverse /publications http://localhost:3000/publications
            
            ProxyPass /exist/rest/db/repository !
            ProxyPass /exist http://localhost:8080/exist
            ProxyPassReverse /exist http://localhost:8080/exist

        </IfModule>



# Change the SoSOL application config
This part is of the config needs to be improved.

Currently the configuration files in the rails-3-perseus-merge branch contain the default settings for the perseids dev environment. These can't be merged back into master because they would tromp the papyri.info settings. This will be addressed in a future build of SoSOL but for now I've listed all the settings that can be set in application.rb (or lower in e.g. environments/development.rb) that are specific to the perseids deployment.

The following should be the same for production and dev environments:

         SITE_NAME = 'Perseids'
         SITE_FULL_NAME = 'Perseids'
         SITE_TAG_LINE = 'powered by Son of Suda Online'
         SITE_WIKI_LINK = '<a href="http://sites.tufts.edu/perseids">Perseids Blog</a>
         SITE_LAYOUT = 'perseus'
         SITE_IDENTIFIERS = 'CitationCTSIdentifier,EpiCTSIdentifier,EpiTransCTSIdentifier,OACIdentifier,CTSInventoryIdentifier,CommentaryCiteIdentifier,TreebankCiteIdentifier,AlignmentCiteIdentifier'
         SITE_CATALOG_SEARCH = "View In Catalog"
         SITE_USER_NAMESPACE = "data.perseus.org" 
         SITE_OAC_NAMESPACE = "http://data.perseus.org/perseids/annotations/"
         SITE_CITE_COLLECTION_NAMESPACE = "http://data.perseus.org/collections"
         SITE_EMAIL_FROM = 'admin@perseids.org'
         REPOSITORY_ROOT = "/usr/local/gitrepos"
         XSUGAR_STANDALONE_URL="http://localhost:9999/"
         XSUGAR_STANDALONE_USE_PROXY="true"

The following are specific to the deployment and depend upon the port(s) at which you are running the eXist server or servers.  

         EXIST_STANDALONE_URL="http://localhost:8080"
        
EXIST_STANDALONE_URL is the url for the eXist instance that is local to the sosol application (and which has the cts inventories loaded that corresponded to the read/write data in the cnaonical git repo)

Other config files:

cts.yml - configures the CTS repository urls

tools.yml - configures other tools and services

agents.yml - configures client agents

	
# setup the RPX api key

for development environment  use environments/development_secret.rb
for production environment use environments/production_secret.rb

must contain:

        Sosol::Application.configure do            
          config.rpx_api_key = '<put secret here - see janrain login details in perseids it inventory doc>'
          config.rpx_realm = 'perseus-sosol'
        end
        
# Update the database.yml

The version of database.yml that is currently in the rails-3-perseus-merge branch works for a dev or test environment.

For a production environment, the database connection url and username and password need to be set to the credentials for the production mysql db. See the Perseis IT Inventory for these details. (They are not for publication here...)

# Update the tools.yml

For dev environment: if not on localhost, edit the tools.yml, replacing instances of localhost with base route to the deployed sosol environment.

# TODO: Configure shibboleth

# Set RAILS_ENV

For a dev environment: Change RAILS_ENV value to development

# TODO: Run the tests


# Run the bundle commands

	bundle install
	bundle exec cap local externals:setup
	bundle exec rake db:create
	bundle exec rake db:migrate --verbose --trace
	bundle exec rake test:perseids
	bundle exec rails server
	
# Get the javascript submodules

        git submodule update --init --recursive
        
## Notes for war file

bouncy-castle-java should be added to the Gemfile but for some reason bundle install isn't installing it to the .rbenv directory so the warble command can't find it.  had to install it by running gem install bouncy-castle-java

contents of lib/rxsugar don't get copied the war -- need to figure out how to get warbler to pick externals up
also note that path to jruby_helper in the rxsugar initializer code is wrong for the war file deployment -- must not start with lib but rxsugar.

need to be sure to update database.yml, production_secret.rb

sass is missing from the war?

# Deploy on AWS

## Create Elastic IP
console.aws.amazon.com
	> Services
		> EC2
	> Running Instances
	> [Instance]
	> Elastic IP -
	> Allocate New Address
	> Associate Address

## Create DNS Hostname
console.aws.amazon.com
	> Services
		> Route 53
	> Domain Name Table
		> [x] [Domain Name]
	> Go to Record Sets
	> Create Record Set

	Create Record Set
		Name: [whatever].perseids.org
		Type A: IPv4 address
		Value: [Elastic IP]

## Add host to janrain whitelist
https://dashboard.janrain.com/


# Start the Server
You have to start the eXist DB and rails

	/usr/local/exist/bin/startup.sh &
	cd /usr/local/sosol;
	bundle exec rails server &

TODO for a production environment we want to run eXist through wrapper and deploy app war file under tomcat



