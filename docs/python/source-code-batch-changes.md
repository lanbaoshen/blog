# Source Code Batch Changes

Sometimes, existing code needs to be modified in batches, 
and these operations are too complex to be achieved through the IDE's replacement function.

[AST (Abstract Syntax Trees)](https://docs.python.org/3/library/ast.html) is a good choice to do that.

For example, perform operations on the following code:

:   * Replace class name `Old` with `New`  
    * Replace class variable name `OLD` with `NEW`  
    * Replace `#!python dict({key: value})` with `Base(key=value)`
    * Remove empty class
    * Remove `main`

```python title="elements.py"
class OldElements:
    FOO = ''
    OLD_BTN = {'text': 'Lanbao'}
    OLD_CHECKBOX = {'resource_id': 'Lanbao'}
    OLD_BAR = {'xpath': FOO + 'Lanbao'}

    
class FooElements(OldElements):
    FOO = ' '
    OLD_BTN = {'text': ''}
    
    
class EmptyElements:
    ...


if __name__ == '__main__':
    ...
```

Visiting ast node through NodeTransformer:

```python title="transformer.py"
from ast import *

import astor


class _BaseTransformer(NodeTransformer):
    def visit_ClassDef(self, node: ClassDef):
        if len(node.body) == 1:
            return None

        if node.name.startswith('Old'):
            node.name = node.name.replace('Old', 'New', 1)
            
        for base in node.bases:
            if base.id.startswith('Old'):
                base.id = base.id.replace('Old', 'New', 1)
                
        return node

    def visit_If(self, node: If):
        if node.test.left.id == '__name__':
            return None


class ElementTransformer(_BaseTransformer):
    def visit_ClassDef(self, node: ClassDef):
        if not (node := super().visit_ClassDef(node)):
            return None

        for sub_node in node.body:
            if not isinstance(sub_node, Assign):
                continue
            for target in sub_node.targets:
                if target.id.startswith('OLD'):
                    target.id = target.id.replace('OLD', 'NEW', 1)

                if isinstance(sub_node.value, Dict):
                    keywords = [
                        keyword(arg=key.value, value=value)
                        for key, value in zip(sub_node.value.keys, sub_node.value.values)
                    ]

                    sub_node.value = Call(
                        func=Name(id='Base', ctx=Load()),
                        args=[],
                        keywords=keywords,
                    )
                    
        return node


if __name__ == '__main__':
    tree = astor.parse_file('elements.py')
    et = ElementTransformer()
    modify_tree = et.visit(tree)
    with open('new_elements.py', 'w') as file:
        file.write(unparse(modify_tree))
```

Execute `transformer.py` to get the modified code:

```python title="new_elements.py"
class NewElements:
    FOO = ''
    NEW_BTN = Base(text='Lanbao')
    NEW_CHECKBOX = Base(resource_id=f'{FOO}Lanbao')
    NEW_BAR = Base(xpath=FOO + 'Lanbao')

class FooElements(NewElements):
    FOO = ' '
    NEW_BTN = Base(text='')
```

!!! Warning
    Notice that `ast.parse` discards all the comments. 
    And if you need to format the code, you may want to explore tools or libraries like `autopep8` or `black`.
