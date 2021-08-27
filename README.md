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

* How to **delete a file completely** with e.g. sensitive data from a git repo including all historic commits etc:
  * Make sure your git repo is clean of any changes and pending commits: `git status && git pull && git push` 
  * If you want to keep the latest version then copy the bad file (here called `bad.file` but replace that with the actual name of the file) to a place outside the git repo. `cp bad.file ~/tmp/.` 
  * `git rm bad.file` 
  * `git commit -m "delete bad file"`
  * Download the latest jar of the `bfg` utility here: https://rtyley.github.io/bfg-repo-cleaner/
  * In the top dir of the repo write: `java -jar bfg-1.14.0.jar -D bad.file`
  * `git reflog expire --expire=now --all && git gc --prune=now --aggressive`
  * `git push --force`
  * If you want back your latest version then copy it back into the repo and commit.

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

* How to install nice fonts for coding?
```
sudo apt install fonts-firacode
 
```

* How to manage installed apps on Ubuntu?
  * Installing apps is a mess; there is apt, snap, flatpak, manual deb-files, ... 
  * A typical problem is that you have forgotten how you installed an app.
  * How to check what is installed with each package manager?
    * `flatpak list` 
    * `snap list` 
    * show all apt packages (loooong list, consider appending `| grep -i appname`)
      * `apt list --installed 2>/dev/null`
    * shows which deb-packages are installed *locally* using e.g. `dpkg -i`, `gdebi`, or double-click/software:  
      * `apt list --installed 2>/dev/null | grep -iw local` 
  * How to update apps to latest version?
    * `flatpak update`
    * `sudo snap refresh`
    * `sudo apt full-upgrade`
  * How to remove an app?
    * `flatpak uninstall nameofapp`
    * `sudo snap remove nameofapp`
    * Remove unused flatpak runtimes: `flatpak uninstall --unused`
    * Change number of old snap version to 2 (default 3): `sudo snap set system refresh.retain=2`
    * `dpkg -r nameofapp`
    * See also: https://askubuntu.com/questions/22200/how-to-uninstall-a-deb-package

### grub

Grub is the bootloader that enables dual boot with Ubuntu + Windows.

* How to set grub to remember last boot choice?
  * Read this: https://www.maketecheasier.com/set-grub-remember-last-selection/

```
sudo nano /etc/default/grub
```
Look for `GRUB_DEFAULT="0"` and change it to `GRUB_DEFAULT="saved"` and then add `GRUB_SAVEDEFAULT=true` below the GRUB_DEFAULT line.

Make changes available:
```
sudo update-grub
```

* How to set font in grub boot screen to make it readable on a high dpi screen?
  * Read this: http://blog.wxm.be/2014/08/29/increase-font-in-grub-for-high-dpi.html

```
sudo grub-mkfont --output=/boot/grub/fonts/DejaVuSansMono24.pf2 --size=48 /usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf
sudo nano /etc/default/grub
```
Add this att end of above file:
```
# More readable font on high dpi screen, generated with
# sudo grub-mkfont --output=/boot/grub/fonts/DejaVuSansMono24.pf2 --size=48 /usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf
GRUB_FONT=/boot/grub/fonts/DejaVuSansMono24.pf2
```

Make changes available:
```
sudo update-grub
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
