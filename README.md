# tips-and-tricks

## bash, ubuntu

* How to time a command with information on memory usage:
```
/usr/bin/time --verbose ./target/scala-2.13/scala-native-out
```

## sbt

* How to add an environment variables to a scala-native build:
```
import scala.scalanative.build._
ThisBuild / envVars := Map(  
  "GC_NONE_PREALLOC_SIZE" -> sys.env.getOrElse("GC_NONE_PREALLOC_SIZE", "4000M"))
```

* How to add your local maven repo with libs you have `publishLocal` to another build:
```
//https://www.scala-sbt.org/release/docs/Library-Management.html
resolvers += "Local Maven Repository" at "file://"+Path.userHome.absolutePath+"/.m2/repository"
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
