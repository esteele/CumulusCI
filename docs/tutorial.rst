========
Tutorial
========

Part 1: Installing CumulusCI
============================

Requirements
------------

* You must have Python version 2.7.x installed
* A local git repository containing Salesforce metadata in the `src/` subfolder or fork then clone CumulusCI-Test for demo::

    git clone https://github.com/YOUR_GITHUB_FORK_USER/CumulusCI-Test

* Ensure you have virtualenv installed by either installing a package for your OS or installing with pip::

    pip install virtualenv


Using virtualenv
----------------

Run the following::

    virtualenv ~/cumulusci_venv
    source ~/cumulusci_venv/bin/activate

Once activated, you will see (venv) at the start of your shell prompt to let you know the virtualenv is active.  From this point, any Python packages you install will be installed only into the virtualenv and leave your system's Python alone.

If you want to always have the CumulusCI commands available, you can add the following line to your ~/.bash_profile::

    source ~/cumulusci_venv/bin/activate

More information about using virtualenv is available at: http://docs.python-guide.org/en/latest/dev/virtualenvs/


Installation
------------

With your virtualenv activated::

    pip install cumulusci

This will install the latest version of CumulusCI and all its dependencies into the virtualenv.  You can verify the installation by running:

    $ cumulusci2
    Usage: cumulusci2 [OPTIONS] COMMAND [ARGS]...
    
    Options:
    --help  Show this message and exit.
    
    Commands:
    flow     Commands for finding and running flows for a...
    org      Commands for connecting and interacting with...
    project  Commands for interacting with project...
    shell    Drop into a python shell
    task     Commands for finding and running tasks for a... 
    version  Print the current version of CumulusCI


Project Initialization
----------------------

The `cumulusci2` command is git repository aware.  Changing directories from one local git repository to another will change the project context.  Each project context isolates the following:

* Connected App: The Salesforce Connected App to use for OAuth authentication
* Orgs: Connected Salesforce Orgs are stored in a project specific keychain
* Services: Named service connections such as Github, ApexTestsDB, and mrbelvedere

If you run the `cumulusci2` command from outside a git repository, it will generate an error::

    $ cd ~

    $ cumulusci2
    No repository found in current path.  You must be inside a repository to initialize the project configuration

If you run the `cumulusci2 org list` command from inside a git repository that has not yet been set up for CumulusCI, you will get an error::

    $ cd path/to/your/repo

    $ cumulusci2 project info
    Usage: cumulusci2 project info [OPTIONS]
    Error: No project configuration found.  You can use the "project init" command to initilize the project for use with CumulusCI

As the instructions say, you can use the `cumulusci2 project init` command to initialize the configuration::

    $ cumulusci2 project init
    Name: MyRepoName    
    Package name: My Repo Name
    Package namespace: mynamespace
    Package api version [38.0]: 
    Git prefix feature [feature/]: 
    Git default branch [master]: 
    Git prefix beta [beta/]: 
    Git prefix release [release/]: 
    Test namematch [%_TEST%]: 
    Your project is now initialized for use with CumulusCI
    You can use the project edit command to edit the project's config file
    
    $ cat cumulusci.yml
    project:
        name: MyRepoName
        package:
            name: My Repo Name
            namespace: mynamespace

The newly created `cumulusci.yml` file is the configuration file for wiring up any project specific tasks, flows, and CumulusCI customizations for this project.  You can add and commit it to your git repository::

    $ git add cumulusci.yml
    $ git commit -m "Initialized CumulusCI Configuration"

Part 2: Connecting Salesforce Orgs
==================================

First, you will need to create a Salesforce Connected App with the following steps:

* In a Salesforce Org, go to Setup -> Create -> Apps
* Click "New" under Connected Apps
* Enter a unique value for the Name and API Name field
* Enter a Contact Email
* Check "Enable OAuth Settings"
* Set the Callback URL to http://localhost:8080
* Enable the scopes: full, refresh_token, and web
* Save the Connected App
* Click the Manage button, then click Edit
* Go back to Setup -> Create -> Apps, and click on the app you created
* Record the client_id (Consumer Key) and the client_secret (Consumer Secret)

Configure the Connected App in your project's keychain::

    $ cumulusci2 org configure_connected_app
    client_id:
    client_secret:
    
Configuring the Connected App is a one time operation per project.  Once configured, you can start connecting Salesforce Orgs to your project's keychain::

    $ cumulsci2 org connect dev

    Launching web browser for URL https://login.salesforce.com/services/oauth2/authorize?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:8080/callback&scope=web%20full%20refresh_token&prompt=login
    Spawning HTTP server at http://localhost:8080/callback with timeout of 300 seconds.
    If you are unable to log in to Salesforce you can press ctrl+c to kill the server and return to the command line.

