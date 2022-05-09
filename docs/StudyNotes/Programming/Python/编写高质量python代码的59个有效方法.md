# 第一章 用Pythonic的方式来思考

1. 确认自己使用的Python版本

   ```shell
   python --version
   python3 --version
   ```

   ```python
   import sys
   
   print(sys.version_info)
   print(sys.version)
   ```

   * 有Cpython、Jpython、IronPython、PyPy等多种Python运行时环境
   * 使用Python3

2. 遵循[PEP8](https://peps.python.org/pep-0008/)风格指南

3. 了解bytes、str与unicode的区别

   * utf-8就是一种将Unicode字符表示为二进制数据（即原始8位值）的方法

   * 通过编写函数实现bytes和str之间的转换

     ```python
     def to_str(bytes_or_str):
         if isinstance(bytes_or_str, bytes):
             value = bytes_or_str.decode("utf-8")
         else:
             value = bytes_or_str
         return value
     
     
     def to_bytes(bytes_or_str):
         if isinstance(bytes_or_str, str):
             value = bytes_or_str.encode("utf-8")
         else:
             value = bytes_or_str
         return value
     ```

   * Python3中open函数默认是以utf-8格式进行操作，而Python2默认是以二进制形式操作

4. 使用辅助函数取代复杂的表达式

5. 了解切割序列的办法

   * `somelist[start,end]`，start作为起始索引是包含在内的，而end作为终止索引是不包含在内的。

   * ```python
     last_twenty_items = a[-20:]
     ```

6. 在单次切片操作内，不要同时指定start、end和stride

7. 用列表推导来来取代map和filter

   * dict和set也支持推到表达式

     ```python
     chile_ranks = {'ghost': 1, "habanero": 2, "cayenne": 3}
     rank_dict = {rank:name for name,rank in chile_ranks.items()}
     chile_len_set={len(name) for name in chile_ranks.keys()}
     ```

 8.  不要使用含有两个以上表达式的列表推导

     * 列表推导支持多级循环，每一集循环也支持多项条件。

       ```python
       matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
       flat = [x for row in matrix for x in row]
       squard = [[x ** 2 for x in row] for row in matrix]
       ```

 9.  用生成式表达式来改写数据量较大的列表推导

 10.  尽量使用enumerate取代range

 11.  用zip函数同时遍历两个迭代器

      ```python
      names = ['Cecilia', 'Lise', 'Marie']
      letters = [len(n) for n in names]
      for name,letter in zip(names,letters):
          print(name,letter)
      ```

 12.  不要再for和while循环后写else块

 13.  合理利用try/except/else/finally结构中的每个代码块

      * `try/except/else`

        如果try块中没有发生异常，则执行else块中的代码

      * try/except/else/finally




# 第二章 函数

14. 尽量用异常来表示特殊情况，而不要返回None

    如果使用None来表示特殊情况，那么在使用if进行判断的时候可能会因为多种情况的杂糅而导致含义不明。

    ```python
    def divide(a, b):
        try:
            return a / b
        except ZeroDivisionError:
            return None
    ```

    使用`if(divide(a.,b))`可能会出现分子为0和分母为0都判断为False的情况，这种情况可以使用两种方法应对：

    1. 返回值拆为两部分，放入一个二元组中

       ```python
       def divide(a, b):
           try:
               return True, a / b
           except ZeroDivisionError:
               return False, None
       
       success, result = divide(2, 5)
       if not success:
           print("Invalid inputs")
       ```

       但是Python程序员习惯使用下划线跳过自己不感兴趣的值

       ```python
       _, result = divide(1, 2)
       if not result:
           print("Invalid inputs")
       ```

    2. 继续抛出异常而非返回None，从而强迫调用者处理异常

       ```python
       def divide(a, b):
           try:
               return a / b
           except ZeroDivisionError as e:
               raise ValueError("Invalid inputs") from e
       
       
       x, y = 5, 2
       try:
           result = divide(x, y)
       except ValueError:
           print("Invalid inputs")
       else:
           print('result is %.1f' % result)
       ```

15. 了解如何在闭包里使用外围作用域中的变量

    ```python
    def sort_priority(values, group):
        def helper(x):
            if x in group:
                return (0, x)
            return (1, x)
    
        values.sort(key=helper)
    
    
    numbers = [8, 3, 1, 2, 5, 4, 7, 6]
    group = {2, 3, 5, 7}
    sort_priority(numbers, group)
    print(numbers)
    ```

    * Python支持闭包
    * Python元祖的比较规则，先比较_1,然后以此类推

    如何获取闭包内的数据

    ```python
    def sort_priority(values, group):
        found = False
    
        def helper(x):
            nonlocal found
            if x in group:
                found = True
                return (0, x)
            return (1, x)
    
        values.sort(key=helper)
        return found
    
    numbers = [8, 3, 1, 2, 5, 4, 7, 6]
    group = {2, 3, 5, 7}
    print(sort_priority(numbers, group))
    print(numbers)
    ```

    * `nonlocal`的限制表示在给相关变量赋值时应该在撒好难过层作用于中查找该变量
    * `nonlocal`的限制在于它不能延伸到模块级别
    * `global`可以与`nonlocal`的限制互补

16. 考虑用生成器来改写直接返回列表的函数

    ```python
    # 返回每个单次的首字母在String中的index
    def index_words(text):
        result = []
        if text:
            result.append(0)
        for index, letter in enumerate(text):
            if letter == ' ':
                result.append(index + 1)
        return result
    
    
    address = "Four score and seven years ago..."
    result=index_words(address)
    print(result)
    ```

    改写为生成器版本

    ```python
    def index_words(text):
        if text:
            yield 0
        for index, letter in enumerate(text):
            if letter == ' ':
                yield index + 1
    
    
    address = "Four score and seven years ago..."
    result = list(index_words(address))
    print(result)
    ```