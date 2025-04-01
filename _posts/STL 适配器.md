适配器？就是你给它一个东西，它给你包装成另一个东西，虽然表面上看起来不一样，但骨子里的东西是没有变的。



根据作用对象的不同，可以分为以下几种：容器适配器、迭代器适配器、仿函数适配器



# 仿函数适配器

仿函数适配器：接收一/多个仿函数，将其封装成另一个仿函数

- bind：绑定仿函数数适配器，接收一个仿函数，及绑定列表，将其封装成一个 “bind 仿函数”
- compose1/compose2：组合仿函数适配器，接收多个仿函数，将其封装成一个 “compose 仿函数”

## bind

本质上是一个函数模板，返回一个适配器对象。

```cpp
vector<int> a(10);
iota(a.begin(), a.end(), 0);
// 将 apt 的第一个参数作为 less 的第一个参数
// 将 8 作为 less 的第二个参数
auto apt = bind(less<int>(), _1, 8);
int cnt = count_if(a.begin(), a.end(), apt);
cout << cnt << endl;
```

```cpp
template<bool _SocketLike, typename _Func, typename... _BoundArgs>
    struct _Bind_helper
    : _Bind_check_arity<typename decay<_Func>::type, _BoundArgs...>
    {
      typedef typename decay<_Func>::type __func_type;
      typedef _Bind<__func_type(typename decay<_BoundArgs>::type...)> type;
    };

  // Partial specialization for is_socketlike == true, does not define
  // nested type so std::bind() will not participate in overload resolution
  // when the first argument might be a socket file descriptor.
  template<typename _Func, typename... _BoundArgs>
    struct _Bind_helper<true, _Func, _BoundArgs...>
    { };

  /**
   *  @brief Function template for std::bind.
   *  @ingroup binders
   *  @since C++11
   */
  template<typename _Func, typename... _BoundArgs>
    inline _GLIBCXX20_CONSTEXPR typename
    _Bind_helper<__is_socketlike<_Func>::value, _Func, _BoundArgs...>::type
    bind(_Func&& __f, _BoundArgs&&... __args)
    {
      typedef _Bind_helper<false, _Func, _BoundArgs...> __helper_type;
      return typename __helper_type::type(std::forward<_Func>(__f),
					  std::forward<_BoundArgs>(__args)...);
    }
```



## not_fn

取反仿函数适配器

```cpp
auto greater3 = bind(greater<int>(), _1, 3);
auto lesseq3 = not_fn(greater3);
cout << lesseq3(2, 3) << endl;
cout << lesseq3(3, 3) << endl;
cout << lesseq3(4, 3) << endl;

auto greatereq = not_fn(less<int>());
cout << greatereq(20, 21) << endl;
cout << greatereq(20, 20) << endl;
cout << greatereq(20, 19) << endl;
```

