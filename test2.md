## test

```python
import sys
def _explain_type(decl):
    return ('struct%s ' % (' ' + decl.name if decl.name else '') + ('containing {%s}' % members if members else ''))

```  
