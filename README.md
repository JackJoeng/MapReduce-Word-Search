# MapReduce

This repo provides two examples and a script to easily define and run mapper and reducer functions against Hadoop MapReduce.

Generally speaking, on the top level there are folders that must contain:

* mapper.py
* reducer.py
* sample directory with data to be copied in hdfs within docker container

__ENSURE THAT .py FILES HAVE `chmod +x` PERMISSIONS__
(This is hadoop requirement)

Of course, the idea is to add more folders that demonstrate different aggregations that can be achieved with MapReduce over different datasaets (that are available in the sample folder). The aspiration was to make something reusable quickly and cheaply.

## Usage

1. Download this repo
2. CD into this repo
3. Run the following

```
docker run \
  -v $(pwd):/usr/local/hadoop/py \
  -it sequenceiq/hadoop-docker:2.7.1 \
  /usr/local/hadoop/py/py_runner.sh basic_grep
```
(notice the **grep** keyword at the end - corresponds to the folder **basic_grep**!)

expected output:

```
foo	6
quux	4
```

Another example:

```
docker run \
  -v $(pwd):/usr/local/hadoop/py \
  -it sequenceiq/hadoop-docker:2.7.1 \
  /usr/local/hadoop/py/py_runner.sh count
```
(notice the **count** keyword at the end  - corresponds to the folder **count**!)

expected output:

```
bar	0
foo	6
labs	0
quux	4
```

*Question*: How would you update the simple grep above to manage __any__ type of search? (In this case it encodes the "f" / "x" searching inside the reducer function). So basically, what if I wanted to find all the words that have "oo" or all the words that start in "k" but end in "e" or all the words that have a single capital letter in them?

As you can imagine, the fix is not to hardcode all of these scenarios inside the map/reduce functions but instead, to come up with a more generic way to solve this.

*Solution*: 

1. Creat another folder called "search."
2. Create the "mapper.py" and the "reducer.py" in the "search" folder.
3. In the "mapper.py", write

```
#!/usr/bin/env python
"""mapper.py"""

import sys
import re

criteria = sys.argv[-1]
# input comes from STDIN (standard input)
for line in sys.stdin:
# remove leading and trailing whitespace
line = line.strip()
# split the line into words
words = line.split()
# increase counters
for word in words:
# write the results to STDOUT (standard output);
# what we output here will be the input for the
# Reduce step, i.e. the input for reducer.py
#
# tab-delimited; the trivial word count is 1
if re.search(criteria, word):
print '%s\t%s' % (word, 1)
else:
print '%s\t%s' % (word, 0)
```
3. In the "reducer.py", copy the same code from the "reducer.py" in the "basic_grep" folder

To find words that start wth "f" or end in "x":

```
docker run \
-v $(pwd):/usr/local/hadoop/py \
-it sequenceiq/hadoop-docker:2.7.1 \
/usr/local/hadoop/py/py_runner.sh search "^f|x$"
```

To find words that have "oo":
```
docker run \
-v $(pwd):/usr/local/hadoop/py \
-it sequenceiq/hadoop-docker:2.7.1 \
/usr/local/hadoop/py/py_runner.sh search "oo"
```

To find words that start with "k" or end with "e":

```
docker run \
-v $(pwd):/usr/local/hadoop/py \
-it sequenceiq/hadoop-docker:2.7.1 \
/usr/local/hadoop/py/py_runner.sh search "^k|e$"
```

To find words that has one upper letter:

```
docker run \
-v $(pwd):/usr/local/hadoop/py \
-it sequenceiq/hadoop-docker:2.7.1 \
/usr/local/hadoop/py/py_runner.sh search "^[A-Z].*[A-Z]$"
```


