ooo-build provides Ant xml to bootstrap a vanilla Ant installation with Maven support via [Maven's Ant tasks](http://maven.apache.org/ant-tasks/). The bootstrap code fetches the bulk of ooo-build and the Maven tasks dynamically, so it allows Ant projects to use Maven and any other Maven-based tasks without requiring anything besides Ant from its users.

Getting Started
================
To use ooo-build, copy [bootstrap.xml](https://raw.github.com/threerings/ooo-build/master/etc/bootstrap.xml) into your project alongside the build.xml for Ant. Then import the bootstrap code at the top of your build.xml with the following:

    <property name="ooo-build.vers" value="2.7"/>
    <ant antfile="bootstrap.xml"/>
    <import file="${user.home}/.m2/ooo-build/${ooo-build.vers}/ooo-build.xml"/>

Once it's imported, add a dependency on `-init-ooo` to your lowest common denominator targets, and it's ready to use. Running that target makes the Maven Ant tasks available, loads [ant-contrib](http://ant-contrib.sourceforge.net/ ), and sets up ooo-build's Maven macros.

Upgrading
=========
If a new version of ooo-build is released, you can change the value of `ooo-build.vers` -- the first property -- to start using it. `bootstrap.xml` pulls its version from `ooo-build.vers`, so it can remain the same through versions.

Macros
======
mavendep
--------
mavendep fetches Maven dependencies for a pom file or inline dependency set, caches the results, and exposes filesets and path properties of the results.

`<mavendep pom="pom.xml"/>` fetches the dependencies of pom.xml and makes them available as `pom.xml.path` for inclusion as `javac`'s classpathref or for other users of path like structures.


    <mavendep id="tools">
      <artifact:dependency groupId="com.threerings" artifactId="narya-tools" version="1.7"/>
    </mavendep>

fetches `com.threerings.narya-tools-1.7` and makes them available as `tools.path` in the same way as `pom.xml.path` above.

Either the pom or id attribute must be given to the macro. All elements inside the `mavendep` element are passed directly into [Maven Ant Tasks' dependencies element](http://maven.apache.org/ant-tasks/reference.html#dependencies)

**All Attributes**

* _pom_ - The pom file to load dependencies from.
* _id_ - The id to use for generated paths and files. Defaults to _pom_ if not given.
* _fileset_ - The name of the fileset to create containing the paths to the fetched dependencies. Defaults to _id_.fileset.
* _path_ - The name of the path to create containing the fetched dependencies. Defaults to _id_.path.
* _cacheDependencyRefs_ - If the dependencies should be cached. Defaults to true. *Added in 2.3*
* _dependencyFile_ - The cache file to generate. Defaults to ${deploy.dir}/_id_.dependencies.
* _pathProp_ - A property to fill in with the paths to all the dependencies. Not filled if no value is given.
* _pathSep_ - The separator to use between paths in _pathProp_. Defaults to `,`.
* _destdir_ - A directory to copy the fetched dependencies to. The dependencies will just be the `artifactId` and `packaging` in the directory. The `groupId`, `qualifier` and `version` are all stripped off. Defaults to not copying.
* _scope_ - The scope of dependencies to fetch. Defaults to `compile`.

cleanmavendepcache
------------------
Removes the cache generated by `mavendep`. It takes a `pom` or `id` attribute like `mavendep` and deletes the file that would be generated for that attribute by `mavendep`

**All Attributes**

* _pom_ - The pom file used to load dependencies.
* _id_ - The id to use for generated files. Defaults to _pom_ if not given.
* _dependencyFile_ - The cache file to delete. Defaults to ${deploy.dir}/_id_.dependencies.

mavendeploy
-----------
Deploys a file to a Maven repository. If the `maven.deploy.repo` property is set, the file and pom are deployed to that repository. If it isn't set, the file and pom are installed in the user's local repository.

**All Attributes**

* _pom_ - The pom file to be deployed.
* _file_ - The packaged artifact to be deployed.
* _srcdir_ - If given, the directory of source to be zipped and deployed with the artifact. No source is deployed if the `srcdir` isn't set.

maventaskdef
------------
Fetches a Maven artifact and load tasks from it e.g. `<maventaskdef groupId="com.threerings.ant" artifactId="javanailgun" version="1.0"/>` loads the [javanailgun task](https://github.com/threerings/ant-javanailgun). The macro checks if the task has already been loaded before running, so it's safe to specify the same task in multiple places.

**All Attributes**

* _groupId_ - The group for the jar containing the tasks.
* _artifactId_ - The artifact for the jar containing the tasks.
* _version_ - The version of the jar containing the tasks.
* _resource_ - The resource to be passed to the [taskdef](http://ant.apache.org/manual/Tasks/typedef.html) task. By default, this is `antlib.xml` in the path to the `groupId` and `artifactId` eg `com/threerings/ant/javanailgun/antlib.xml` for the initial example.
* _cacheDependencyRefs_ - If the dependencies should be cached. Defaults to true. *Added in 2.3*
* _id_ - The base name of the created classpath and loader. Defaults to
  `groupId`.`artifactId`-`version`-taskdef. The classpath is accessible as `id`.path and the loader
  is accessible as `id`.loader for use in classpathref and loaderref in additional
  [typedefs](http://ant.apache.org/manual/Tasks/typedef.html) using this task.

