# Storage
Storage is a set of .Net Standard libraries designed to provide access to file (and other) storage in a very generic _storage-agnostic_ way. 

Imagine a world where the repetative act of copying, moving, opening files was as simple as one line of code. After decades of writing custom code for file access the idea arose of making a generic system, that could hide all the mechanics of where your files reside, and just use standard file operations for everything (Read, Write, Copy etc) regardless of whether it was stored on a local drive or hosted in the cloud. It also opens up the ability to process multiple source and/or destination paths in a single command. e.g.

`storage.CopyAsync("Templates:\Masterfile.pdf;LocalFileStore:\someFolder\;SecondStore:\anotherFolder\", "TargetStore:\folder\folder")`

## .Net Standard by default
Using .Net Standard makes the librares available to both .Net Core and .Net Framework projects* and on most environments using the exact same code (e.g. Blob storage, Local file systems, Embedded resources and even Email systems _eventually_). 

*Note: Where specific providers cannot use .Net Standard multiple framework versions of libraries will be provided in the packages.

## Typical file operations supported:
- Copy files
- Move files
- Rename files (where allowed)
- Delete files (where allowed)
- Read files
- Write files (where allowed)
- Get file information (i.e. metadata)
- Get contents of a folder

## Deployment Via NuGet
Deployment will be via a set of related NuGet packages, so that you only need to include the packages that you require (i.e. just include the storage providers you wish to support in a given application).

## File specification
`{schema}:{relativePath}\{filespec}`

The Schema part of a standard file path name (e.g. drive letter in the old DOS parlence) is used to decide which Storage provider to use.
This means all file access, regardless of actual storage, is via paths like:
- `TempFiles:\folder\subfolder\somefile.pdf` where `TempFiles` is specific named Storage configuration entry
- `UploadPath:\folder\subfolder\`* where `UploadPath` is specific named Storage configuration entry

*Note: A trailing \ (or /) is used to identify a directory/folder avoiding the ambiguity that can occur (especially with providers like Azure Blob Storage).

## Streams
To allow generic file access, Storage uses the lowest common denominator for any Storage Provider, which is the basic `Stream`. More complex operations, like `Copy`, are build on top of the basic Stream-driven methods.

## Metadata
Metadata is additional information associated with a given file. This can include useful information like the original filename, the filename for downloading, the file size or the file Metadat type. In storage providers like Azure Blob Storage this information can be stored against the file Blob directly as a set if key-value pairs. To support Metadata on all Storage providers, Storage allows for having  metadata in alternate storage (e.g. database entries or even parallel files with a JSON extension).

## Configuration Driven
Configuration alone decides which Storage provider to use, for any given Schema name in a file path, so you can redirect you storage to a new location as/when you require without a code change.

## Features
### Read
`IStorage.ReadAsync(sourcePath)`

### Copy
`IStorage.CopyAsync(sourcePaths, destinationPaths)`

- Copy from multiple sources, to multiple sources assuming a simple delimited set of paths. e.g. the following will copy all files from 2 sources to 2 destination paths `await storage.CopyAsync("Temp1:\files\;Temp2:\files\", "Storage1:\files;Storage2:\files\");`

### Move
`IStorage.MoveAsync(sourcePaths, destinationPaths)`

- This is implemented by the high-level code as calls to Read, write then delete.
- Where a provider supports deletion, the source files will be deleted after successful copying.

## Read
`IStorage.ReadAsync(fileOrFolderPath)`

## Write
`IStorage.RenameAsync(newFileOrFolderPath, stream [, metadata])`

### Rename
`IStorage.RenameAsync(fileOrFolderPath, newFileOrFolderName)`

### Delete
`IStorage.DeleteAsync(fileOrFolderPaths)`

Where the provider supports deletion, a simple call to await `storage.DeleteAsync()` will remove the file(s) so

## Initial design includes support for the following Storage environments
- Local file storage
- Embedded resource files (readonly for obvious reasons)
- Azure blob storage

## Additional storage providers could include
- Email servers
- FTP & SFTP servers
- Inline Compression/Decompression (e.g. Zip/UnZip)

## Factory Pattern Decides Storage provider
In order to decide which Storage Provider library to implement against a given configuration Storge Type, it was felt a simple Factory pattern would be the simplest to implement while allowing granular inclusion of Storage libraries. A simple Factory class is created for each of:
- `IStorageProviderFactory`
- `IMetadataProviderFactory`
- `IEncryptionProviderFactory`

 As the slowest part of file operations (by orders of magnitude) is the actual file transfer, the overhead of looking up and generating providers on-the-fly, via the Class Factories, is minimal and not considered an issue at this time. 
