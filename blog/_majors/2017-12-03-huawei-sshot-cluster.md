---
title: 'Script: Screenshot Cluster'
tags: [ script, python, huawei, sshot, starred, sort ]
---

```python
import os
import re
import itertools

def fnamesfx(filename, suffix):
    name, ext = os.path.splitext(filename)
    return '{}_{}{}'.format(name, suffix, ext)

vnsal00_screenshot_pattern = re.compile(
    r'Screenshot_\d{4}-\d{2}-\d{2}-\d{2}-\d{2}-\d{2}.(png|jpg)')

def work():
    d = dict()

    for i in os.listdir('.'):
        if vnsal00_screenshot_pattern.match(i):
            date = i[11:21]
            if not date in d:
                    d[date] = []
            d[date].append(i)
            if not os.path.exists(date):
                os.mkdir(date)

    for i in d.keys():
        for j in d[i]:
            target = '{0}/{1}'.format(i, j)
            try:
                os.rename(j, target)
            except FileExistsError:
                for suffix in itertools.count(2):
                    try:
                        target2 = fnamesfx(target, suffix)
                        os.rename(j, target2)
                        print('renamed {} to {}'.format(target, target2))
                        break
                    except FileExistsError:
                        pass
work()
```
