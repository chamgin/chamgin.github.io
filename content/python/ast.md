---
title: "Python 代码插桩及语法树分析"
date: 2021-10-17T22:33:39+08:00
toc: false
images:
---

 python支持对编写的程序进行源代码的改动，例如通过AST模块将代码进行解析成语法树，然后在遍历语法节点的过程中对代码进行改动

 本文主要讨论Python的ast语法解析及代码插桩过程。

---

### Python的ast功能

#### 字符串代码解释
Python支持执行以字符串形式提供的源代码，例如：
```
	>>> x = 42
	>>> eval('2 + 3*4 + x')
	56
	>>> exec('for i in range(5): print(i)')
	0
	1
	2
	3
	4
```

#### Python 语法树转换
可以使用ast模块将python源代码编译为一个抽象语法树，这样就可以分析源代码，例如：
```
>>> import ast
>>> ex = ast.parse('2 + 3*4 +x', mode='eval')
>>> ex
<ast.Expression object at 0x7f19a73c1d90>
>>> ast.dump(ex)
"Expression(body=BinOp(left=BinOp(left=Constant(value=2), op=Add(), right=BinOp(left=Constant(value=3), op=Mult(), right=Constant(value=4))), op=Add(), right=Name(id='x', ctx=Load())))"
```

ast的parse方法会将源代码解析为语法树的节点，mode参数可以选择为:
* exec, 代表编译一个模块
* single, 一个单独的语句
* eval，一个表达式
通过dump方法可以看到展开的语法树，树的结构如下图：

---

ast模块支持对各个树的节点进行操作，通过定义一个访问者类，在类中实现了各种的visit_NodeName()方法，把感兴趣的节点保留下来

下面给出了一个类似的例子，记录被加载、保存和删除过的节点名称：
```
import ast

class CodeAnalyzer(ast.NodeVisitor):
	def __init__(self):
		self.loaded = set()
		self.stored = set()
		self.deleted = set()
	def visit_Name(self, node):
		if isinstance(node.ctx, ast.Load):
			self.loaded.add(node.id)
		elif isinstance(node.ctx, ast.Store):
			self.stored.add(node.id)
		elif isinstance(node.ctx, ast.Del):
			self.deleted.add(node.id)

if __name__ == '__main__':
	code = '''
for i in range(10):
    print(i)
del i
	'''

	top = ast.parse(code, mode='exec')

	c = CodeAnalyzer()
	c.visit(top)
	print('Loaded:', c.loaded)
	print('Stored:', c.stored)
	print('Deleted:', c.deleted)
```

这里通过对visit_Name的方法保留节点的名字，可以解析出code变量代码的节点名称。运行代码的输出：
```
Loaded: {'i', 'print', 'range'}
Stored: {'i'}
Deleted: {'i'}
```

---

除了遍历节点名字，还可以通过函数，然后修改代码以此达到插桩代码的效果。

下面会增加一个装饰器，该装饰器会对被装饰的函数进行代码的插桩，通过插入gloabls变量将全局变量变成局部变量，来比对其中的性能：

```
# namelower.py
import ast
import inspect

# Node visitor that lowers globally accessed names into
# the function body as local variables.
class NameLower(ast.NodeVisitor):
	def __init__(self, lowered_names):
		self.lowered_names = lowered_names

	def visit_FunctionDef(self, node):
		# Compile some assignment to lower the constants
		code = '_globals = globals()\n'
		code += '\n'.join("{0} = _globals['{0}']".format(name)
				for name in self.lowered_names)
		code_ast = ast.parse(code, mode='exec')

		# Inject new statements into the function body
		node.body[:0] = code_ast.body

		# Save the functions object
		self.func = node
	
# Decorator that turns global names into locals
def lower_names(*namelist):
	def lower(func):
		srclines = inspect.getsource(func).splitlines()
		# Skip source lines prior to the @lower_names decorator
		for n, line in enumerate(srclines):
			if '@lower_names' in line:
				break

		src = '\n'.join(srclines[n+1:])
		# Hack to deal with indented code
		if src.startswith((' ', '\t')):
			src = 'if 1:\n' + src
		top = ast.parse(src, mode='exec')

		# Transform the AST
		cl = NameLower(namelist)
		cl.visit(top)

		# Execute the modified AST
		temp = {}
		exec(compile(top, '', 'exec'), temp, temp)

		# Pull out the modified code object
		func.__code__ = temp[func.__name__].__code__
		return func
	return lower

# test.py
from namelower import lower_names
from functools import wraps
import time

def timethis(func):
	@wraps(func)
	def wrapper(*args, **kwargs):
		start = time.perf_counter()
		r = func(*args, **kwargs)
		end = time.perf_counter()
		print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
		return r
	return wrapper

INCR = 1

@timethis
@lower_names('INCR')
def countdown(n):
	while n > 0:
		n -= INCR

@timethis
def countdown_raw(n):
	while n > 0:
		n -= INCR

if __name__ == '__main__':
	print('no add local variable')
	countdown_raw(20000000)
	print('add local variable')
	countdown(20000000)
```
其中有两个文件，一个是namelower.py模块，提供了lower_names装饰器，对被装饰的函数进行插桩，被插桩后的countdown函数如下：

```
def countdown(n):
	_globals = globals()
	INCR = _globals['INCR']
	while n > 0:
		n -= INCR
```
然后执行test.py文件进行性能测试，测试中能发现插桩后的代码性能提升了20%:
```
no add local variable
__main__.countdown_raw : 1.2053479370079003
add local variable
__main__.countdown : 0.9712031570088584
```