This should open a browser on your computer pointed to the Salesforce login page.  Log in and then grant access to the app.  Note that since the login to capture credentials occurs in your normal browser, you can use browser password managers such as LastPass to log in.  Once access is granted and you see a browser page that says `OK` you can close the browser tab and return to the terminal.  Your org is now connected via OAuth and CumulusCI never needs to know your actual user password.  As an added benefit, OAuth authentication remains valid even after password changes::

    $ cumulusci2 org list

    org        is_default
    ---------  ----------
    dev

    $ cumulusci2 org default dev

    dev is now the default org
     
    $ cumulusci2 org list

    org        is_default
    ---------  ----------
    dev        *

    $ cumulusci2 org default dev --unset

    dev is no longer the default org.  No default org set.

    $ cumulusci2 org list

    org        is_default
    ---------  ----------
    dev

So we can start running some tasks, let's set dev as our default again::

    $ cumulusci2 org default dev

Part 3: Running Tasks
=====================

Once you have some orgs connected, you can start running tasks against them.  First, you'll want to get a list of tasks available to run::

    $ cumulusci2 task list
    
    task                            description
    ------------------------------  -------------------------------------------------------------------------------------------------------
    create_package                  Creates a package in the target org with the default package name for the project
    create_managed_src              Modifies the src directory for managed deployment.  Strips //cumulusci-managed from all Apex code
    create_unmanaged_ee_src         Modifies the src directory for unmanaged deployment to an EE org
    deploy                          Deploys the src directory of the repository to the org
    deploy_pre                      Deploys all metadata bundles under unpackaged/pre/
    deploy_post                     Deploys all metadata bundles under unpackaged/post/
    deploy_post_managed             Deploys all metadata bundles under unpackaged/post/
    get_installed_packages          Retrieves a list of the currently installed managed package namespaces and their versions
    github_clone_tag                Lists open pull requests in project Github repository
    github_master_to_feature        Merges the latest commit on the master branch into all open feature branches
    github_pull_requests            Lists open pull requests in project Github repository
    github_release                  Creates a Github release for a given managed package version number
    github_release_notes            Generates release notes by parsing pull request bodies of merged pull requests between two tags
    install_managed                 Install the latest managed production release
    install_managed_beta            Installs the latest managed beta release
    push_all                        Schedules a push upgrade of a package version to all subscribers
    push_qa                         Schedules a push upgrade of a package version to all orgs listed in push/orgs_qa.txt
    push_sandbox                    Schedules a push upgrade of a package version to all subscribers
    push_trial                      Schedules a push upgrade of a package version to Trialforce Template orgs listed in push/orgs_trial.txt
    retrieve_packaged               Retrieves the packaged metadata from the org
    retrieve_src                    Retrieves the packaged metadata into the src directory
    revert_managed_src              Reverts the changes from create_managed_src
    revert_unmanaged_ee_src         Reverts the changes from create_unmanaged_ee_src
    run_tests                       Runs all apex tests
    run_tests_debug                 Runs all apex tests
    run_tests_managed               Runs all apex tests in the packaging org or a managed package subscriber org
    uninstall_managed               Uninstalls the managed version of the package
    uninstall_packaged              Uninstalls all deleteable metadata in the package in the target org
    uninstall_packaged_incremental  Deletes any metadata from the package in the target org not in the local workspace
    uninstall_src                   Uninstalls all metadata in the local src directory
    uninstall_pre                   Uninstalls the unpackaged/pre bundles
    uninstall_post                  Uninstalls the unpackaged/post bundles
    uninstall_post_managed          Uninstalls the unpackaged/post bundles
    update_admin_profile            Retrieves, edits, and redeploys the Admin.profile with full FLS perms for all objects/fields
    update_dependencies             Installs all dependencies in project__dependencies into the target org
    update_meta_xml                 Updates all -meta.xml files to have the correct API version and extension package versions
    update_package_xml              Updates src/package.xml with metadata in src/
    update_package_xml_managed      Updates src/package.xml with metadata in src/
    upload_beta                     Uploads a beta release of the metadata currently in the packaging org
    upload_production               Uploads a beta release of the metadata currently in the packaging org