```cpp
/// Generalized negator.
  template<typename _Fn>
    class _Not_fn
    {
      template<typename _Fn2, typename... _Args>
	using __inv_res_t = typename __invoke_result<_Fn2, _Args...>::type;

      template<typename _Tp>
	static decltype(!std::declval<_Tp>())
	_S_not() noexcept(noexcept(!std::declval<_Tp>()));

    public:
      template<typename _Fn2>
	constexpr
	_Not_fn(_Fn2&& __fn, int)
	: _M_fn(std::forward<_Fn2>(__fn)) { }

      _Not_fn(const _Not_fn& __fn) = default;
      _Not_fn(_Not_fn&& __fn) = default;
      ~_Not_fn() = default;

      // Macro to define operator() with given cv-qualifiers ref-qualifiers,
      // forwarding _M_fn and the function arguments with the same qualifiers,
      // and deducing the return type and exception-specification.
#define _GLIBCXX_NOT_FN_CALL_OP( _QUALS )				\
      template<typename... _Args>					\
	_GLIBCXX20_CONSTEXPR						\
	decltype(_S_not<__inv_res_t<_Fn _QUALS, _Args...>>())		\
	operator()(_Args&&... __args) _QUALS				\
	noexcept(__is_nothrow_invocable<_Fn _QUALS, _Args...>::value	\
	    && noexcept(_S_not<__inv_res_t<_Fn _QUALS, _Args...>>()))	\
	{								\
	  return !std::__invoke(std::forward< _Fn _QUALS >(_M_fn),	\
				std::forward<_Args>(__args)...);	\
	}
      _GLIBCXX_NOT_FN_CALL_OP( & )
      _GLIBCXX_NOT_FN_CALL_OP( const & )
      _GLIBCXX_NOT_FN_CALL_OP( && )
      _GLIBCXX_NOT_FN_CALL_OP( const && )
#undef _GLIBCXX_NOT_FN_CALL_OP

    private:
      _Fn _M_fn;
    };

  // [func.not_fn] Function template not_fn
#define __cpp_lib_not_fn 201603L
  /** Wrap a function object to create one that negates its result.
   *
   * The function template `std::not_fn` creates a "forwarding call wrapper",
   * which is a function object that wraps another function object and
   * when called, forwards its arguments to the wrapped function object.
   *
   * The result of invoking the wrapper is the negation (using `!`) of
   * the wrapped function object.
   *
   *  @ingroup functors
   *  @since C++17
   */
  template<typename _Fn>
    _GLIBCXX20_CONSTEXPR
    inline auto
    not_fn(_Fn&& __fn)
    noexcept(std::is_nothrow_constructible<std::decay_t<_Fn>, _Fn&&>::value)
    {
      return _Not_fn<std::decay_t<_Fn>>{std::forward<_Fn>(__fn), 0};
    }
```



## mem_fn

返回类的成员函数

```cpp
string s = "cc";
auto spb = mem_fn(&string::push_back);
spb(s, 'a');
cout << s << endl;
```

```cpp
  template<typename _MemFunPtr,
	   bool __is_mem_fn = is_member_function_pointer<_MemFunPtr>::value>
    class _Mem_fn_base
    : public _Mem_fn_traits<_MemFunPtr>::__maybe_type
    {
      using _Traits = _Mem_fn_traits<_MemFunPtr>;

      using _Arity = typename _Traits::__arity;
      using _Varargs = typename _Traits::__vararg;

      template<typename _Func, typename... _BoundArgs>
	friend struct _Bind_check_arity;

      _MemFunPtr _M_pmf;

    public:

      using result_type = typename _Traits::__result_type;

      explicit constexpr
      _Mem_fn_base(_MemFunPtr __pmf) noexcept : _M_pmf(__pmf) { }

      template<typename... _Args>
	_GLIBCXX20_CONSTEXPR
	auto
	operator()(_Args&&... __args) const
	noexcept(noexcept(
	      std::__invoke(_M_pmf, std::forward<_Args>(__args)...)))
	-> decltype(std::__invoke(_M_pmf, std::forward<_Args>(__args)...))
	{ return std::__invoke(_M_pmf, std::forward<_Args>(__args)...); }
    };

  // Partial specialization for member object pointers.
  template<typename _MemObjPtr>
    class _Mem_fn_base<_MemObjPtr, false>
    {
      using _Arity = integral_constant<size_t, 0>;
      using _Varargs = false_type;

      template<typename _Func, typename... _BoundArgs>
	friend struct _Bind_check_arity;

      _MemObjPtr _M_pm;

    public:
      explicit constexpr
      _Mem_fn_base(_MemObjPtr __pm) noexcept : _M_pm(__pm) { }

      template<typename _Tp>
	_GLIBCXX20_CONSTEXPR
	auto
	operator()(_Tp&& __obj) const
	noexcept(noexcept(std::__invoke(_M_pm, std::forward<_Tp>(__obj))))
	-> decltype(std::__invoke(_M_pm, std::forward<_Tp>(__obj)))
	{ return std::__invoke(_M_pm, std::forward<_Tp>(__obj)); }
    };

  template<typename _MemberPointer>
    struct _Mem_fn; // undefined

  template<typename _Res, typename _Class>
    struct _Mem_fn<_Res _Class::*>
    : _Mem_fn_base<_Res _Class::*>
    {
      using _Mem_fn_base<_Res _Class::*>::_Mem_fn_base;
    };
  /// @endcond

  // _GLIBCXX_RESOLVE_LIB_DEFECTS
  // 2048.  Unnecessary mem_fn overloads
  /**
   * @brief Returns a function object that forwards to the member pointer
   * pointer `pm`.
   *
   * This allows a pointer-to-member to be transformed into a function object
   * that can be called with an object expression as its first argument.
   *
   * For a pointer-to-data-member the result must be called with exactly one
   * argument, the object expression that would be used as the first operand
   * in a `obj.*memptr` or `objp->*memptr` expression.
   *
   * For a pointer-to-member-function the result must be called with an object
   * expression and any additional arguments to pass to the member function,
   * as in an expression like `(obj.*memfun)(args...)` or
   * `(objp->*memfun)(args...)`.
   *
   * The object expression can be a pointer, reference, `reference_wrapper`,
   * or smart pointer, and the call wrapper will dereference it as needed
   * to apply the pointer-to-member.
   *
   * @ingroup functors
   * @since C++11
   */
  template<typename _Tp, typename _Class>
    _GLIBCXX20_CONSTEXPR
    inline _Mem_fn<_Tp _Class::*>
    mem_fn(_Tp _Class::* __pm) noexcept
    {
      return _Mem_fn<_Tp _Class::*>(__pm);
    }
```



