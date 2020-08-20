# GameMaker Studio `ds_map` serialization format

When saving a `ds_map` object using `ds_map_write`, it gets serialized to a string. I found no documentation anywhere about the format of this string, so I decided to reverse engineer it.

## String format

The string consists of hexadecimal numbers representing an array of bytes. For example, `"CAFEBABE"` would represent the array `[0xCA, 0xFE, 0xBA, 0xBE]`.

This array of bytes represents data in a custom format.

## Data format

All serialized `ds_map` objects follow the structure below:

| Data | Size | Type |
|:-:|:-:|:-:|
| Magic number | 4 bytes | Integer (402) |
| Number of entries | 4 bytes | Integer |
| Entry 1 | Variable | Entry |
| Entry 2 | Variable | Entry |
| ... |||
| Entry n | Variable | Entry |



### Entry format

Each entry follows the format below:

| Data | Size | Type |
|:-:|:-:|:-:|
| Key | Variable | Object |
| Value | Variable | Object |

### Object format

There are two types of objects: numbers and strings.

#### Number format

| Data | Size | Type |
|:-:|:-:|:-:|
| Type | 4 bytes | Integer (0) |
| Value | 8 bytes | IEEE754 Double precision |

#### String format

| Data | Size | Type |
|:-:|:-:|:-:|
| Type | 4 bytes | Integer (1) |
| Length | 4 bytes | Integer |
| Value | Variable | ASCII string |

### Endianness

All integers and doubles are stored in little-endian order.

## Example

Suppose we serialize the following `ds_map`:

```gml
map = ds_map_create();
ds_map_add(map, "random", 4);
ds_map_add(map, "universe", 42);
ds_map_add(map, 3.14, "pi");
show_debug_message(ds_map_write(map));
```

The output is the following:

```
9201000003000000010000000600000072616E646F6D000000000000000000001040000000001F85EB51B81E0940010000000200000070690100000008000000756E697665727365000000000000000000004540
```

Now, let's parse the bytes following the format specification previously described.

#### Header
| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Magic number | 92 01 00 00 | 402 |
| Number of entries | 03 00 00 00 | 3 |

#### Entry 1

##### Key

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 01 00 00 00 | 1 (string) |
| Length | 06 00 00 00 | 6 |
| Value | 72 61 6E 64 6F 6D | random |

##### Value

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 00 00 00 00 | 0 (number) |
| Value | 00 00 00 00 00 00 10 40 | 4 |

#### Entry 2

##### Key

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 00 00 00 00 | 0 (number) |
| Value | 1F 85 EB 51 B8 1E 09 40 | 3.14 |

##### Value

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 01 00 00 00 | 1 (string) |
| Length | 02 00 00 00 | 2 |
| Value | 70 69 | pi |

#### Entry 3

##### Key

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 01 00 00 00 | 1 (string) |
| Length | 08 00 00 00 | 8 |
| Value | 75 6E 69 76 65 72 73 65 | universe |

##### Value

| Data | Raw bytes | Value |
|:-:|:-:|:-:|
| Type | 00 00 00 00 | 0 (number) |
| Value | 00 00 00 00 00 00 45 40 | 42 |

As you can see, the original `ds_map` has been correctly deserialized!
