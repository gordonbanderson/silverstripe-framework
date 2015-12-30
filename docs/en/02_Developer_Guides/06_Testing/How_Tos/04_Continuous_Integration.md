title: How to use continuous integration services

# How to Use Continuous Integration Services

## Introduction

If a module is open source and hosted on Github, one can take advantage of a
number of free services to improve and monitor the quality of the code.

* Travis - Build and test a module with various combinations of PHP, 
SilverStripe, database servers and more.
* Scrutinzer - Automatic code analysis e.g. fix spacing, detect variables not
used, detect methods with high complexity.
* CodeCov - View a code coverage report for every build processed by Travis.  
Also view code coverage over time.

## Accounts

The following descriptions assume that you have already logged into Github in
another tab on the browser being used.

### Scrutinizer
* Log in to the same GitHub account that the module source is hosted in.
* Go to https://scrutinizer-ci.com/ in the same web browser
* Click 'Login' in the top right hand corner.
* Choose 'Login via GitHub'
* Follow the on screen prompts to authorise your Scrutinizer account.
* After logging in, click 'Add Repository' on the right hand side of the screen.
* Type the name of the relevant repository, namely username/github_project_name.
* Select PHP for the default config.
* Do not tick the box to run tests, as this is not a free service.
* Click on settings.
* In the section marked 'Tracked Branches' select the branches to check with
Scrutinizer.  Each time code is pushed to GitHub on the tracked branches, the
codebase will be analyzed by Scrutinizer.

### Travis
* Go to https://travis-ci.org/ in a browser.
* Click 'Sign in with Github' and follow the on screen instructions.
* Click on the '+' icon on the left hand side next to 'My Repositories'
* A list of repositories associated with the logged in GitHub account will be
shown.  Toggle the switch on the module being tested.
* Create a .travis.yml file as above, commit and push it to the GitHub
repository as normal.  This will instigate a build on Travis.
* Click on the TravisCI logo in the upper left corner to go to the homag page.
There the progress of current builds can be seen live.
* To select what criteria triggers a build, select a previous build from the
left hand side and then click 'Settings' on the right hand side.  here one can
choose whether or not pushes and pull requests trigger a new build.

### CodeCov
* Go to https://codecov.io/
* Choose 'GitHub' from the modal box, and follow the on screen instructions.
* Change code in the module being tested and push to GitHub.  This will trigger
a build on Travis.
* Provided the build passes, code coverage will appear on your home page.  Note
that if not the main branch, click on the module name and then the branch being
tested.

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

    :::yml
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

	:::php
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

## Executing Tests
With the above configuration in place and Travis configured to build via a
push request, then simply pushing code to GitHub will trigger a new build,
create an analysis report on Scrutinizer and upload code coverage to CodeCov.

### Example Reports
* [Successful Build of the Mappable Module]
(https://travis-ci.org/gordonbanderson/Mappable/jobs/99297103)
* [Scrutinizer Report](https://scrutinizer-ci.com/g/gordonbanderson/Mappable/?branch=3.1-WIP)
* [CodeCov code coverage](https://codecov.io/github/gordonbanderson/Mappable?branch=3.1-WIP)

