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