You can view the details on an individual task::

    $ cumulusci2 task info update_package_xml

    Description: Updates src/package.xml with metadata in src/
    Class: cumulusci.tasks.metadata.package.UpdatePackageXml
    
    Default Option Values
        path: src
    
    Option   Required  Description
    -------  --------  ----------------------------------------------------------------------------------------------
    path     *         The path to a folder of metadata to build the package.xml from
    delete             If True, generate a package.xml for use as a destructiveChanges.xml file for deleting metadata
    managed            If True, generate a package.xml for deployment to the managed package packaging org
    output             The output file, defaults to <path>/package.xml

You can run a task::

    $ cumulusci2 task run update_package_xml

    INFO:UpdatePackageXml:Generating src/package.xml from metadata in src

And you can run a task passing any of the options via the command line::

    $ cumulusci2 task run update_package_xml -o managed True -o output managed_package.xml

    INFO:UpdatePackageXml:Generating managed_package.xml from metadata in src
 
Running Tasks Against a Salesforce Org
--------------------------------------
 
The update_package_xml task works only on local files and does not require a connection to a Salesforce org.  The deploy task uses the Metadata API to deploy the src directory to the target org and thus requires a Salesforce org.  Since we already made dev our default org, we can still just run the task against our dev org by calling it without any options::

    $ cumulusci2 task info deploy

    Description: Deploys the src directory of the repository to the org
    Class: cumulusci.tasks.salesforce.Deploy
    
    Default Option Values
        path: src
    
    Option  Required  Description
    ------  --------  ----------------------------------------------
    path    *         The path to the metadata source to be deployed

    $ cumulusci2 task run deploy

    INFO:Deploy:Pending
    INFO:Deploy:[InProgress]: Processing Type: ApexComponent
    INFO:Deploy:[InProgress]: Processing Type: CustomObject
    INFO:Deploy:[InProgress]: Processing Type: CustomObject
    INFO:Deploy:[InProgress]: Processing Type: Layout
    INFO:Deploy:[InProgress]: Processing Type: ApexClass
    INFO:Deploy:[InProgress]: Processing Type: ApexTrigger
    INFO:Deploy:[InProgress]: Processing Type: ApexTrigger
    INFO:Deploy:[Done]
    INFO:Deploy:[Success]: Succeeded

Now that the metadata is deployed, you can run the tests::
    
    $ cumulusci2 task info run_tests
    Description: Runs all apex tests
    Class: cumulusci.tasks.salesforce.RunApexTests
    
    Option             Required  Description
    -----------------  --------  ------------------------------------------------------------------------------------------------------
    test_name_exclude            Query to find Apex test classes to exclude ("%" is wildcard).  Defaults to project__test__name_exclude
    managed                      If True, search for tests in the namespace only.  Defaults to False
    test_name_match    *         Query to find Apex test classes to run ("%" is wildcard).  Defaults to project__test__name_match
    poll_interval                Seconds to wait between polling for Apex test results.  Defaults to 3
    namespace                    Salesforce project namespace.  Defaults to project__package__namespace
    junit_output                 File name for JUnit output.  Defaults to test_results.xml
    junit_namespace              Prepend a namespace string to class names in junit output. Defaults to none

    $ cumulusci2 task run run_tests
    
Part 4: Flows
-------------

Flows are simply named sequences of tasks.  Flows are designed to be run against a single target org.  CumulusCI comes with a number of best practice flows out of the box.::

    $ cumulusci2 flow list

    flow          description
    ------------  --------------------------------------------------------------------------------
    dev_org       Deploys the unmanaged package metadata and all dependencies to the target org
    ci_feature    Deploys the unmanaged package metadata and all dependencies to the target org
    ci_master     Deploys the managed package metadata and all dependencies to the packaging org
    ci_beta       Installs a beta version and runs tests
    ci_release    Installs a production release version and runs tests
    release_beta  Uploads and releases a beta version of the metadata currently in packaging
    unmanaged_ee  Deploys the unmanaged package metadata and all dependencies to the target EE org

