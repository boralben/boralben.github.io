---
layout: post
title:  "Extracting Info from xcresult files"
date:   2021-05-25 08:20:00 -0600
categories: xcode, junit, xmlstarlet, trainer
---

I needed a script to quickly extract some basic test execution information from an `xcresult` directory. My script needed to get the number tests run, the number that failed, and the names of those failed tests. That's it.

The `xcresult` directory has a ton of helpful info, but it's complicated to access. Xcode offers the `xcresulttool` command line tool to access that info, but I found the ramp up on `xcresulttool` to be excessively steep for my simple case.

I was able to combine a couple of tools to quickly get what I wanted:

1. [Trainer](https://github.com/fastlane-community/trainer), which translates an xcresult into a junit xml file (they did the work of learning xcresulttool for me)
2. [XMLStarlet](http://xmlstar.sourceforge.net/overview.php), which is a command line xml query tool for xml


My simple script:
```
trainer -p /path/to/xcresult -o /path/to/junit
xmlstarlet sel -t -m '//testsuites/testsuite/testcase' -i failure -v @classname -o " - " -v @name -n /path/to/junit.xml
xmlstarlet sel -t -m '//testsuites' -v @tests -o " total tests, " -v @failures -o " failed." /path/to/junit.xml 
```

What's going on here?

1. `trainer` generates a `junit.xml` file that looks something like this:

```
<testuites tests="10" failures="1">
  <testsuite name="a_name" tests="5" failures="1" time="0.1">
    <testcase classname="a_class" name="a_test" time="0.1"> # a passing test
    </testcase>
    <testcase classname="a_class" name="a_test" time="0.1">
      <failure message="a_message">
      </failure>
    </testcase>
  </testsuite>
  ...
</testsuites>
```

The use of `trainer` and `junit.xml` was arbitrary. I just wanted the information in some structured format, and this was what I found. 

2. Now that the information is structured, I can query it. Since it's in xml, I use `xmlstarlet`. My command queries for elements matching (`sel -t -m`) `//testsuites/testsuite/testcase` that include (`-i`) a `failure`. It grabs the value (`-v`) of the `classname` and `name` attributes, outputting (`-o`) a " - " between them.

3. `xmlstarlet` queries for elements matching (`sel -t -m`) `//testsuites`, getting the values (`-v`) of the `@tests` count and `@failures` count, and outputting some helpful text around those values.

It's always fun to cobble these types of scripts together. Hopefully this is useful to you.