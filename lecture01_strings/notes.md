# Strings

## Problems

Safety:
`strcpy(const chat* invalid_ptr, const char* src);`

Solution:
`strncpy(const chat* dest, const char* src, size_t n);`

Can't make n-function for `strlen()`;

Discussion:

String length in C is not an invariant.
We need to keep this invariant safe -> make incapsulation.

Popular string implementations:
- CString
- QString
- CComBSTR
- FBString -- Facebook

## std::string

- Data
- Size
- Capacity -- for amortization

## Hometask

Copy constructor and assignment operator for String

## Discussion

Why do we need 0-terminator if we have size?
1. Fast access to size
2. Back compatibility

`std::string::size_type`

## Problems of strings

1. Memory allocation

```
string astr = "hello";
string bstr;
bstr.reserve(15); // can cause memory allocation
```

2. Strings formation

a. String

```
std::string a = ssl ? "https" : "http";
a = a + "://" + path + "/";
```

b. fstreams

`std::stringstream ss;`

FMT, starting from C++20

`auto fmt = std::format("{}://{}/{}", (ssl ? "https" : "http"), path, query);`

Example with format fmt:

`#include <format>`

raises an error, is not in a standart except MVS C++;

Problem 2: Static strings

`static const std::string kName = "FOO"; // copy`

allocation before main()
can cause bad alloc

`static const char* kName = "FOO";`

also copy

Solution: 
`string_view` from C++17

Non owning pointer

address and size

static const std::string_view kName = "FOO"

String view doesn't own the pointer.

```C++
remove_prefix()
remove_suffix()
```

Problem: short strings

Often they are identical.

Dozens of copies in a program.
Is it a problem of a class or is it a problem of a programm?

It's a class string problem.

Optimization:

COW == Copy on Write idiom

```C++
class stingbuf {
	char* data;
	size_t size;
	size_t capacity;
	int refcount;

	// etc
}

class string{
	stringbug* buf;
}

string s1 = "hello";
string s2 = s1; // no copying
```

By GOOGLE, 20% optimization for Search Engine, many short strings.

Interview task: Lazy String

It's a shared object and it has shared refcount -> problems in multiprocessing.

Procs:

1. Memory optimization
2. Copying time

Cons:

1. Multiprocessing problem
2. More data fields
3. Two layers pointers
4. Pointer invalidation

Pointer Invalidation:

```C++
string a = "Hello";
const char* p = &a[3];
a += "world"; // after this p should not be used

string s("str");
const char* p = s.data();

std::string s2(s);
s[0] = 'S';
```

p is not valid anymore for COW strings

С++11 operator[] can't invalidate pointers

So it cancels implementation for COW std::string

It pushed the others to make many custom implementations.

Control block of 24 bytes: data, capacity, size

SSO = Small String Optimization

```C++
class string {
	size_type size_;
	union {
		struct{
			char* data_;
			size_type capacity_;
		} large_;
		char small_[sizeof(large_)];
	};
	// etc
};
```

Problems: move constructor has many if brackets.

GCC string:

32 byte: data points to hea, size > 15, capacity, padding

data, size < 15, size = 16 vyte for small string

Problem with UTF32

If one symbol takes 4 bytes instead of 1, it's more difficult:
Then we make templates.

## String Templates

basic_string

```C++
template <typename CharT> class basic_string {
	CharT* data;
	size_t size;
	union {
		size_t capacity;
		enum {SZ = ()};
		CharT small_str[SZ];
		} sso;
	public:
	// 89 methods
}
```

It will not work for `basic_string<float>` because we need to make comparisons.

Solution: add Traits

```C++
template <typename CharT,
		typename Traits = std::char_traits<CharT>>
class basic_string {
	...
}
```

## Allocators

Memory allocator is abstracting the memory allocation.

```C++
template <typename CharT,
		typename Traits = std::char_traits<CharT>
		typename Allocator std::allocator<CharT>>
class basic_string {
	...
}
```

## Boost tokenizer

Symbol is a generalization point.
So if we change the string code works in the same way.

## Problem

Implement `operator==()` for basic_string
is it a class member or a free function?

## Literature:

1. ISO Information technology C++ 14882:2020
2. Stroustrup C++ 4th Editoion 2013
3. Nicholas Ormrod, CppCon16 The strange detail of string at Facebook