To set up our newly connected dev org, run the dev_org flow::

    $ cumulusci2 flow run dev_org

    INFO:BaseFlow:---------------------------------------
    INFO:BaseFlow:Initializing flow class BaseFlow:
    INFO:BaseFlow:---------------------------------------
    INFO:BaseFlow:Flow Description: Deploys the unmanaged package metadata and all dependencies to the target org
    INFO:BaseFlow:Tasks:
    INFO:BaseFlow:  create_package: Creates a package in the target org with the default package name for the project
    INFO:BaseFlow:  update_dependencies: Installs all dependencies in project__dependencies into the target org
    INFO:BaseFlow:  deploy_pre: Deploys all metadata bundles under unpackaged/pre/
    INFO:BaseFlow:  deploy: Deploys the src directory of the repository to the org
    INFO:BaseFlow:  uninstall_packaged_incremental: Deletes any metadata from the package in the target org not in the local workspace
    INFO:BaseFlow:  deploy_post: Deploys all metadata bundles under unpackaged/post/
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: create_package
    INFO:BaseFlow:Options:
    INFO:BaseFlow:  api_version: 33.0
    INFO:BaseFlow:  package: CumulusCI-Test
    INFO:CreatePackage:Pending
    INFO:CreatePackage:[Done]
    INFO:CreatePackage:[Success]: Succeeded
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: update_dependencies
    INFO:BaseFlow:Options:
    INFO:UpdateDependencies:Project has no dependencies, doing nothing
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: deploy_pre
    INFO:BaseFlow:Options:
    INFO:BaseFlow:  path: unpackaged/pre
    INFO:DeployBundles:Deploying all metadata bundles in path /Users/jlantz/dev/CumulusCI-Test/unpackaged/pre
    INFO:DeployBundles:Deploying bundle: unpackaged/pre/account_record_types
    INFO:DeployBundles:Pending
    INFO:DeployBundles:[InProgress]: Processing Type: CustomObject
    INFO:DeployBundles:[InProgress]: Processing Type: CustomObject
    INFO:DeployBundles:[Done]
    INFO:DeployBundles:[Success]: Succeeded
    INFO:DeployBundles:Deploying bundle: unpackaged/pre/opportunity_record_types
    INFO:DeployBundles:Pending
    INFO:DeployBundles:[Done]
    INFO:DeployBundles:[Success]: Succeeded
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: deploy
    INFO:BaseFlow:Options:
    INFO:BaseFlow:  path: src
    INFO:Deploy:Pending
    INFO:Deploy:[InProgress]: Processing Type: ApexPage
    INFO:Deploy:[InProgress]: Processing Type: CustomObject
    INFO:Deploy:[InProgress]: Processing Type: CustomObject
    INFO:Deploy:[InProgress]: Processing Type: QuickAction
    INFO:Deploy:[InProgress]: Processing Type: ApexClass
    INFO:Deploy:[Done]
    INFO:Deploy:[Success]: Succeeded
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: uninstall_packaged_incremental
    INFO:BaseFlow:Options:
    INFO:BaseFlow:  path: src
    INFO:BaseFlow:  package: CumulusCI-Test
    INFO:UninstallPackagedIncremental:Retrieving metadata in package CumulusCI-Test from target org
    INFO:UninstallPackagedIncremental:Pending
    INFO:UninstallPackagedIncremental:[Done]
    INFO:UninstallPackagedIncremental:Deleting metadata in package CumulusCI-Test from target org
    INFO:UninstallPackagedIncremental:Pending
    INFO:UninstallPackagedIncremental:[Done]
    INFO:UninstallPackagedIncremental:[Success]: Succeeded
    INFO:BaseFlow:
    INFO:BaseFlow:Running task: deploy_post
    INFO:BaseFlow:Options:
    INFO:BaseFlow:  namespace_token: %%%NAMESPACE%%%
    INFO:BaseFlow:  path: unpackaged/post
    INFO:BaseFlow:  namespace: ccitest
    INFO:BaseFlow:  managed: False
    INFO:BaseFlow:  filename_token: ___NAMESPACE___
    INFO:DeployNamespacedBundles:Deploying all metadata bundles in path /Users/jlantz/dev/CumulusCI-Test/unpackaged/post
    INFO:DeployNamespacedBundles:Deploying bundle: unpackaged/post/salesforce1
    INFO:DeployNamespacedBundles:Pending
    INFO:DeployNamespacedBundles:[Done]
    INFO:DeployNamespacedBundles:[Success]: Succeeded

Part 5: Digging Deeper
======================
   
Custom Tasks
------------

Create a local python tasks module::

    $ mkdir tasks
    $ touch tasks/__init__.py

Create the file `tasks/salesforce.py` with the following content::

    from cumulusci.tasks.salesforce import BaseSalesforceApiTask
    from cumulusci.tasks.salesforce import BaseSalesforceToolingApiTask
    
    class ListContacts(BaseSalesforceApiTask):
    
        def _run_task(self):
            res = self.sf.query('Select Id, FirstName, LastName from Contact LIMIT 10')
            for contact in res['records']:
                self.logger.info('{Id}: {FirstName} {LastName}'.format(**contact))
    
    class ListApexClasses(BaseSalesforceToolingApiTask):
    
        def _run_task(self):
            res = self.tooling.query('Select Id, Name, NamespacePrefix from ApexClass LIMIT 10')
            for apexclass in res['records']:
                self.logger.info('{Id}: [{NamespacePrefix}] {Name}'.format(**apexclass)) 

