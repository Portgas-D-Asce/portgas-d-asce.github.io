# 最值算法

## min_element

```cpp
template<typename _ForwardIterator, typename _Compare>
    _GLIBCXX14_CONSTEXPR
    _ForwardIterator
    __min_element(_ForwardIterator __first, _ForwardIterator __last,
		  _Compare __comp)
    {
      if (__first == __last)
	return __first;
      _ForwardIterator __result = __first;
      while (++__first != __last)
	if (__comp(__first, __result))
	  __result = __first;
      return __result;
    }

  /**
   *  @brief  Return the minimum element in a range.
   *  @ingroup sorting_algorithms
   *  @param  __first  Start of range.
   *  @param  __last   End of range.
   *  @return  Iterator referencing the first instance of the smallest value.
  */
  template<typename _ForwardIterator>
    _GLIBCXX14_CONSTEXPR
    _ForwardIterator
    inline min_element(_ForwardIterator __first, _ForwardIterator __last)
    {
      // concept requirements
      __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(_LessThanComparableConcept<
	    typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);
      __glibcxx_requires_irreflexive(__first, __last);

      return _GLIBCXX_STD_A::__min_element(__first, __last,
				__gnu_cxx::__ops::__iter_less_iter());
    }

  /**
   *  @brief  Return the minimum element in a range using comparison functor.
   *  @ingroup sorting_algorithms
   *  @param  __first  Start of range.
   *  @param  __last   End of range.
   *  @param  __comp   Comparison functor.
   *  @return  Iterator referencing the first instance of the smallest value
   *  according to __comp.
  */
  template<typename _ForwardIterator, typename _Compare>
    _GLIBCXX14_CONSTEXPR
    inline _ForwardIterator
    min_element(_ForwardIterator __first, _ForwardIterator __last,
		_Compare __comp)
    {
      // concept requirements
      __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(_BinaryPredicateConcept<_Compare,
	    typename iterator_traits<_ForwardIterator>::value_type,
	    typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);
      __glibcxx_requires_irreflexive_pred(__first, __last, __comp);

      return _GLIBCXX_STD_A::__min_element(__first, __last,
				__gnu_cxx::__ops::__iter_comp_iter(__comp));
    }
```



## max_element

```cpp
template<typename _ForwardIterator, typename _Compare>
    _GLIBCXX14_CONSTEXPR
    _ForwardIterator
    __max_element(_ForwardIterator __first, _ForwardIterator __last,
		  _Compare __comp)
    {
      if (__first == __last) return __first;
      _ForwardIterator __result = __first;
      while (++__first != __last)
	if (__comp(__result, __first))
	  __result = __first;
      return __result;
    }

  /**
   *  @brief  Return the maximum element in a range.
   *  @ingroup sorting_algorithms
   *  @param  __first  Start of range.
   *  @param  __last   End of range.
   *  @return  Iterator referencing the first instance of the largest value.
  */
  template<typename _ForwardIterator>
    _GLIBCXX14_CONSTEXPR
    inline _ForwardIterator
    max_element(_ForwardIterator __first, _ForwardIterator __last)
    {
      // concept requirements
      __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(_LessThanComparableConcept<
	    typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);
      __glibcxx_requires_irreflexive(__first, __last);

      return _GLIBCXX_STD_A::__max_element(__first, __last,
				__gnu_cxx::__ops::__iter_less_iter());
    }

  /**
   *  @brief  Return the maximum element in a range using comparison functor.
   *  @ingroup sorting_algorithms
   *  @param  __first  Start of range.
   *  @param  __last   End of range.
   *  @param  __comp   Comparison functor.
   *  @return  Iterator referencing the first instance of the largest value
   *  according to __comp.
  */
  template<typename _ForwardIterator, typename _Compare>
    _GLIBCXX14_CONSTEXPR
    inline _ForwardIterator
    max_element(_ForwardIterator __first, _ForwardIterator __last,
		_Compare __comp)
    {
      // concept requirements
      __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(_BinaryPredicateConcept<_Compare,
	    typename iterator_traits<_ForwardIterator>::value_type,
	    typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);
      __glibcxx_requires_irreflexive_pred(__first, __last, __comp);

      return _GLIBCXX_STD_A::__max_element(__first, __last,
				__gnu_cxx::__ops::__iter_comp_iter(__comp));
    }
```

## min

