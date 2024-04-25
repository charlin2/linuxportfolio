# Charlie's CSDS 285 Portfolio

### Scripts for Java TAs
This semester, I was a TA for CSDS 132, so I came up with a few simple aliases to help with grading projects.

#### Bulk compile/delete
```Bash
alias compile="ls | grep '.*.java' > sources.txt; javac @sources.txt"
alias clean="rm *.class; rm *.java"
```
TLDR:
- Compile and delete CSDS 132 project files very quickly

The first two 132 projects don't have a lot of files, so I was fine with compiling and deleting these by hand after grading.  The third project, however, had on average 10 files to compile and sift through, so I wrote these aliases to compile all of the project files at once and to delete all of them at once.  This made it very easy to verify that programs could actually run, and it probably **saved me at least 20 min per grading session**.

![Compile/Clean demo](/images/grading.png)

#### Switch Java version
```Bash
alias j21="export JAVA_HOME=`/usr/libexec/java_home -v 21`; java -version"
alias j17="export JAVA_HOME=`/usr/libexec/java_home -v 17`; java -version"
alias j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8`; java -version"
```
TLDR:
- Switch Java versions for different classes

In addition to TA'ing, I was also taking CSDS 293, which required a different version of Java than 132.  This meant that I was constantly having to switch between Java versions for class work and TA work.  These aliases made switching between versions way faster.

![Switch Java version](/images/switch.png)
