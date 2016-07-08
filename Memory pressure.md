# Howto measure memory pressure
The main goal is to get a probe to show when lack of memory impacts a computer's performances.

## Bibliography
[SOF - Which metric should I use to determine when a server is low on memory?](http://serverfault.com/questions/437138/which-metric-should-i-use-to-determine-when-a-server-is-low-on-memory)

## Counting the number of pages activation/deactivation
```python
import os
import sys
import re
import time

PERIODIC = 1
pgs = re.compile('^MemTotal:\s+([0-9]+) kB\n([a-zA-Z]+:\s+[0-9]+ kB\n){5}Active:\s+([0-9]+) kB\nInactive:\s+([0-9]+) kB')
meminfo = open('/proc/meminfo')

def read_meminfo():
    content = meminfo.read(4096)
    m = pgs.search(content)
    total, active, inactive = m.group(1), m.group(3), m.group(4)
    meminfo.seek(0, 0)
    return total, active,inactive

if __name__ == "__main__":
    old_active = "0"
    old_inactive = "0"
    while True:
        time.sleep(PERIODIC)
        total, active, inactive = read_meminfo()

        value_total = float(total);
        absolute_pressure = (abs(float(old_active) - float(active)) + abs(float(old_inactive) - float(inactive))) / 2.0;
        relative_pressure = absolute_pressure / (value_total * 0.01)
        print "Relative Pressure:\t%d %%" % relative_pressure

        old_active = active
        old_inactive = inactive
  ```
Not great : we need to integrate time notion to the algorithm.

## Amount of activated/deactivated memory over time