```cpp
template<typename _Tp>
_GLIBCXX14_CONSTEXPR
inline _Tp
min(initializer_list<_Tp> __l)
{
  __glibcxx_requires_irreflexive(__l.begin(), __l.end());
  return *_GLIBCXX_STD_A::__min_element(__l.begin(), __l.end(),
  __gnu_cxx::__ops::__iter_less_iter());
}

template<typename _Tp, typename _Compare>
_GLIBCXX14_CONSTEXPR
inline _Tp
min(initializer_list<_Tp> __l, _Compare __comp)
{
  __glibcxx_requires_irreflexive_pred(__l.begin(), __l.end(), __comp);
  return *_GLIBCXX_STD_A::__min_element(__l.begin(), __l.end(),
  __gnu_cxx::__ops::__iter_comp_iter(__comp));
}
```



## max

```CPP
template<typename _Tp>
    _GLIBCXX14_CONSTEXPR
    inline _Tp
    max(initializer_list<_Tp> __l)
    {
      __glibcxx_requires_irreflexive(__l.begin(), __l.end());
      return *_GLIBCXX_STD_A::__max_element(__l.begin(), __l.end(),
	  __gnu_cxx::__ops::__iter_less_iter());
    }

  template<typename _Tp, typename _Compare>
    _GLIBCXX14_CONSTEXPR
    inline _Tp
    max(initializer_list<_Tp> __l, _Compare __comp)
    {
      __glibcxx_requires_irreflexive_pred(__l.begin(), __l.end(), __comp);
      return *_GLIBCXX_STD_A::__max_element(__l.begin(), __l.end(),
	  __gnu_cxx::__ops::__iter_comp_iter(__comp));
    }
```



# 其它

## unique

```cpp
template<typename _ForwardIterator, typename _BinaryPredicate>
    _GLIBCXX20_CONSTEXPR
    _ForwardIterator
    __unique(_ForwardIterator __first, _ForwardIterator __last,
	     _BinaryPredicate __binary_pred)
    {
      // Skip the beginning, if already unique.
      __first = std::__adjacent_find(__first, __last, __binary_pred);
      if (__first == __last)
	return __last;

      // Do the real copy work.
      _ForwardIterator __dest = __first;
      ++__first;
      while (++__first != __last)
	if (!__binary_pred(__dest, __first))
	  *++__dest = _GLIBCXX_MOVE(*__first);
      return ++__dest;
    }

  /**
   *  @brief Remove consecutive duplicate values from a sequence.
   *  @ingroup mutating_algorithms
   *  @param  __first  A forward iterator.
   *  @param  __last   A forward iterator.
   *  @return  An iterator designating the end of the resulting sequence.
   *
   *  Removes all but the first element from each group of consecutive
   *  values that compare equal.
   *  unique() is stable, so the relative order of elements that are
   *  not removed is unchanged.
   *  Elements between the end of the resulting sequence and @p __last
   *  are still present, but their value is unspecified.
  */
  template<typename _ForwardIterator>
    _GLIBCXX20_CONSTEXPR
    inline _ForwardIterator
    unique(_ForwardIterator __first, _ForwardIterator __last)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_ForwardIteratorConcept<
				  _ForwardIterator>)
      __glibcxx_function_requires(_EqualityComparableConcept<
		     typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);

      return std::__unique(__first, __last,
			   __gnu_cxx::__ops::__iter_equal_to_iter());
    }

  /**
   *  @brief Remove consecutive values from a sequence using a predicate.
   *  @ingroup mutating_algorithms
   *  @param  __first        A forward iterator.
   *  @param  __last         A forward iterator.
   *  @param  __binary_pred  A binary predicate.
   *  @return  An iterator designating the end of the resulting sequence.
   *
   *  Removes all but the first element from each group of consecutive
   *  values for which @p __binary_pred returns true.
   *  unique() is stable, so the relative order of elements that are
   *  not removed is unchanged.
   *  Elements between the end of the resulting sequence and @p __last
   *  are still present, but their value is unspecified.
  */
  template<typename _ForwardIterator, typename _BinaryPredicate>
    _GLIBCXX20_CONSTEXPR
    inline _ForwardIterator
    unique(_ForwardIterator __first, _ForwardIterator __last,
	   _BinaryPredicate __binary_pred)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_ForwardIteratorConcept<
				  _ForwardIterator>)
      __glibcxx_function_requires(_BinaryPredicateConcept<_BinaryPredicate,
		typename iterator_traits<_ForwardIterator>::value_type,
		typename iterator_traits<_ForwardIterator>::value_type>)
      __glibcxx_requires_valid_range(__first, __last);

      return std::__unique(__first, __last,
			   __gnu_cxx::__ops::__iter_comp_iter(__binary_pred));
    }
```



