## test

```python:ヘッダファイルを解析するサンプル
import sys
from pycparser import c_parser, c_ast, parse_file

# https://github.com/eliben/pycparser/blob/master/examples/cdecl.py
def _explain_type(decl):
    """ Recursively explains a type decl node
    """
    typ = type(decl)

    if typ == c_ast.TypeDecl:
        quals = ' '.join(decl.quals) + ' ' if decl.quals else ''
        return quals + _explain_type(decl.type)
    elif typ == c_ast.Typename or typ == c_ast.Decl:
        return _explain_type(decl.type)
    elif typ == c_ast.IdentifierType:
        return ' '.join(decl.names)
    elif typ == c_ast.PtrDecl:
        quals = ' '.join(decl.quals) + ' ' if decl.quals else ''
        return quals + _explain_type(decl.type) + '*'
    elif typ == c_ast.ArrayDecl:
        arr = 'array'
        if decl.dim: 
            arr = '[%s]' % decl.dim.value
        else:
            arr = '[]'

        return _explain_type(decl.type) + arr

    elif typ == c_ast.FuncDecl:
        if decl.args:
            params = [_explain_type(param) for param in decl.args.params]
            args = ', '.join(params)
        else:
            args = ''

        return ('function(%s) returning ' % (args) +
                _explain_type(decl.type))

    elif typ == c_ast.Struct:
        decls = [_explain_decl_node(mem_decl) for mem_decl in decl.decls]
        members = ', '.join(decls)

        return ('struct%s ' % (' ' + decl.name if decl.name else '') +
                ('containing {%s}' % members if members else ''))

def show_func_defs(filename):
    # Note that cpp is used. Provide a path to your own cpp or
    # make sure one exists in PATH.
    ast = parse_file(filename, use_cpp=False,
                     cpp_args=r'-Iutils/fake_libc_include')
    if not isinstance(ast, c_ast.FileAST):
        return

    for ext in ast.ext:
        if type(ext.type) == c_ast.FuncDecl:
            print(f"function name: {ext.name} -----------------------")
            args = ''
            print("parameters----------------------------------------")
            if ext.type.args:
                for arg in ext.type.args:
                    print(f"{_explain_type(arg)} {arg.name}")
            print("return----------------------------------------")
            print(_explain_type(ext.type.type))

if __name__ == "__main__":
    if len(sys.argv) > 1:
        filename  = sys.argv[1]
    else:
        exit(-1)

    show_func_defs(filename)
```  