Finally, wire in your new tasks by editing the cumulusci.yml file in your repo and adding the following lines::
    
    tasks:
        list_contacts:
            description: Prints out 10 Contacts from the target org using the Enterprise API
            class_path: tasks.salesforce.ListContacts
        list_apex_classes:
            description: Prints out 10 ApexClasses from the target org using the Tooling API
            class_path: tasks.salesforce.ListApexClasses

Now your new tasks are available in the task list::
    
    $ cumulusci2 task list
    task                            description
    ------------------------------  ---------------------------------------------------------------------------------
    create_package                  Creates a package in the target org with the default package name for the project
    ...
    list_contacts                   Prints out 10 Contacts from the target org using the Enterprise API
    list_apex_classes               Prints out 10 ApexClasses from the target org using the Tooling API

Run the tasks::
    
    $ cumulusci2 task run list_contacts

    INFO:ListContacts:003j00000045WfwAAE: Siddartha Nedaerk
    INFO:ListContacts:003j00000045WfxAAE: Jake Llorrac
    INFO:ListContacts:003j00000045WfeAAE: Rose Gonzalez
    INFO:ListContacts:003j00000045WffAAE: Sean Forbes
    INFO:ListContacts:003j00000045WfgAAE: Jack Rogers
    INFO:ListContacts:003j00000045WfhAAE: Pat Stumuller
    INFO:ListContacts:003j00000045WfiAAE: Andy Young
    INFO:ListContacts:003j00000045WfjAAE: Tim Barr
    INFO:ListContacts:003j00000045WfkAAE: John Bond
    INFO:ListContacts:003j00000045WflAAE: Stella Pavlova

    $ cumulusci2 task run list_apex_classes

    INFO:ListApexClasses:01pj000000164zgAAA: [npe01] Tests
    INFO:ListApexClasses:01pj000000164zeAAA: [npe01] IndividualAccounts
    INFO:ListApexClasses:01pj000000164zfAAA: [npe01] NPSPPkgVersionCheck
    INFO:ListApexClasses:01pj000000164zdAAA: [npe01] Constants
    INFO:ListApexClasses:01pj000000164zsAAA: [npe03] RecurringDonations
    INFO:ListApexClasses:01pj000000164ztAAA: [npe03] RecurringDonationsPkgVersionCheck
    INFO:ListApexClasses:01pj000000164zuAAA: [npe03] RecurringDonations_BATCH
    INFO:ListApexClasses:01pj000000164zvAAA: [npe03] RecurringDonations_SCHED
    INFO:ListApexClasses:01pj000000164zwAAA: [npe03] RecurringDonations_TEST
    INFO:ListApexClasses:01pj000000164zxAAA: [npe4] Relationships_INST

Further Exploration
-------------------

These will be filled out in more detail in the future but are a brief overview of commands to explore next::
 
    $ cumulusci2 project connect_github
    $ cumulusci2 project connect_apextestsdb
    $ cumulusci2 project connect_mrbelvedere


Environment Keychain
--------------------

The keychain class can be overridden to change storage implementations.  The default keychain for the cumulusci2 CLI stores AES encrypted files under `~/.cumulusci`.  The EnvironmentProjectKeychain class provides a keychain implementation which receives its credentials from environment variables.  This is useful for using the CLI on CI servers such as Jenkins or CircleCI.::

    $ cumulusci2 org connected_app
    $ cumulusci2 org info feature
    $ cumulusci2 org info packaging
    $ cumulusci2 org info beta
    $ cumulusci2 project show_github 
    $ export CUMULUSCI_KEYCHAIN_CLASS=cumulusci.core.keychain.EnvironmentProjectKeychain
    $ cumulusci2 org list
    $ export CUMULUSCI_CONNECTED_APP="{__COPIED_FROM_ABOVE__}"
    $ export CUMULUSCI_ORG_feature="{__COPIED_FROM_ABOVE__}"
    $ export CUMULUSCI_ORG_packaging="{__COPIED_FROM_ABOVE__}"
    $ export CUMULUSCI_ORG_beta="{__COPIED_FROM_ABOVE__}"
    $ export CUMULUSCI_SERVICE_github="{__COPIED_FROM_ABOVE__}"
    $ cumulusci2 org list
    $ cumulusci2 task run --org feature deploy
