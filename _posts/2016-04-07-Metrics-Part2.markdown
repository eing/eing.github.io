---
layout: post
title:  "Data to understand behavior - Part 2"
date:   2016-04-07 07:43:59
author: Eing Ong
categories: Metrics
tags: Perforce
---
Another good use of data is to understand behavior in order to identify what we need to correct. If engineers are taking a longer time to understand, fix and add new code to a code base, it could be that the code has low code coverage, poor documentation and many other [code smells](http://www.industriallogic.com/wp-content/uploads/2005/09/smellstorefactorings.pdf). One other issue is much of the code is orphaned and there is no owner for a long time. Learning such code without unit tests, documentation or any engineer who is familiar with the code can be an issue. 

<h2>How orphaned is your code base</h2>
I wanted to find out how much code is orphaned, i.e. has not be modified for past 2, 3, 4 and 5 years. This code base is in P4 and I could use "p4 fstat -T headModTime {filename}" command to find the last modified time. Here's the script.

~~~text
cd ${project_workspace}

# Be sure you have these configured
# P4HOSTNAME=
# P4CLIENT=
# P4USER=
# P4PORT=

# find last modified time of all java files
find . -name "*.java" -exec p4 fstat -T headModTime {} >> headModTime \; 

# remove any blank lines
awk 'NF > 0' headModTime > headModTimeBlankLinesRemoved

# last check-in is 1 or more years ago
awk 'BEGIN { FS = " " }; { if ($3 < 1425769728) print $3 }' headModTimeBlankLinesRemoved > headModTime1YearOrMore
# last check-in is 2 or more years ago
awk 'BEGIN { FS = " " }; { if ($3 < 1394233728) print $3 }' headModTimeBlankLinesRemoved > headModTime2YearOrMore
# last check-in is 3 or more years ago
awk 'BEGIN { FS = " " }; { if ($3 < 1362697728) print $3 }' headModTimeBlankLinesRemoved > headModTime3YearOrMore
# last check-in is 4 or more years ago
awk 'BEGIN { FS = " " }; { if ($3 < 1331161728) print $3 }' headModTimeBlankLinesRemoved > headModTime4YearOrMore
# last check-in is 5 years ago
awk 'BEGIN { FS = " " }; { if ($3 < 1299539330) print $3 }' headModTimeBlankLinesRemoved > headModTime5YearOrMore

# get line count for each year
echo Number of files not modified in last 1 year:
wc -l headModTime1Year
echo Number of files not modified in last 2 year:
wc -l headModTime2Year
echo Number of files not modified in last 3 year:
wc -l headModTime3Year
echo Number of files not modified in last 4 year:
wc -l headModTime4Year
echo Number of files not modified in last 5 year:
wc -l headModTime5Year

# get total number of java files
echo Total number of java files
find . -name '*.java' | xargs cat | wc -l
~~~

Here's how you can get the timestamps for past 5 years.

~~~text
# 1 years ago timestamp
$ date -j -f "%m%d%Y" 03072015 "+%s"
1425769728
# 2 years ago timestamp
$ date -j -f "%m%d%Y" 03072014 "+%s"
1394233728
# 3 years ago timestamp
$ date -j -f "%m%d%Y" 03072013 "+%s"
1362697728
# 4 years ago timestamp
$ date -j -f "%m%d%Y" 03072012 "+%s"
1331161728
# 5 years ago timestamp
$ date -j -f "%m%d%Y" 03072011 "+%s"
1299539330

# timestamp to date lookup
$ date -r 1299539330
Mon Mar  7 15:08:50 PST 2011
~~~

<h2>Other good blogs</h2>

Last but not least, this week I had the opportunity to meet with Marty, Drew & Bill from [AKF partners](http://akfpartners.com). Their [blogs](http://akfpartners.com/techblog/all_posts/) and [books](http://www.amazon.com/Martin-L.-Abbott/e/B002KYQTZ4/ref=sr_ntt_srch_lnk_1?qid=1459967398&sr=1-1) are highly regarded in the industry. They have a few blogs on using data as metrics as well -

[Selecting metrics for your Agile teams](http://akfpartners.com/techblog/2015/01/19/selecting-metrics-agile-teams/)
[Engineering metrics](http://akfpartners.com/techblog/2012/11/01/engineering-metrics/)

Finally, I'll leave you with the good ol' quote <i>"You can't improve what you don't measure"</i>, use the right data to drive change and behavior to achieve the outcome.