## function

是一个类，将一个可调用类封装为另一个可调用类

```cpp
  /**
   *  @brief Polymorphic function wrapper.
   *  @ingroup functors
   *  @since C++11
   */
  template<typename _Res, typename... _ArgTypes>
    class function<_Res(_ArgTypes...)>
    : public _Maybe_unary_or_binary_function<_Res, _ArgTypes...>,
      private _Function_base
    {
      // Equivalent to std::decay_t except that it produces an invalid type
      // if the decayed type is the current specialization of std::function.
      template<typename _Func,
	       bool _Self = is_same<__remove_cvref_t<_Func>, function>::value>
	using _Decay_t
	  = typename __enable_if_t<!_Self, decay<_Func>>::type;

      template<typename _Func,
	       typename _DFunc = _Decay_t<_Func>,
	       typename _Res2 = __invoke_result<_DFunc&, _ArgTypes...>>
	struct _Callable
	: __is_invocable_impl<_Res2, _Res>::type
	{ };

      template<typename _Cond, typename _Tp = void>
	using _Requires = __enable_if_t<_Cond::value, _Tp>;

      template<typename _Functor>
	using _Handler
	  = _Function_handler<_Res(_ArgTypes...), __decay_t<_Functor>>;

    public:
      typedef _Res result_type;

      // [3.7.2.1] construct/copy/destroy

      /**
       *  @brief Default construct creates an empty function call wrapper.
       *  @post `!(bool)*this`
       */
      function() noexcept
      : _Function_base() { }

      /**
       *  @brief Creates an empty function call wrapper.
       *  @post @c !(bool)*this
       */
      function(nullptr_t) noexcept
      : _Function_base() { }

      /**
       *  @brief %Function copy constructor.
       *  @param __x A %function object with identical call signature.
       *  @post `bool(*this) == bool(__x)`
       *
       *  The newly-created %function contains a copy of the target of
       *  `__x` (if it has one).
       */
      function(const function& __x)
      : _Function_base()
      {
	if (static_cast<bool>(__x))
	  {
	    __x._M_manager(_M_functor, __x._M_functor, __clone_functor);
	    _M_invoker = __x._M_invoker;
	    _M_manager = __x._M_manager;
	  }
      }

      /**
       *  @brief %Function move constructor.
       *  @param __x A %function object rvalue with identical call signature.
       *
       *  The newly-created %function contains the target of `__x`
       *  (if it has one).
       */
      function(function&& __x) noexcept
      : _Function_base(), _M_invoker(__x._M_invoker)
      {
	if (static_cast<bool>(__x))
	  {
	    _M_functor = __x._M_functor;
	    _M_manager = __x._M_manager;
	    __x._M_manager = nullptr;
	    __x._M_invoker = nullptr;
	  }
      }

      /**
       *  @brief Builds a %function that targets a copy of the incoming
       *  function object.
       *  @param __f A %function object that is callable with parameters of
       *  type `ArgTypes...` and returns a value convertible to `Res`.
       *
       *  The newly-created %function object will target a copy of
       *  `__f`. If `__f` is `reference_wrapper<F>`, then this function
       *  object will contain a reference to the function object `__f.get()`.
       *  If `__f` is a null function pointer, null pointer-to-member, or
       *  empty `std::function`, the newly-created object will be empty.
       *
       *  If `__f` is a non-null function pointer or an object of type
       *  `reference_wrapper<F>`, this function will not throw.
       */
      // _GLIBCXX_RESOLVE_LIB_DEFECTS
      // 2774. std::function construction vs assignment
      template<typename _Functor,
	       typename _Constraints = _Requires<_Callable<_Functor>>>
	function(_Functor&& __f)
	noexcept(_Handler<_Functor>::template _S_nothrow_init<_Functor>())
	: _Function_base()
	{
	  static_assert(is_copy_constructible<__decay_t<_Functor>>::value,
	      "std::function target must be copy-constructible");
	  static_assert(is_constructible<__decay_t<_Functor>, _Functor>::value,
	      "std::function target must be constructible from the "
	      "constructor argument");

	  using _My_handler = _Handler<_Functor>;

	  if (_My_handler::_M_not_empty_function(__f))
	    {
	      _My_handler::_M_init_functor(_M_functor,
					   std::forward<_Functor>(__f));
	      _M_invoker = &_My_handler::_M_invoke;
	      _M_manager = &_My_handler::_M_manager;
	    }
	}

      /**
       *  @brief Function assignment operator.
       *  @param __x A %function with identical call signature.
       *  @post `(bool)*this == (bool)x`
       *  @returns `*this`
       *
       *  The target of `__x` is copied to `*this`. If `__x` has no
       *  target, then `*this` will be empty.
       *
       *  If `__x` targets a function pointer or a reference to a function
       *  object, then this operation will not throw an exception.
       */
      function&
      operator=(const function& __x)
      {
	function(__x).swap(*this);
	return *this;
      }

      /**
       *  @brief Function move-assignment operator.
       *  @param __x A %function rvalue with identical call signature.
       *  @returns `*this`
       *
       *  The target of `__x` is moved to `*this`. If `__x` has no
       *  target, then `*this` will be empty.
       *
       *  If `__x` targets a function pointer or a reference to a function
       *  object, then this operation will not throw an exception.
       */
      function&
      operator=(function&& __x) noexcept
      {
	function(std::move(__x)).swap(*this);
	return *this;
      }

      /**
       *  @brief Function assignment to empty.
       *  @post `!(bool)*this`
       *  @returns `*this`
       *
       *  The target of `*this` is deallocated, leaving it empty.
       */
      function&
      operator=(nullptr_t) noexcept
      {
	if (_M_manager)
	  {
	    _M_manager(_M_functor, _M_functor, __destroy_functor);
	    _M_manager = nullptr;
	    _M_invoker = nullptr;
	  }
	return *this;
      }

      /**
       *  @brief Function assignment to a new target.
       *  @param __f  A function object that is callable with parameters of
       *              type  `_ArgTypes...` and returns a value convertible
       *              to `_Res`.
       *  @return `*this`
       *  @since C++11
       *
       *  This function object wrapper will target a copy of `__f`. If `__f`
       *  is `reference_wrapper<F>`, then this function object will contain
       *  a reference to the function object `__f.get()`. If `__f` is a null
       *  function pointer or null pointer-to-member, this object will be
       *  empty.
       *
       *  If `__f` is a non-null function pointer or an object of type
       *  `reference_wrapper<F>`, this function will not throw.
       */
      template<typename _Functor>
	_Requires<_Callable<_Functor>, function&>
	operator=(_Functor&& __f)
	noexcept(_Handler<_Functor>::template _S_nothrow_init<_Functor>())
	{
	  function(std::forward<_Functor>(__f)).swap(*this);
	  return *this;
	}

      /// @overload
      template<typename _Functor>
	function&
	operator=(reference_wrapper<_Functor> __f) noexcept
	{
	  function(__f).swap(*this);
	  return *this;
	}

      // [3.7.2.2] function modifiers

      /**
       *  @brief Swap the targets of two %function objects.
       *  @param __x A %function with identical call signature.
       *
       *  Swap the targets of `this` function object and `__f`.
       *  This function will not throw exceptions.
       */
      void swap(function& __x) noexcept
      {
	std::swap(_M_functor, __x._M_functor);
	std::swap(_M_manager, __x._M_manager);
	std::swap(_M_invoker, __x._M_invoker);
      }

      // [3.7.2.3] function capacity

      /**
       *  @brief Determine if the %function wrapper has a target.
       *
       *  @return `true` when this function object contains a target,
       *  or `false` when it is empty.
       *
       *  This function will not throw exceptions.
       */
      explicit operator bool() const noexcept
      { return !_M_empty(); }

      // [3.7.2.4] function invocation

      /**
       *  @brief Invokes the function targeted by `*this`.
       *  @returns the result of the target.
       *  @throws `bad_function_call` when `!(bool)*this`
       *
       *  The function call operator invokes the target function object
       *  stored by `this`.
       */
      _Res
      operator()(_ArgTypes... __args) const
      {
	if (_M_empty())
	  __throw_bad_function_call();
	return _M_invoker(_M_functor, std::forward<_ArgTypes>(__args)...);
      }

#if __cpp_rtti
      // [3.7.2.5] function target access
      /**
       *  @brief Determine the type of the target of this function object
       *  wrapper.
       *
       *  @returns the type identifier of the target function object, or
       *  `typeid(void)` if `!(bool)*this`.
       *
       *  This function will not throw exceptions.
       */
      const type_info&
      target_type() const noexcept
      {
	if (_M_manager)
	  {
	    _Any_data __typeinfo_result;
	    _M_manager(__typeinfo_result, _M_functor, __get_type_info);
	    if (auto __ti =  __typeinfo_result._M_access<const type_info*>())
	      return *__ti;
	  }
	return typeid(void);
      }
#endif

      /**
       *  @brief Access the stored target function object.
       *
       *  @return Returns a pointer to the stored target function object,
       *  if `typeid(_Functor).equals(target_type())`; otherwise, a null
       *  pointer.
       *
       * This function does not throw exceptions.
       *
       * @{
       */
      template<typename _Functor>
	_Functor*
	target() noexcept
	{
	  const function* __const_this = this;
	  const _Functor* __func = __const_this->template target<_Functor>();
	  // If is_function_v<_Functor> is true then const_cast<_Functor*>
	  // would be ill-formed, so use *const_cast<_Functor**> instead.
	  return *const_cast<_Functor**>(&__func);
	}

      template<typename _Functor>
	const _Functor*
	target() const noexcept
	{
	  if _GLIBCXX17_CONSTEXPR (is_object<_Functor>::value)
	    {
	      // For C++11 and C++14 if-constexpr is not used above, so
	      // _Target_handler avoids ill-formed _Function_handler types.
	      using _Handler = _Target_handler<_Res(_ArgTypes...), _Functor>;

	      if (_M_manager == &_Handler::_M_manager
#if __cpp_rtti
		  || (_M_manager && typeid(_Functor) == target_type())
#endif
		 )
		{
		  _Any_data __ptr;
		  _M_manager(__ptr, _M_functor, __get_functor_ptr);
		  return __ptr._M_access<const _Functor*>();
		}
	    }
	  return nullptr;
	}
      /// @}

    private:
      using _Invoker_type = _Res (*)(const _Any_data&, _ArgTypes&&...);
      _Invoker_type _M_invoker = nullptr;
    };
```

