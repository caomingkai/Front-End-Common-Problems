---
title: Web Template Engine
date: 2018-03-27 10:02:04
type: "categories"
comments: true
categories: 
- Portfolios
tags:
- Python
---

### I. Main idea: 
similar to Python string's **format method** like " A is {title} of B".format(title="father"); Whereas, **template engine** is also a formatter, but much more powerful.
> An important phase in any web application is generating HTML to be served to the browser.

> Very few HTML pages are completely static: they involve at least a small amount of __dynamic data__, such as the user's name. Usually, they contain a great deal of dynamic data: product listings, friends' news updates, and so on.

> The mostly-static style used in templates is the opposite of how most programming languages work.

> A template language flips this around: **the template file is mostly static literal text, with special notation to indicate the executable dynamic parts.**

### II. tech stacks
 **Python -- Template language Django**

**Github**: https://github.com/caomingkai/500-Lines-Or-Less

### III. template syntax
1.  dot : 
```
<p>The price is: {{product.price}}, with a {{product.discount}}% discount.</p>
```
2. filters with pipe character: ```
<p>Short name: {{story.subject|slugify|lower}}</p>
```
3. logic control: 
```
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}: {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```
4. comment:
```
{# This is the best template ever! #}
```
### III. Two approach to implement the Template Engine
1. Two main phases: **parsing the template**, and then **rendering the template**

2. For 'parsing phase':
- interpretation (Django )
- compilation (Jinja2 and Mako | also, we will use)


### IV. Compiling to Python
1. The template engine will compile templates to Python method called **"render_function(context, do_dots)"**
2. context dictionary : carry kinds of dynamic variables to be embedded in template
3. do_dots method: passed-in method name, handling dot access

### V. Code_bulider
1. Being called by template object, to help to read passed-in lines, using **'add-line()'**
2. Integrate lines into a well-formed python script
3. return to templage object the final executable script by calling " exec(python_source, global_namespace)"

```html
Code-builder
    |__   add_line()    : read in ready-to-run python lines
    |__   add_section() : set aside space for later variable assignment
    |__   indent()      : indent to form well-formed python
    |__   dedent()      : dedent to form well-formed python
    |__   get_globles() : 1- call "exec(python_source, global_namespace)"
                          2- return executable script by calling get_globles()['render_function']
```
### VI. Template
1. used to handle template, and parse it into executable lines
2. call functions of Code-builder object, to produce an executable script to output 'HTML string'
3. with helper function: do_dots()

```html
Template
    |__   __init__(context):   1- read context( ie, values for var in template )
    |                          2- break down template text by Regex into 4 cases
    |                          3- call functions of Code_bulider to produce HTML
    |
    |__   4 cases:             1- Literal content(pure string)
    |                          2- {{ ..... }} variables, converted to string
    |                             For case 1 and 2, just concatenate them;
    |                          3- {# ..... #}
    |                             Just ignore
    |                          4- {% ..... %} control statements
    |                             4.1- ops_stack() for nested control statements
    |                             4.2- filter_handler() for 'var|upper'
    |                             4.3- indent()/ dedent()
    |
    |__   self._render_function() = code.get_globles['render_function'];
    |
    |__   render():             calls self._render_function() with context param
```

### VII. Template Test

- great introduction of unittest: https://www.jianshu.com/p/38948d0d73f5
- unittest是Python自带的单元测试框架，我们可以用其来作为我们自动化测试框架的用例组织执行框架

1. unittest的流程：写好TestCase，然后由TestLoader加载TestCase到TestSuite，然后由TextTestRunner来运行TestSuite，运行的结果保存在TextTestResult中，我们通过命令行或者unittest.main()执行时，main会调用TextTestRunner中的run来执行，或者我们可以直接通过TextTestRunner来执行用例。
2. 一个class继承unittest.TestCase即是一个TestCase，其中以 test 开头的方法在load时被加载为一个真正的TestCase。
3. verbosity参数可以控制执行结果的输出，0 是简单报告、1 是一般报告、2 是详细报告。
可以通过addTest和addTests向suite中添加case或suite，可以用TestLoader的loadTestsFrom__()方法
4. 用 setUp()、tearDown()、setUpClass()以及 tearDownClass()可以在用例执行前布置环境，以及在用例执行后清理环境
5. 我们可以通过skip，skipIf，skipUnless装饰器跳过某个case，或者用TestCase.skipTest方法。
参数中加stream，可以将报告输出到文件：可以用TextTestRunner输出txt报告，以及可以用HTMLTestRunner输出html报告。