## reverse

```cpp
/**
   *  This is an uglified reverse(_BidirectionalIterator,
   *                              _BidirectionalIterator)
   *  overloaded for bidirectional iterators.
  */
  template<typename _BidirectionalIterator>
    _GLIBCXX20_CONSTEXPR
    void
    __reverse(_BidirectionalIterator __first, _BidirectionalIterator __last,
	      bidirectional_iterator_tag)
    {
      while (true)
	if (__first == __last || __first == --__last)
	  return;
	else
	  {
	    std::iter_swap(__first, __last);
	    ++__first;
	  }
    }

  /**
   *  This is an uglified reverse(_BidirectionalIterator,
   *                              _BidirectionalIterator)
   *  overloaded for random access iterators.
  */
  template<typename _RandomAccessIterator>
    _GLIBCXX20_CONSTEXPR
    void
    __reverse(_RandomAccessIterator __first, _RandomAccessIterator __last,
	      random_access_iterator_tag)
    {
      if (__first == __last)
	return;
      --__last;
      while (__first < __last)
	{
	  std::iter_swap(__first, __last);
	  ++__first;
	  --__last;
	}
    }

  /**
   *  @brief Reverse a sequence.
   *  @ingroup mutating_algorithms
   *  @param  __first  A bidirectional iterator.
   *  @param  __last   A bidirectional iterator.
   *  @return   reverse() returns no value.
   *
   *  Reverses the order of the elements in the range @p [__first,__last),
   *  so that the first element becomes the last etc.
   *  For every @c i such that @p 0<=i<=(__last-__first)/2), @p reverse()
   *  swaps @p *(__first+i) and @p *(__last-(i+1))
  */
  template<typename _BidirectionalIterator>
    _GLIBCXX20_CONSTEXPR
    inline void
    reverse(_BidirectionalIterator __first, _BidirectionalIterator __last)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_BidirectionalIteratorConcept<
				  _BidirectionalIterator>)
      __glibcxx_requires_valid_range(__first, __last);
      std::__reverse(__first, __last, std::__iterator_category(__first));
    }
```



## rotate

