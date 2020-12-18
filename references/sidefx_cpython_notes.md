# SideFX CPython Porting Notes

A collection of fixes and notes from porting Houdini's CPython code to Python 3.

## Version Checking

Python 3 version check:
```cpp
#if PY_MAJOR_VERSION >= 3
    // Run Python 3 code here.
#else
    // Run Python 2 code here.
#endif
```

More granular version check:
```cpp
#if PY_VERSION_HEX >= 0x03070000
    // Run code for Python 3.7 and above here.
#else
    // Run code for Python 3.6 and below here.
#endif
```

## Int to Long Conversion
Examples:
- `PyInt_Check` => `PyLong_Check`
- `PyInt_CheckExact`=> `PyLong_CheckExact`
- `PyInt_AsLong` => `PyLong_AsLong`
- `PyInt_FromLong` => `PyLong_FromLong`

**Note:**
Use `PyLong_AsLongLong` and `PyLong_FromLongLong` instead when dealing with large numbers (i.e. > 4 bytes).  The Windows `long` type is 4 bytes instead of 8.  Calling `PyLong_AsLong` on large numbers raises an exception on Windows.


## String to Unicode Conversion
Examples:
- `PyString_Check` => `PyUnicode_Check`
- `PyString_FromString` => `PyUnicode_FromString`
- `PyString_FromStringAndSize` => `PyUnicode_FromStringAndSize`
- `PyString_FromFormat` -> `PyUnicode_FromFormat`
- `PyString_AsString` => `PyUnicode_AsUTF8`

