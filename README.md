# JSpecify Nullness Checker for the Checker Framework

This project is a
[custom checker plugin](https://checkerframework.org/manual/#creating-a-checker)
for [the Checker Framework](https://checkerframework.org/).

It is a prototype. It is not yet suitable for use on real code. To get it
working properly, we will need to do more work, including proposing new
configuration hooks in the Checker Framework itself. Running it against the
regular version of the Checker Framework will behave incorrectly, as we have
[made related changes in a temporary fork of the Checker Framework](https://github.com/jspecify/checker-framework).
In fact, even *building* it against the regular version of the Checker Framework
does not work after
[8fcc87f5e4e28c2e4511eb8cf26092d935425fea](https://github.com/jspecify/nullness-checker-for-checker-framework/commit/8fcc87f5e4e28c2e4511eb8cf26092d935425fea).

We have made it available so that JSpecify project members can experiment with
it, collaborate on it, and discuss the easiest way to incorporate changes into
the Checker Framework itself. If all goes well, it will never be released for
general use. Instead, existing tools (including the Checker Framework) will be
modified to support JSpecify annotations directly, perhaps based on some of the
code here.

Our main work is happening in repo https://github.com/jspecify/jspecify.

This is not an officially supported product of any of the participant
organizations.

## Usage

Again, this is not ready for general use. But for those in our group who are
looking to try it out:

```
# Build the checker:

$ git clone https://github.com/jspecify/nullness-checker-for-checker-framework

$ cd nullness-checker-for-checker-framework

$ ./gradlew assemble


# Use it:

$ ../checker-framework/checker/bin/javac -processorpath ../jspecify/build/libs/jspecify-0.0.0-SNAPSHOT.jar:build/libs/nullness-checker-for-checker-framework.jar -processor com.google.jspecify.nullness.NullSpecChecker -AcheckImpl -cp ../jspecify/build/libs/jspecify-0.0.0-SNAPSHOT.jar:build/libs/nullness-checker-for-checker-framework.jar:... ...


# For example:

$ cat > SomeTest.java
import org.jspecify.nullness.NullMarked;
import org.jspecify.nullness.Nullable;

@NullMarked
class SomeTest {
  Object passThrough(@Nullable Object o) {
    return o;
  }
}

$ ../checker-framework/checker/bin/javac -processorpath ../jspecify/build/libs/jspecify-0.0.0-SNAPSHOT.jar:build/libs/nullness-checker-for-checker-framework.jar -processor com.google.jspecify.nullness.NullSpecChecker -AcheckImpl -cp ../jspecify/build/libs/jspecify-0.0.0-SNAPSHOT.jar:build/libs/nullness-checker-for-checker-framework.jar SomeTest.java
SomeTest.java:7: error: [nullness] incompatible types in return.
    return o;
           ^
  type of expression: Object?
  method return type: Object
1 error


# Run tests:

$ ./gradlew jspecifySamplesTest


# The tests will fail for two reasons:
#
# 1. Our checker has some known issues.
# 2. The samples use @NullnessUnspecified, an annotation that currently doesn't exist in the mainline of jspecify/jspecify.
#
# To get the tests to pass, you can check and build out a different branch of
# our sample repo, one that has the expected incorrect results encoded into the
# samples *and* has @NullnessUnspecified present:

$ ( cd ../jspecify && git checkout samples-google-prototype && ./gradlew )


# During development, you may wish to pass `-x ensureCheckerFrameworkBuilt` to
# `gradlew` for every build after your first. This will prevent the build
# process from also rebuilding some of our *dependencies* (which is slow).


# Also note: For some of the repos that the build process clones, it creates
# single-branch clones that use an https URL instead of a git URL. You may want
# to convert them into more "normal" GitHub clones:

$ perl -pi -e 's#https://github.com/#git\@github.com:#; s#fetch = \+refs/heads/(main|master):refs/remotes/origin/(main|master)#fetch = +refs/heads/*:refs/remotes/origin/*#;' ../*/.git/config

$ for F in ../*; do ( cd $F && git fetch --unshallow ); done

# (Some repos are already non-shallow, so the command will fail in those
# directories but still succeed in the others.)


# And finally... there is often some version skew between the versions of
# dependencies that our checker currently depends on and the newest versions
# that our build process pulls from upstream. As a result, the build may fail,
# typically because it can't find the stubparser jar. *As of this writing*
# (January 11, 2022), you can use the following command to set up appropriate
# versions *after the build has failed and after you have unshallowed the other
# repositories*.

$ ( cd ../annotation-tools && git checkout dc56ce2a7d8cb85e4af72c78d3bf26d25c2fa7ec ) && ( cd ../stubparser && git checkout f41e795f71198e47925247b2d40061be670252d4 )
```
