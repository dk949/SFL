# SFL

The Simple File Library.

## Important:

1.  In order for this library to work, put define `SFL_IMPL` in one (1) .cpp
    file **before** including this header.
2.  If you are unable to use `alloca`, define `SFL_NO_ALLOCA`, preferably for
    the whole project (e.g. by passing `-DSFL_NO_ALLOCA` to gcc and clang)
    * Note: If `alloca` is not available, `malloc` and `free` will be used. If
      you prefer you can define `sfl_alloca(size)` and `sfl_freea(pointer)` and
      they will be used instead. **You have to either define both or neither**
3.  If using `alloca`, SFL will avoid allocating a new string on the heap. You
    can define `SFL_MAX_STACK_STRING` (512 by default) to set the threshold when
    the string will be allocated.
4.  If a file cannot be closed, `SFL_FAILED_TO_CLOSE(ExpectedVoid)` will be
    called. By default it does nothing, but can be defined by the user.
5. This library uses
   [TartanLlama/expected](https://github.com/TartanLlama/expected) to indicate
   errors. It is already included in the file.

## Examples

### Read/write the whole file

``` cpp
{
    // Writing to a file
    auto e = File::writeFile("test.txt", "hello file\n");
    if (!e) {
        std::cerr << "Failed to write to the file\n";
        return 1;
    }
}

{
    // Reading from a file
    auto e = File::readFile("test.txt");
    if (!e) {
        std::cerr << "Failed to read the file\n";
        return 1;
    }
    std::cout << e.value() << '\n';
}
```

### Using `File` with no helper functions

``` cpp
{
    auto e = File::open("test.txt", File::Binary | File::Append);
    if (!e) {
        std::cerr << "Failed to open file in binary mode for appending\n";
    }
    if (!e->write("Appending to the file"sv)) {
        std::cerr << "Failed to write to file in binary mode for appending\n";
    }
}
```

## API

### `File` member functions

* `File(const File &) = delete;`
* `File &operator=(const File &) = delete;`
* `[[nodiscard]] ExpectedString read() noexcept;`
  * Returns the file as a string.
* `[[nodiscard]] ExpectedVoid write(const char *data, size_t size) noexcept;`
  * Writes `size` bytes of `data` to the file.
* `[[nodiscard]] ExpectedVoid write(std::string_view data, bool simple = false) noexcept;`
  * Writes all of `data` to the file.
  * If `simple` is true, the caller guarantees that the string\_view is pointing
    to a null-terminated string.
  * If `simple` is false and `data.size() > SFL_MAX_STACK_STRING` a new string
    will be created to fit the data.
* `[[nodiscard]] ExpectedVoid write(std::string_view data) noexcept;`
  * Writes all of `data` to the file.
* `[[nodiscard]] static ExpectedFile open(std::string_view filename, uint8_t state) noexcept;`
  * Opens a file and returns a `File` object for it, wrapped in `expected`.
* `[[nodiscard]] static ExpectedString readFile(std::string_view filename) noexcept;`
  * Return contents of a file as a string wrapped in `expected`.
* `[[nodiscard]] static ExpectedVoid writeFile(std::string_view filename, std::string_view data) noexcept;`
  * Write string to file.
* `~File() noexcept;`

### Modes

* Read
* Write
* Append
* Binary

These can be accessed as `File::[Mode]` and combined with the pipe (`|`)
operator.