**Note:**
Use `PyBytes*` functions instead when storing raw binary data in strings.  See [Bytes API](https://docs.python.org/3/c-api/bytes.html).

## Files

Python removed `PyFile_AsFile`.  Use `PyObject_AsFileDescriptor` instead:
```cpp
PyObject *sysout = PySys_GetObject("stdout");
FILE *stdout_file = nullptr;
#if PY_MAJOR_VERSION >= 3
    int fd = PyObject_AsFileDescriptor(sysout);
    if (fd != -1)
        stdout_file = ::fdopen(fd, "a");
#else
    stdout_file = PyFile_AsFile(sysout);
#endif
```

**Note:**
Python I/O objects are buffered in Python 3.  I/O objects must be flushed before destruction.

## Compiled Extension Modules

Python 3 documentation for creating extension modules can be found [here](https://docs.python.org/3/extending/extending.html).

Use `PyModule_Create` instead of `Py_InitModule`:
```cpp
#if PY_MAJOR_VERSION >= 3
    PyObject *module = PyModule_Create(&module_def);
#else
    PyObject *module = Py_InitModule("mymodule", module_methods);
#endif
```

**Note:**
Unlike `Py_InitModule`, `PyModule_Create` creates the module object but does **not** register it with the global modules registry.  In Python 3, there is no way to explicitly register a module.  Python 3 implicitly registers the module object returned by the `PyInit_<modulename>` hook, making it impossible to create in-memory only modules.

## Macro Definition Conversions
- `RO` => `READONLY`, file mode

## Deprecation Warnings

`PyErr_WarnPy3k` no longer exists:
```cpp
#if PY_MAJOR_VERSION >= 3
    std::cerr << "X not supported in 3.x" << std::endl;
#else
    PyErr_WarnPy3k("file.softspace not supported in 3.x", 1)
#endif
```

## PyCObject API

Python 3 removed the `PyCObject` API.  Use the `PyCapsule` replacement API instead.

See the PyCObject to PyCapsule section in the [Python modernization documentation](https://py3c.readthedocs.io/en/latest/guide-modernization.html).

Examples:
- `PyCObject_FromVoidPtrAndDesc` => `PyCapsule_New`
- `PyCObject_Check` => `PyCapsule_CheckExact`

## Wide Character String Arguments

Several CPython functions, including `PySys_SetArgvEx` and `PyMain`, accept `wchar` string arguments instead of `char` in Python 3.

Example of passing program arguments, `argc` and `argv`, to Python:
```cpp
#if PY_MAJOR_VERSION >= 3
    // Allocate and initialize wchar arguments from argv.
    wchar_t** wargv = static_cast<wchar_t **>(PyMem_Malloc(sizeof(wchar_t*)*argc));
    for (int i=0; i<argc; i++)
    {
        wchar_t *warg = Py_DecodeLocale(argv[i], nullptr);
        wargv[i] = warg;
    }
    
    PySys_SetArgvEx(argc, wargv, update_path);
    
    // Free allocated wchar arguments.
    for (int i=0; i<argc; i++)
        PyMem_Free(wargv[i]);
    PyMem_Free(wargv);
#else
    PySys_SetArgvEx(argc, argv, update_path);
#endif
```

## Exception Handling
Exceptions fetched in C++ must be normalized in Python 3.

Example of printing an exception stack trace:
```cpp
    // Fetch last raised exception.
    PyObject *exception_type, *value, *traceback;
    PyErr_Fetch(&exception_type, &value, &traceback);

#if PY_MAJOR_VERSION >= 3
    PyErr_NormalizeException(&exception_type, &value, &traceback);
#endif

    // Fetch the exception stack trace.
    PyObject *traceback_module = PyImport_ImportModule("traceback");
    PyObject *tb_list = PyObject_CallMethod(
        traceback_module,
        "format_exception",
        "OOO",
        exception_type,
        value == nullptr ? Py_None() : value,
        traceback == nullptr ? Py_None() : traceback);
        
    // Print the stack trace.
    PyObject *empty_string = PyString_FromString("");
    PyObject *python_string = PyObject_CallMethod(empty_string, "join", "O", tb_list.ptr());
    std::cerr << PyString_AsString(python_string) << std::endl;


    // Cleanup.
    Py_XDECREF(python_string);
    Py_XDECREF(empty_string);
    Py_XDECREF(tb_list);
    Py_XDECREF(traceback_module);
    Py_XDECREF(exception_type);
    Py_XDECREF(value);
    Py_XDECREF(traceback);
```

**Note:**
`PyErr_NormalizeException` can change what object the `value` argument points to.  In that case, the function decrements the ref count of the old `value` object and increments the ref count of the new `value` object.

## Thread-safe GIL Checking

Use `PyGILState_Check` in Python 3:
```cpp
#if PY_MAJOR_VERSION >= 3
    bool thread_has_gil = PyGILState_Check();
#else
    // Note that this check is threadsafe.  If we have the GIL then a different
    // thread cannot acquire it, so the current Python thread state cannot
    // change.  If we don't have it, then even if another thread acquires or
    // releases it, _PyThreadState_Current will still not be set to our thread
    // state.
    PyThreadState *this_thread_state = PyGILState_GetThisThreadState();
    bool thread_has_gil = (this_thread_state && this_thread_state == _PyThreadState_Current());
#endif
```

## readline Module
- Python 3 changed the signature of the readline callback function.
- The readline callback function must return a string allocated with `PyMem_RawMalloc` in Python 3, otherwise, a crash will occur.

```cpp
#if PY_MAJOR_VERSION >= 3
static char *readLineFunction(FILE *input_file, FILE *output_file, const char *prompt)
{
    const char *some_line = "Hello World!";
    char *result = (char*)PyMem_RawMalloc(::strlen(some_line) + 1);
    ::strcpy(result, some_line);
    return result;
}
#else
static char *readLineFunction(FILE *input_file, FILE *output_file, char *prompt)
{
    const char *some_line = "Hello World!";
    char *result = (char*)PyMem_Malloc(::strlen(some_line) + 1);
    ::strcpy(result, some_line);
    return result;
}
#endif

void main(int argc, char *argv[])
{
    PyOS_ReadlineFunctionPointer() = readLineFunction;
    
    ...
    
    return 0;
}
```

## Building Python 3
- Use default `configure` options.

  The exception is on macOS.  Add the `--enable-framework` option.  Building Python as a framework matches the system Python distribution that comes with Mac.  However, building as a framework makes Python incompatible with other Python distributions on Mac (i.e. Mac Ports, Anaconda, etc.).
- On Linux and macOS, the default memory allocator is set to `pymalloc`.  This causes the build system to add the `m` suffix to various components in the distribution:
    - binary name => `python3.7m`
    - library name => `libpython3.7m.so.1.0`
    - compiled extension module names => `_ctypes.cpython-37m-x86_64-linux-gnu.so`
    - header include directory => `<prefix>/include/python3.7m`

  Windows does not use `pymalloc` and therefore does not have the `m` suffix.
