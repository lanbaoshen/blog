# BFS and DFS in Dict Traversal

I answered such a question on Stack Overflow: 
[How do I extract all keys from this JSON file?](https://stackoverflow.com/questions/72470261/how-do-i-extract-all-keys-from-this-json-file/72470505#72470505)

Retrieve all the dict keys from a dict whose values consist of multiple Iterable obj.

```python
data = {
    'k1': 'Lanbao',
    'k2': {
        'k3': 'Lanbao',
        'k4': [
            'Lanbao',
            {
                'k5': 'Lanbao',
                'k6': ['Lanbao']
            }
        ],
        'k7': 'Lanbao',
    },
    'k8': 'Lanbao'
}
```

By constructing generators to implement **BFS** (Breadth-First Search) and **DFS** (Depth-First Search) respectively.

```python title='bfs.py'
from collections import deque
from collections.abc import Iterable


def get_keys(data, ignore_type=(str, bytes)):
    queue = deque([data])  
    while queue:
        node = queue.popleft()
        if isinstance(node, dict):
            for k, v in node.items():
                yield k
                queue.append(v)
        elif isinstance(node, Iterable) and not isinstance(node, ignore_type):
            for i in node:
                queue.append(i)
```

```python title='dfs.py'
from collections.abc import Iterable


def get_keys(data, ignore_type=(str, bytes)):
    if isinstance(data, dict):
        for k, v in data.items():
            yield k
            yield from get_keys(v)
    elif isinstance(data, Iterable) and not isinstance(data, ignore_type):
        for i in data:
            yield from get_keys(i)
```
