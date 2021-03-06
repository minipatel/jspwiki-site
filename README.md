Apache JSPWiki website
======================

This repository contains the source for the website of Apache JSPWiki, so it 
can be deployed via [gitpubsub](https://blogs.apache.org/infra/entry/git_based_websites_available) 
(see https://issues.apache.org/jira/browse/INFRA-13716)

The website itself is completely static, being automatically generated by [JBake](https://jbake.org/) 
prior to deployment. The content of the website is written in a mixture of Freemarker templates and 
Markdown. Templated content is interpreted only at build time, with the final result being 
completely static.

To facilitate ease of development and testing, this repository also contains an `mvn-ci.sh` build 
script, the usage of which is documented below.

Table of contents
-----------------

* [Repository structure](#repository-structure)
* [Build prerequisites](#build-prerequisites)
* [Testing changes locally](#testing-changes-locally)
* [Publishing changes](#publishing-changes)

Repository structure
--------------------

In addition to the `LICENSE` and `NOTICE` files required of any proper Apache-licensed project, 
the repository contains the following critical files:

| Filename            | Description
| -------------       | -----------
| `./pom.xml`         | to orchestrate the website build
| `./Jenkinsfile`     | file meant to be run by a Jenkins CI server, to build and deploy the website.
| `./mvn-ci.sh`       | script to build and deploy the website using Maven.
| `./src/main/jbake/` | standard [jbake maven plugin structure](https://github.com/ingenieux/jbake-maven-plugin), contains three subdirectories:
|  -> `assets/`       | The assets directory is where you should place your static files such as images, CSS files and JavaScript files etc. These files are copied over to the baked output as is. You can create any directory structure you like in the assets directory and this structure will be maintained when copied.
|  -> `content/`      | holds content files, with the extension of these files determining what type of content it contains (i.e.: `.md` for [Markdown](http://daringfireball.net/projects/markdown/syntax), `.html` for raw HTML, etc.)
|  -> `templates/`    | holds templates files, with the extension of these files determining the templates engine (i.e.: `.ftl` for [Freemarker](https://freemarker.apache.org/), `.thyme` for [Thymeleaf](http://www.thymeleaf.org/), etc.)
| `./content/`        | the actual content served for jspwiki.apache.org, see [publishing changes](#publishing-changes) below

Build prerequisites
-------------------

The CI build (`Jenkinsfile`) needs Java 8 and a Maven installation. This 
script is run at [ASF's Jenkins instance](https://builds.apache.org/job/jspwiki-site/). 
This script takes the changes from the "jbake" branch, generates the site, pulls the 
javadocs, generates the binary compatibility reports and puts them on the "asf-site" 
branch, where they get published, usually a few second later. See "publishing changes"
section below for details.

The Maven build (`mvn-ci.sh`) needs at least Maven 3 and Java 8, thus these must 
be installed first. It's essentially the same build as the CI build, but meant to be 
run locally.

Testing changes locally
-----------------------

Just run `mvn clean jbake:inline` and the site will be accesible through http://localhost:8080. 
Changes will be reloaded on the fly. When done testing your local changes, press `Enter` to stop 
the web server and return to the shell.

In order to be able to navigate locally, you'll need to edit the `site.host` property at 
`./src/main/jbake/jbake.properties`. Do **NOT** commit that change.

Publishing changes
------------------

Changes to the website are published using Apache's 
[gitpubsub](https://blogs.apache.org/infra/entry/git_based_websites_available) which relies on a 
special branch called "asf-site" containing *all website content*.

In the Apache JSPWiki website repository, the "asf-site" branch is an "orphan" branch. Updating 
the website thus involves:

1. Making and testing your changes locally
2. Replacing the entire contents within "asf-site" with the newly-generated site from the 
   "jbake" branch.

Second step is automated through `Jenkinsfile` and `mvn-ci.sh` files. 

Keep in mind that the new content must be *staged* for and committed in this branch. Once you have 
verified that the staged content is as expected, commit your changes (along with a useful commit 
message describing the changes at a high level) using `git commit` and publish the update using 
`git push origin`.

If you wish to unstage your changes, use `git reset --hard HEAD` to return to the original state 
of "jbake", wiping out any local modifications. You can then return to whichever branch you 
were working on with `git checkout`.
