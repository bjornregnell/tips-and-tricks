# tips-and-tricks

## git

* What to do when someone says: "Can you rebase this to latest master and force push?" e.g. like [here](https://github.com/lampepfl/dotty/pull/11878#pullrequestreview-620149574)?
  * The below commands assumes that you have forked a repo on GitHub/GitLab and cloned it locally and that the main branch is called `master` and that you have added upstream by `git remote add upstream https://github.com/SOMEORG/SOMELIB.git`
  * More details here [by timonweb.com](https://timonweb.com/misc/how-to-update-a-forked-repo-from-an-upstream-with-git-rebase-or-merge/)
```
git remote -v                  # check that you already have done git remote add upstream  https://github.com/...
git checkout master            # make sure you are on the right branch
git fetch upstream master      # download the master branch from upstream
git rebase upstream/master     # overwrite your master with upstream's master
git push origin master --force # force push your changes to master
```

## bash, ubuntu

* How to time a command with information on memory usage:
```
/usr/bin/time --verbose ./target/scala-2.13/scala-native-out
```

* How to fix errors such as "User limit of inotify instances reached or too many open files"
  * Check your inotify settings: `sysctl fs.inotify`
  * Update your settings by adding these lines at end of `sudo nano /etc/sysctl.conf`
  ```
  #######  MY SETTINGS FOR inotify, increased to prevent errors
  fs.inotify.max_user_watches=204800
  fs.inotify.max_user_instances=256
  fs.inotify.max_queued_events = 204800
  ```
  * Hot reload of changes (or wait until restart): `sudo sysctl -p`
  * Defaults on Ubuntu 18.04 are: 
  ```
  #fs.inotify.max_user_watches=16348
  #fs.inotify.max_user_instances=128
  #fs.inotify.max_queued_events = 16384
  ```

## sbt

* How to add initial commands to console REPL in sbt:
  ```
  Compile / console / initialCommands  := """
    import mypackage.{given, *}
    import scala.language.{postfixOps, implicitConversions}
    println("Hello")
  """
  ```
* Compiler options for Scala 3:
  ```
   scalacOptions ++= Seq(
    "-encoding", "utf8", 
    "-source", "future",
    "-Xfatal-warnings",  
    "-deprecation",
    "-unchecked",
    "-Ysafe-init",
    //"-Yexplicit-nulls",
    //"-language:postfixOps"
  )
  ```
* How to add an environment variables to a scala-native build: (write this after `import scala.scalanative.build._`)
```
ThisBuild / envVars := Map(  
  "GC_NONE_PREALLOC_SIZE" -> sys.env.getOrElse("GC_NONE_PREALLOC_SIZE", "4000M"))
```

* How to add your local maven repo with libs you have `publishLocal` to another build:
```
//https://www.scala-sbt.org/release/docs/Library-Management.html
resolvers += "Local Maven Repository" at "file://"+Path.userHome.absolutePath+"/.m2/repository"
```
### Minimal scala-native build
In `build.sbt`:
```
scalaVersion := "2.13.4"

// Set to false or remove if you want to show stubs as linking errors
nativeLinkStubs := true

enablePlugins(ScalaNativePlugin)

import scala.scalanative.build._

nativeConfig ~= { 
  _.withLTO(LTO.thin)
    .withMode(Mode.releaseFast) // change to releaseFull for optimized binary
    .withGC(GC.commix) // change to GC.none to get dummy GC
}
```
In `project/plugins.sbt`:
```
addSbtPlugin("org.scala-native" % "sbt-scala-native" % "0.4.0")
```

## Create flame graphs (of a Scala Native app)

```
# assumes (1) you have perf installed on a linux system
# sudo apt update && sudo apt install linux-tools-generic

# assumes (2) you have cloned https://github.com/brendangregg/FlameGraph
# into this folder ~/git/hub/FlameGraph/
# the svg flame graph is created in kernel.svg 

# assumes (3) that you have done this to get a binary to run
# sbt run

sudo perf record -F 1000 -a -g ./target/scala-2.13/scala-native-out
sudo perf script > out.perf
~/git/hub/FlameGraph/stackcollapse-perf.pl out.perf > out.folded
~/git/hub/FlameGraph/flamegraph.pl out.folded > kernel.svg

firefox kernel.svg

# The perf option -F 1000 means that the sampling frequency is 1000Hz, 
#  change this to get lagom accuracy, start with -F 99 and increase as needed to see what is interesting
```
