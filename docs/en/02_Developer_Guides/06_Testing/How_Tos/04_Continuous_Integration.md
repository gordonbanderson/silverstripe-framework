title: How to use continuous integration services

# How to Use Continuous Integration Services

## Introduction

If your module is open source and hosted on Github, you can take advantage of a number of free services to improve the quality of the code.

* Travis - Build and test a module with various combinations of PHP, SilverStripe, database servers and more.
* Scrutinzer - Automatic code analysis e.g. fix spacing, detect variables not used, detect methods with high complexity.
* CodeCov - View a code coverage report for every build processed by Travis.  Also view code coverage over time.

## Accounts
### Scrutinizer

### Travis

### CodeCov

## Configuration

### Scrutinizer

Add a file .scrutinizer.yml in the root directory of a module with the following configuration as a minimum:

    :::yml
    inherit: true

	tools:
	    external_code_coverage:
	      timeout: 600

### Travis

Add a file called .travis.yml in root directory of a module, with the following.  Edit the value *YOUR_MODULE_PATH*, this is the path of the module *after* it has been installed using composer (may differ from the Github repository name).  The environment variable CORE_RELEASE is the version of SilverStripe to test against, here 3.1.

    ::yml
    language: php

	sudo: false

	addons:
	  apt:
	    packages:
	      - tidy

	before_install:
	  - pip install --user codecov

	env:
	  global:
	    - DB=MYSQL CORE_RELEASE=3.1
	    - MODULE_PATH="*YOUR_MODULE_PATH*"

	matrix:
	  allow_failures:
	    - php: hhvm-nightly
	  include:
	    - php: 5.6
	      env: DB=MYSQL PDO=1
	    - php: 5.6
	      env: DB=MYSQL
	    - php: 5.6
	      env: DB=PGSQL
	    - php: 5.5
	      env: DB=MYSQL
	    - php: 5.4
	      env: DB=MYSQL
	    - php: 5.3
	      env: DB=MYSQL
	    - php: hhvm
	      env: DB=MYSQL
	      before_install:


	before_script:
	  - phpenv rehash
	  - composer self-update || true
	  - git clone git://github.com/silverstripe-labs/silverstripe-travis-support.git ~/travis-support
	  - php ~/travis-support/travis_setup.php --source `pwd` --target ~/builds/ss
	  - cd ~/builds/ss

	script:
	  - vendor/bin/phpunit --coverage-clover=coverage.clover -c $MODULE_PATH/phpunit.xml $MODULE_PATH/tests/

	after_success:
	    - cp coverage.clover ~/coverage.xml
	    - mv coverage.clover ~/build/$TRAVIS_REPO_SLUG/
	    - cd ~/build/$TRAVIS_REPO_SLUG

	    # Upload Coverage to Scrutinizer
	    - php ocular.phar code-coverage:upload --format=php-clover coverage.clover

	    # Upload test coverage to codecov
	    - codecov

The above Travis configuration file does the following:
* Checks out the module from Github
* Using the SilverStripe Travis support module create an instance of SilverStripe and install the checked out module using composer.
* Executes the tests whilst simultaneously recording test code coverage.
* Copies the code coverage file to a location inside the original module git checkout.  This is necessary
in order that third party services know where in the Git history to associate the code coverage with.
* Uploads code coverage to Scrutinizer.
* Uploads code coverage to CodeCov

### PHP Unit
In order to restrict code coverage to just the module, add a file phpunit.xml - change the title and the value *YOUR_MODULE_PATH* as above.

	::php
	<phpunit bootstrap="../framework/tests/bootstrap.php" colors="true">

		<testsuite name="*YOUR_MODULE_TITLE%">
			<directory>./tests</directory>
		</testsuite>

		<listeners>
			<listener class="SS_TestListener" file="framework/dev/TestListener.php" />
		</listeners>

		<groups>
			<exclude>
				<group>sanitychecks</group>
			</exclude>
		</groups>

		<filter>
	        <whitelist>
	            <directory>../*YOUR_MODULE_PATH*</directory>
	        </whitelist>
	    </filter>

	</phpunit>
