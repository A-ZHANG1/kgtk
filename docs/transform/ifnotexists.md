The ifnotexists command filters a KGTK file, passing through only those rows for
which one or more specified columns do not match records in a second KGTK file.

This implementation, in Python, builds an in-memory dictionary of the key
values in the --filter-on file.  Performance will be poor, and execution may
fail, if the --filter-on file is very large.  Alternatively, the --cache-input
option causes the code to cache the input file and not the --on-filter-file.

By default, input records are passed in order to the output file.  When                                                                                        
the input file is cached, the output records are order by key value (alpha                                                                                           
sort), then by input order.

The fields to match may be supplied by the user.  If not supplied, the
following defaults will be used.  "left" refers to the KFTK file being
filtered, and "right" refers to the file supplying the matching records.

| Left    | Right   | Key fields |
| ------- | ------- | ---------- |
| edge    | edge    | left.node1 == right.node1 and |
|         |         | left.label == right.label and |
|         |         | left.node2 == right.node2 |
| node    | node    | left.id    == right.id |
| edge    | node    | left.node1 == right.id |
| node    | edge    | right.id   == left.node1 |

## Usage

```bash
usage: kgtk ifnotexists [-h] [--input-keys [INPUT_KEYS [INPUT_KEYS ...]]] --filter-on
                        FILTER_KGTK_FILE [--filter-keys [FILTER_KEYS [FILTER_KEYS ...]]]
                        [-o OUTPUT_KGTK_FILE] [--cache-input [CACHE_INPUT]] [-v]
                        [input_kgtk_file]

Filter a KGTK file based on whether one or more records do not exist in a second KGTK file with matching values for one or more fields.

Additional options are shown in expert help.
kgtk --expert ifnotexists --help

positional arguments:
  input_kgtk_file       The KGTK file to filter. May be omitted or '-' for stdin.

optional arguments:
  -h, --help            show this help message and exit
  --input-keys [INPUT_KEYS [INPUT_KEYS ...]], --left-keys [INPUT_KEYS [INPUT_KEYS ...]]
                        The key columns in the file being filtered (default=None).
  --filter-on FILTER_KGTK_FILE
                        The KGTK file to filter against (required).
  --filter-keys [FILTER_KEYS [FILTER_KEYS ...]], --right-keys [FILTER_KEYS [FILTER_KEYS ...]]
                        The key columns in the filter-on file (default=None).
  -o OUTPUT_KGTK_FILE, --output-file OUTPUT_KGTK_FILE
                        The KGTK file to write (required),
  --cache-input [CACHE_INPUT]
                        Cache the input file instead of the filter keys (default=False).

  -v, --verbose         Print additional progress messages (default=False).
```

## Examples

Suppose that `file1.tsv` contains the following table in KGTK format:

| node1 | label   | node2 | location | years |
| ----- | ------- | ----- | -------- | ----- |
| john  | zipcode | 12345 | home     | 10    |
| john  | zipcode | 12346 |          |       |
| peter | zipcode | 12040 | home     |       |
| peter | zipcode | 12040 | work     | 6     |
| steve | zipcode | 45600 |          | 3     |
| steve | zipcode | 45601 | work     |       |

Suppose that `file2.tsv` contains the following table in KGTK format:

| node1 | label   | node2 |
| ----- | ------- | ----- |
| peter | zipcode | 12040 |

Suppose that `file3.tsv` contains the following table in KGTK format:

| id    |
| ----- |
| steve |
| john  |

Suppose that `file4.tsv` contains the following table in KGTK format:

| id    |
| ----- |
| peter |
| john  |

Suppose that `file5.tsv` contains the following table in KGTK format:

| id   |
| ---- |
| home |

```bash
kgtk ifnotexists file1.tsv --filter-on file2.tsv

```
| node1 | label   | node2 | location | years |
| ----- | ------- | ----- | -------- | ----- |
| john  | zipcode | 12345 | home     | 10    |
| john  | zipcode | 12346 |          |       |
| steve | zipcode | 45600 |          | 3     |
| steve | zipcode | 45601 | work     |       |


```bash
kgtk ifnotexists file1.tsv --filter-on file3.tsv

```
| node1 | label   | node2 | location | years |
| ----- | ------- | ----- | -------- | ----- |
| peter | zipcode | 12040 | home     |       |
| peter | zipcode | 12040 | work     | 6     |

```bash
kgtk ifnotexists file4.tsv --filter-on file3.tsv

```
| id    |
| ----- |
| peter  |

```bash
kgtk ifexists file1.tsv --filter-on file5.tsv --input-keys location

```
| node1 | label   | node2 | location | years |
| ----- | ------- | ----- | -------- | ----- |
| john  | zipcode | 12346 |          |       |
| peter | zipcode | 12040 | work     | 6     |
| steve | zipcode | 45600 |          | 3     |
| steve | zipcode | 45601 | work     |       |

