---
title: LRU cache
date: 2018-09-11 14:43:28
categories: algorithm
tags:
- algorithm
- operate system
- cache
---

一个 LRU Cache，为了方便，直接用 C++ 写的。

LRU (Least Recently Used) 缓存分配算法是指，**在缓存空间不足时，淘汰最近最少使用的 Cache**。

为了让查询和插入删除的时间复杂度都是 $O(1)$，用的是哈希表 + 链表实现，链表保证删除插入为 $O(1)$，哈希表保证查询为 $O(1)$。

每个缓存用 key-value 类型表示，哈希表中存放 key 和到链表中该 key 节点的指针，链表中存放 key-value。

## 思路

```cpp
unordered<key, list::iterator> m;

list<pair<key,value>> l;
```

- put()
    - map 中是否存在
        - 存在找到其在 list 中的位置，放到 list 的 front，更新 map 的映射
        - 不存在，放到 list 的 front，map 中添加映射
    - 判断缓存是否超出 maxsize，超出淘汰 list 的 last，并在 map 删除 key
- get()
    - map 是否存在
        - 存在，移动到 list 的 front，更新 map 的映射
        - 不存在，说明缓存未击中

## 实现

```cpp
/**
 * @author pwxcoo
 * @email pwxcoo@gmail.com
 * @create date 2018-09-11 10:30:47
 * @modify date 2018-09-11 13:28:41
 * @desc LRU cache implemented by c++
*/

#ifndef _LRUCACHE_INCLUDED_
#define _LRUCACHE_INCLUDED_

#include <unordered_map>
#include <list>
#include <cstddef>
#include <stdexcept>
#include <iostream>

namespace cache
{

template <typename key_t, typename value_t>
class lru_cache
{
  public:
    typedef typename std::pair<key_t, value_t> key_value_pair_t;
    typedef typename std::list<key_value_pair_t>::iterator list_iterator_it;

    lru_cache(size_t max_size) : _max_size(max_size) {}

    void put(const key_t &key, const value_t &value)
    {
        auto it = _cache_items_map.find(key);
        if (it != _cache_items_map.end())
        {
            _cache_items_list.erase(it->second);
        }
        _cache_items_list.push_front(key_value_pair_t(key, value));
        _cache_items_map[key] = _cache_items_list.begin();

        if (_cache_items_map.size() > _max_size)
        {
            _cache_items_map.erase(_cache_items_list.rbegin()->first);
            _cache_items_list.pop_back();
        }
    }

    const value_t &get(const key_t &key)
    {
        auto it = _cache_items_map.find(key);
        if (it == _cache_items_map.end())
        {
            throw std::range_error("There is no such key in cache!");
        }
        else
        {
            auto hit = it->second;
            _cache_items_list.erase(hit);
            _cache_items_list.push_front(*hit);
            _cache_items_map[key] = _cache_items_list.begin();
            return hit->second;
        }
    }

    bool exists(const key_t &key) const
    {
        return _cache_items_map.find(key) == _cache_items_list.end();
    }

    size_t size() const
    {
        return _cache_items_map.size();
    }

    void debug(std::string s) const
    {
        std::cout << s << " cache size:" << size() << " | ";
        for (auto item : _cache_items_list)
        {
            std::cout << " {" << item.first << ", " << item.second << "}"
                      << " ";
        }
        std::cout << std::endl;
    }

  private:
    std::list<key_value_pair_t> _cache_items_list;
    std::unordered_map<key_t, list_iterator_it> _cache_items_map;
    size_t _max_size;
};

} // namespace cache

#endif /* _LRUCACHE_HPP_INCLUDED_ */

//////////////////////////////////// TEST MODULE BEGIN ////////////////////////////////////

#include <assert.h>

int main()
{
    printf("-------------------[TEST] LRU cache begin-------------------------\n\n");

    cache::lru_cache<int, int> cache_lru(5);

    for (int i = 0; i < 6; i++)
    {
        cache_lru.debug("before:");
        printf("put() {key:%d, value:%d} to cache\n", i, i * i);

        cache_lru.put(i, i * i);
        assert(cache_lru.get(i) == i * i);

        cache_lru.debug("after :");
        printf("\n");
    }

    for (int i = 0; i < 3; i++)
    {
        cache_lru.debug("before:");

        try
        {
            printf("get() {key:%d} from cache: %d\n", i, cache_lru.get(i));
        }
        catch (const std::exception &e)
        {
            printf("encountered exception when get {key:%d}: %s\n", i, e.what());
        }

        cache_lru.debug("after :");
        printf("\n");
    }

    printf("-------------------[TEST] LRU cache end----------------------------\n");
}

//////////////////////////////////// TEST MODULE END   /////////////////////////////////////////
```

## References

- [lru.cpp - Gist](https://gist.github.com/pwxcoo/fe9bf9310c03c4ed2d21976d210975fd)
- [lamerman/cpp-lru-cache](https://github.com/lamerman/cpp-lru-cache)