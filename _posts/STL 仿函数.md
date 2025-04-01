通过仿函数可以实现算法操作类型的泛化，为算法注入了灵魂，极大地提高了算法的灵活性：

- accumulate：如果将操作类型修改为 multiple，则计算的就是所有元素的乘积
- sort：如果将比较函数修改为 greater，则就是从大到小排序



仿函数的本质是：“行为类似函数” 的对象，重载了函数调用运算符的类的对象。



# Helper

为了可以被适配器适配，一元仿函数需要继承 unary_function、二元仿函数需要继承 binary_function

```cpp
/**
   *  Helper for defining adaptable unary function objects.
   *  @deprecated Deprecated in C++11, no longer in the standard since C++17.
   */
  template<typename _Arg, typename _Result>
    struct unary_function
    {
      /// @c argument_type is the type of the argument
      typedef _Arg 	argument_type;   

      /// @c result_type is the return type
      typedef _Result 	result_type;  
    } _GLIBCXX11_DEPRECATED;
```



```cpp
/**
   *  Helper for defining adaptable binary function objects.
   *  @deprecated Deprecated in C++11, no longer in the standard since C++17.
   */
  template<typename _Arg1, typename _Arg2, typename _Result>
    struct binary_function
    {
      /// @c first_argument_type is the type of the first argument
      typedef _Arg1 	first_argument_type; 

      /// @c second_argument_type is the type of the second argument
      typedef _Arg2 	second_argument_type;

      /// @c result_type is the return type
      typedef _Result 	result_type;
    } _GLIBCXX11_DEPRECATED;
```



# 算数运算仿函数

```CPP
/// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct plus : public binary_function<_Tp, _Tp, _Tp>
    {
      /// Returns the sum
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x + __y; }
    };

  /// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct minus : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x - __y; }
    };

  /// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct multiplies : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x * __y; }
    };

  /// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct divides : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x / __y; }
    };

  /// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct modulus : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x % __y; }
    };

  /// One of the @link arithmetic_functors math functors@endlink.
  template<typename _Tp>
    struct negate : public unary_function<_Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x) const
      { return -__x; }
    };
```



# 关系运算仿函数

```cpp
  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct equal_to : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x == __y; }
    };

  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct not_equal_to : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x != __y; }
    };

  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct greater : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x > __y; }
    };

  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct less : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x < __y; }
    };

  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct greater_equal : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x >= __y; }
    };

  /// One of the @link comparison_functors comparison functors@endlink.
  template<typename _Tp>
    struct less_equal : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x <= __y; }
    };
```



# 逻辑运算仿函数

```cpp
 /// One of the @link logical_functors Boolean operations functors@endlink.
  template<typename _Tp>
    struct logical_and : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x && __y; }
    };

  /// One of the @link logical_functors Boolean operations functors@endlink.
  template<typename _Tp>
    struct logical_or : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x || __y; }
    };

  /// One of the @link logical_functors Boolean operations functors@endlink.
  template<typename _Tp>
    struct logical_not : public unary_function<_Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool
      operator()(const _Tp& __x) const
      { return !__x; }
    };
```



# 位运算仿函数

```cpp
// _GLIBCXX_RESOLVE_LIB_DEFECTS
  // DR 660. Missing Bitwise Operations.
  template<typename _Tp>
    struct bit_and : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x & __y; }
    };

  template<typename _Tp>
    struct bit_or : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x | __y; }
    };

  template<typename _Tp>
    struct bit_xor : public binary_function<_Tp, _Tp, _Tp>
    {
      _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x, const _Tp& __y) const
      { return __x ^ __y; }
    };

  template<typename _Tp>
    struct bit_not : public unary_function<_Tp, _Tp>
    {
    _GLIBCXX14_CONSTEXPR
      _Tp
      operator()(const _Tp& __x) const
      { return ~__x; }
    };
```


