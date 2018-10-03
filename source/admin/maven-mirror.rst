Maven mirroring
===============

Mirroring a maven repository requires downloading all of the artifacts available in the source repository
(not only .pom, .jar files but also jars with dependencies, jars containing sources and javadocs) and then
pushing them all using maven to the target repository.

Helper scripts for mirroring a maven repository available through a RESTful interface can be found
at https://github.com/tungsten-infra/ci-utils/tree/master/tungsten_ci_utils/mirror_maven_repo.

Each command block presented below is executed inside the script directory.

Configure maven
---------------

Configure your maven with access credentials to the target repository:

::

  mkdir $HOME/.m2
  cp settings.xml $HOME/.m2

Edit the $HOME/.m2/settings.xml file providing valid access credentials to the target maven repository.

Artifacts listing
-----------------

The scrapy-maven.py script will be used to crawl the maven repository and generate a list of .jar and .pom files to
download. First install the needed dependencies:

::

  pip install -r requirements.txt

Then update the `start_urls` value with the URL you want to start scraping the repository at in the scrapy-maven.py script.
The script only extracts all links from a page and either follows them or prints them out if they are links to a file with
any extension. Run:

::

  scrapy runspider scrapy-maven.py > links

Artifacts download
------------------

To download all artifacts using the list run:

::

  mkdir artifacts
  for link in `cat links`; do wget -P artifacts $link; done
  rm links

Artifacts grouping
------------------

The following are types of maven artifacts:

* pom files
* jar files
* jars files with dependencies included
* jar source files
* jar javadoc files

Depending on which of them available for a given artifact, the deployment command is different to have them placed
in a target maven repository.

First, segregate the downloaded artifacts. Run the segregate.sh script to segregate the artifacts into groups.

  ::

    bash segregate.sh

Artifact deployment
-------------------

Run the deploy.sh script

  ::

    bash deploy.sh
