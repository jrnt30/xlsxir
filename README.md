# Xlsxir

[![Build Status](https://travis-ci.org/kennellroxco/xlsxir.svg?branch=master)](https://travis-ci.org/kennellroxco/xlsxir)
[![Hex.pm Version](http://img.shields.io/hexpm/v/xlsxir.svg)](https://hex.pm/packages/xlsxir)
[![Hex docs](http://img.shields.io/badge/hex.pm-docs-blue.svg?style=flat)](https://hexdocs.pm/xlsxir)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/kennellroxco/xlsxir/master/LICENSE)

Xlsxir is an Elixir library that parses `.xlsx` files using Simple API for XML (SAX) parsing via the [Erlsom](https://github.com/willemdj/erlsom) Erlang library, extracts the data to an Erlang Term Storage (ETS) process and provides various functions for accessing the data. Xlsxir supports ISO 8601 date formats and large files. 

Testing has been limited to various documents in Microsoft Excel and LibreOffice as well as any issues submitted through GitHub. Only English and Portuguese languages have been tested. A large worksheet containing 100 columns and 514K rows has successfully been parsed. Please submit any issues found and they will be addressed ASAP.

## Installation

You can add Xlsxir as a dependancy to your Elixir project via the Hex package manager by adding the following to your `mix.exs` file: 

```elixir
def deps do
  [ {:xlsxir, "~> 1.3.3"} ]
end
```

Or, you can directly reference the GitHub repo:

```elixir
def deps do
  [ {:xlsxir, github: "kennellroxco/xlsxir"} ]
end
```

## Basic Usage

Xlsxir parses a `.xlsx` file located at a given `path` and extracts the data to an ETS process via the `Xlsxir.extract/3` and `Xlsxir.multi_extract/3` functions:

```elixir
Xlsxir.extract(path, index, timer \\ false)
Xlsxir.multi_extract(path, index, timer \\ false)
```

The `multi_extract/3` function allows multiple worksheets to be parsed by creating a separate ETS process for each worksheet and returning a unique table identifier for each.

Argument descriptions:
- `path` the path of the file to be parsed in `string` format
- `index` is the position of the worksheet you wish to parse (zero-based index)
- `timer` is a boolean flag that controls an extraction timer that returns time elapsed when set to `true`. Defalut value is `false`.

Upon successful completion, the extraction process returns: 
- for `extract/3`:
    * `:ok` with `timer` set to `false`
    * `{:ok, time_elapsed}` with `timer` set to `true`
- for `multi_extract/3`:
    * `{:ok, table_id}` with `timer` set to `false`
    * `{:ok, table_id, time_elapsed}` with `timer` set to `true`

<br/>
The extracted worksheet data can be accessed using any of the following functions:
```elixir
Xlsxir.get_list(table_id)
Xlsxir.get_map(table_id)
Xlsxir.get_mda(table_id)
Xlsxir.get_cell(table_id, cell_ref)
Xlsxir.get_row(table_id, row_num)
Xlsxir.get_col(table_id, col_ltr)
Xlsxir.get_info(table_id, num_type)
```
**Note:** `table_id` defaults to `:worksheet` and is therefore not required when using `Xlsxir.extract/3` to parse a given worksheet. The `table_id` parameter is only used with `Xlsxir.multi_extract/3`.

`Xlsxir.get_list/1` returns entire worksheet in a list of row lists (i.e. `[[row 1 values], ...]`)<br/>
`Xlsxir.get_map/1` returns entire worksheet in a map of cell names and values (i.e. `%{"A1" => value, ...}`)<br/>
`Xlsxir.get_mda/1` returns entire worksheet in an indexed map which can be accessed like a multi-dimensional array (i.e. `some_var[0][0]` for cell "A1")<br/>
`Xlsxir.get_cell/2` returns value of specified cell (i.e. `"A1"` returns value contained in cell A1)<br/>
`Xlsxir.get_row/2` returns values of specified row (i.e. `1` returns the first row of data)<br/>
`Xlsxir.get_col/2` returns values of specified column (i.e. `"A"` returns the first column of data)<br/>
`Xlsxir.get_info/1` and `Xlsxir.get_multi_info/2` return count data for `num_type` specified (i.e. `:rows`, `:cols`, `:cells`, `:all`)<br/>

Once the table data is no longer needed, run the following function to delete the ETS process and free memory:
```elixir
Xlsxir.close(table_id) 
```
When using `Xlsxir.extract/3`, be sure to [close an open ETS process before trying to parse another worksheet](https://hexdocs.pm/xlsxir/Xlsxir.html#close/0) in the same session or process. If you try to open a new `:worksheet` ETS process when one already exists, you will get an error. If the parsing of multiple worksheets is desired, use `Xlsxir.multi_extract/3` instead.

Refer to [Xlsxir documentation](https://hexdocs.pm/xlsxir/index.html) for more detailed examples. 

## Considerations

Cell references are formatted as a string (i.e. "A1"). Strings will be returned as type `string`, resulting values for functions from within the worksheet are returned as type `string`, `integer` or `float` depending on the type of the resulting value, data formatted as a number in the worksheet will be returned as type `integer` or `float`, and ISO 8601 date formatted values will be returned in Erlang `:calendar.date()` type format (i.e. `{year, month, day}`). Xlsxir does not currently support dates prior to 1/1/1900.

## Planned Development

- Additional performance improvement for larger files
- Adding time support for dates (i.e. {{YYYY, MM, DD}, {h, m, s}})
- Export functionality to .xlsx file type with formatting options
- Implement Elixir 1.3 calendar datatypes support

## Contributing

Contributions are encouraged. Feel free to fork the repo, add your code along with appropriate tests and documentation (ensuring all existing tests continue to pass) and submit a pull request. 

## Bug Reporting

Please report any bugs or request future enhancements via the [Issues](https://github.com/kennellroxco/xlsxir/issues) page. 

## Acknowledgements

I'd like to thank the following people who were a big help in the development of this library:

- Paulo Almeida (@pma) was a big help with testing and has provided several great ideas for development.
- Benjamin Tan's (@benjamintanweihao) article on [SAX parsing with Elrsom](http://benjamintan.io/blog/2014/10/01/parsing-wikipedia-xml-dump-in-elixir-using-erlsom/) was invaluable.
- Daniel Berkompas' (@danielberkompas) article [Multidimensional Arrays in Elixir](http://blog.danielberkompas.com/2016/04/23/multidimensional-arrays-in-elixir.html?utm_campaign=elixir_radar_48&utm_medium=email&utm_source=RD+Station) inspired `Xlsxir.get_mda/0`.
                           
