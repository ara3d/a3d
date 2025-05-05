# A3D and A3DZ

A3D is a compact and streamable 3D geometry format built for large architectural and infrastructure models.
It is designed for:

- Fast loading of massive models 
- Minimal parsing and memory overhead
- Easy implementation and validation
- Compatibility with modern GPU pipelines

A3DZ is an encoded and compressed version of the A3D format optimized for transmission.    

# Motivation 

Architectural and infrastructure models can be extremely large containing upwards of a million objects,
and significant amounts of repetition of meshes. Handling models of this size efficiently requires tradeoffs.

The A3D format is designed to for efficient GPU-based instancing and to be as compact as possible 
for common architectural and engineering workflows. 

# Specification

The A3D file format is a collection of named buffers. Each name contains three components:

1. Association - What part of the scene the data is associated with 
2. Semantic - The role of the buffer
3. Data type - The type of each element in the buffer 

## Association 

Each buffer associates elements with either of the following:

- Instance
- Mesh
- Vertex
- Corner

## Names

- `Instance:Position:Vector3`  
- `Instance:Rotation:Quaternion` 
- `Instance:Scale:Vector3` 
- `Instance:Color:UInt32`
- `Instance:Metallic:Byte`
- `Instance:Roughness:Byte`
- `Instance:ID:UUID`
- `Mesh:IndexCount:UInt32`
- `Mesh:VertexOffset:UInt32`
- `Mesh:InstanceCount:UInt32`
- `Mesh:InstanceIndex:UInt32`
- `Vertex:X:Float32`
- `Vertex:Y:Float32`
- `Vertex:Z:Float32`
- `Corner:Index:UInt32`

## Data Types

- UInt32 - 32 bit unsigned integer - 4 bytes 
- Vector3 - 3 single precision floats - 12 bytes
- Quaternion - 4 single precision floats - 16 bytes 
- UUID - 128 bit universally unique id - 16 bytes
- Byte - 8 bit unsigned integer - 1 byte
- Float32 - 1 single precision float - 4 bytes

# Binary Layout 

The binary layout of the A3D buffers follows the [BFAST specification](https://github.com/vimaec/bfast).

The file format consists of three sections:

* Header - Fixed size descriptor (32 bytes) describing the file contents   
* Ranges - An array of offset pairs indicating the begin and end of each buffer (relative to file begin) 
* Data   - 64-byte aligned data buffers 

## Header Section

The header is a 32-byte struct with the following layout:  

```
    [StructLayout(LayoutKind.Explicit, Pack = 8, Size = 32)]
    public struct Header
    {
        [FieldOffset(0)]    public long Magic;         // 0xBFA5
        [FieldOffset(8)]    public long DataStart;     // <= File size and >= 32 + Sizeof(Range) * NumArrays 
        [FieldOffset(16)]   public long DataEnd;       // >= DataStart and <= file size
        [FieldOffset(24)]   public long NumArrays;     // Number of all buffers, including name buffer
    }
```

## Ranges Section

The ranges start at byte 32. There are `NumArrays` of them and they have the following format. 
`NumArrays` is the total count of all buffers, including the first buffer that contains the names.
`NumArrays` should always be equal to or greater than one. Each `Begin` and `End` values are byte 
offsets relative to the beginning of the file.

```
    [StructLayout(LayoutKind.Explicit, Pack = 8, Size = 16)]
    public struct Range
    {
        [FieldOffset(0)] public long Begin;
        [FieldOffset(8)] public long End;
    }		
```

## Data Section

The data section starts at the first 64 byte aligned address immediately following the last `Range` value.
This value is stored for validation purposes in the header as `DataStart`. 

### Names Buffer

The first data buffer contain the names of the subsequent buffers as a concatenated list of Utf-8 encoded 
strings separated by null characters. Names may be zero-length and are not guaranteed to be unique. 
A name may contain any Utf-8 encoded character except the null character. 

There must be N-1 names where N is the number of ranges (i.e. the `NumArrays` value in header). 

## A3DZ

The A3DZ format follows the same layout as an A3D but each attribute is compressed using a combination of techniques.
