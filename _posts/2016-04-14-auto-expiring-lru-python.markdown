---
layout: post
title:  "Auto-expiring LRU cache implementation in python"
date:   2016-04-14 23:50:00
categories: main
tags :
     - programming
     - python
---

```python
import random
import time
import unittest

__all__=['Cache']

_current_time_ms = lambda: int(round(time.time() * 1000))


"""
Doublelinked Cache node
"""
class CacheNode(object):
    def __init__(self, key, val, valid_time_ms=0):
        # if valid_time_ms is 0 disable auto-expiring
        self.auto_expiring = True if valid_time_ms else False
        self.expiring_time = valid_time_ms + _current_time_ms()
        self.key = key
        self.val = val
        self.left = None
        self.right = None

    def is_expired(self):
        return self.auto_expiring and \
            _current_time_ms() > self.expiring_time


class Cache(object):
    """
    A time based auto expiring LRU cache implementation
    """
    def __init__(self, max_cap, expiring_time=0):
        self.expiring_time_unit_ms = expiring_time
        self.max_cap = max_cap
        self.cache_ref = {}
        self.cache_head = CacheNode('head', 0)
        self.cache_tail = CacheNode('tail', 0)
        # linked cache
        self.cache_head.right = self.cache_tail
        self.cache_tail.left = self.cache_head

    def __getitem__(self, key):
        return self.get(key)

    def __setitem__(self, key, val):
        self.put(key, val)

    def __len__(self):
        return len(self.cache_ref)

    def get(self, key):
        val = None
        if key in self.cache_ref.keys():
            node = self._remove(key)
            if not node.is_expired():
                self.cache_ref[key] = node
                self._insert_at_head(node)
                val = node.val
        return val

    def put(self, key, val):
        # if key exits, update it first
        if key in self.cache_ref.keys():
            node = self.cache_ref.pop(key)
            self._remove_node(node)
        node = CacheNode(key, val, self.expiring_time_unit_ms)
        self.cache_ref[key] = node
        self._insert_at_head(node)
        if self.__len__() > self.max_cap:
            tail = self._remove_from_tail()
            self.cache_ref.pop(tail.key)

    # remove key/val from ref too
    def _remove(self, key):
        node = self.cache_ref.pop(key)
        self._remove_node(node)
        return node

    #
    # Only manapulate cache list
    #
    def _insert_at_head(self, node):
        r = self.cache_head.right
        self.cache_head.right = node
        node.left = self.cache_head
        node.right = r
        r.left = node

    def _remove_from_tail(self):
        tail = self.cache_tail.left
        self._remove_node(tail)
        return tail

    def _remove_node(self, node):
        l = node.left
        r = node.right
        l.right = r
        r.left = l
        node.left, node.right = None, None


class Test(unittest.TestCase):

    def test_cache(self):
        c1 = Cache(5, 1000 * 5)
        # without auto-expiring
        c2 = Cache(100, 0)
        for i in range(1000):
            c1[str(random.randint(1,10))] = i
            c2[str(random.randint(1,10))] = i
        for i in c1.cache_ref.keys():
            c1[i]
        for i in c2.cache_ref.keys():
            c2[i]
        # cache should be valid
        self.assertEqual(5, len(c1))
        self.assertEqual(10, len(c2))
        time.sleep(5)
        self.assertEqual(5, len(c1))
        self.assertEqual(10, len(c2))
        for i in c1.cache_ref.keys():
            c1[i]
        for i in c2.cache_ref.keys():
            c2[i]
        self.assertEqual(0, len(c1))
        self.assertEqual(10, len(c2))


if __name__ == '__main__':
    unittest.main()
```

[Implementation on Github](https://github.com/haocs/algorithm_practice/blob/master/libs/cache.py)