```cpp
/// This is a helper function for the rotate algorithm.
  template<typename _ForwardIterator>
    _GLIBCXX20_CONSTEXPR
    _ForwardIterator
    __rotate(_ForwardIterator __first,
	     _ForwardIterator __middle,
	     _ForwardIterator __last,
	     forward_iterator_tag)
    {
      if (__first == __middle)
	return __last;
      else if (__last == __middle)
	return __first;

      _ForwardIterator __first2 = __middle;
      do
	{
	  std::iter_swap(__first, __first2);
	  ++__first;
	  ++__first2;
	  if (__first == __middle)
	    __middle = __first2;
	}
      while (__first2 != __last);

      _ForwardIterator __ret = __first;

      __first2 = __middle;

      while (__first2 != __last)
	{
	  std::iter_swap(__first, __first2);
	  ++__first;
	  ++__first2;
	  if (__first == __middle)
	    __middle = __first2;
	  else if (__first2 == __last)
	    __first2 = __middle;
	}
      return __ret;
    }

   /// This is a helper function for the rotate algorithm.
  template<typename _BidirectionalIterator>
    _GLIBCXX20_CONSTEXPR
    _BidirectionalIterator
    __rotate(_BidirectionalIterator __first,
	     _BidirectionalIterator __middle,
	     _BidirectionalIterator __last,
	      bidirectional_iterator_tag)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_BidirectionalIteratorConcept<
				  _BidirectionalIterator>)

      if (__first == __middle)
	return __last;
      else if (__last == __middle)
	return __first;

      std::__reverse(__first,  __middle, bidirectional_iterator_tag());
      std::__reverse(__middle, __last,   bidirectional_iterator_tag());

      while (__first != __middle && __middle != __last)
	{
	  std::iter_swap(__first, --__last);
	  ++__first;
	}

      if (__first == __middle)
	{
	  std::__reverse(__middle, __last,   bidirectional_iterator_tag());
	  return __last;
	}
      else
	{
	  std::__reverse(__first,  __middle, bidirectional_iterator_tag());
	  return __first;
	}
    }

  /// This is a helper function for the rotate algorithm.
  template<typename _RandomAccessIterator>
    _GLIBCXX20_CONSTEXPR
    _RandomAccessIterator
    __rotate(_RandomAccessIterator __first,
	     _RandomAccessIterator __middle,
	     _RandomAccessIterator __last,
	     random_access_iterator_tag)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_RandomAccessIteratorConcept<
				  _RandomAccessIterator>)

      if (__first == __middle)
	return __last;
      else if (__last == __middle)
	return __first;

      typedef typename iterator_traits<_RandomAccessIterator>::difference_type
	_Distance;
      typedef typename iterator_traits<_RandomAccessIterator>::value_type
	_ValueType;

      _Distance __n = __last   - __first;
      _Distance __k = __middle - __first;

      if (__k == __n - __k)
	{
	  std::swap_ranges(__first, __middle, __middle);
	  return __middle;
	}

      _RandomAccessIterator __p = __first;
      _RandomAccessIterator __ret = __first + (__last - __middle);

      for (;;)
	{
	  if (__k < __n - __k)
	    {
	      if (__is_pod(_ValueType) && __k == 1)
		{
		  _ValueType __t = _GLIBCXX_MOVE(*__p);
		  _GLIBCXX_MOVE3(__p + 1, __p + __n, __p);
		  *(__p + __n - 1) = _GLIBCXX_MOVE(__t);
		  return __ret;
		}
	      _RandomAccessIterator __q = __p + __k;
	      for (_Distance __i = 0; __i < __n - __k; ++ __i)
		{
		  std::iter_swap(__p, __q);
		  ++__p;
		  ++__q;
		}
	      __n %= __k;
	      if (__n == 0)
		return __ret;
	      std::swap(__n, __k);
	      __k = __n - __k;
	    }
	  else
	    {
	      __k = __n - __k;
	      if (__is_pod(_ValueType) && __k == 1)
		{
		  _ValueType __t = _GLIBCXX_MOVE(*(__p + __n - 1));
		  _GLIBCXX_MOVE_BACKWARD3(__p, __p + __n - 1, __p + __n);
		  *__p = _GLIBCXX_MOVE(__t);
		  return __ret;
		}
	      _RandomAccessIterator __q = __p + __n;
	      __p = __q - __k;
	      for (_Distance __i = 0; __i < __n - __k; ++ __i)
		{
		  --__p;
		  --__q;
		  std::iter_swap(__p, __q);
		}
	      __n %= __k;
	      if (__n == 0)
		return __ret;
	      std::swap(__n, __k);
	    }
	}
    }

   // _GLIBCXX_RESOLVE_LIB_DEFECTS
   // DR 488. rotate throws away useful information
  /**
   *  @brief Rotate the elements of a sequence.
   *  @ingroup mutating_algorithms
   *  @param  __first   A forward iterator.
   *  @param  __middle  A forward iterator.
   *  @param  __last    A forward iterator.
   *  @return  first + (last - middle).
   *
   *  Rotates the elements of the range @p [__first,__last) by 
   *  @p (__middle - __first) positions so that the element at @p __middle
   *  is moved to @p __first, the element at @p __middle+1 is moved to
   *  @p __first+1 and so on for each element in the range
   *  @p [__first,__last).
   *
   *  This effectively swaps the ranges @p [__first,__middle) and
   *  @p [__middle,__last).
   *
   *  Performs
   *   @p *(__first+(n+(__last-__middle))%(__last-__first))=*(__first+n)
   *  for each @p n in the range @p [0,__last-__first).
  */
  template<typename _ForwardIterator>
    _GLIBCXX20_CONSTEXPR
    inline _ForwardIterator
    rotate(_ForwardIterator __first, _ForwardIterator __middle,
	   _ForwardIterator __last)
    {
      // concept requirements
      __glibcxx_function_requires(_Mutable_ForwardIteratorConcept<
				  _ForwardIterator>)
      __glibcxx_requires_valid_range(__first, __middle);
      __glibcxx_requires_valid_range(__middle, __last);

      return std::__rotate(__first, __middle, __last,
			   std::__iterator_category(__first));
    }

  } // namespace _V2
```



