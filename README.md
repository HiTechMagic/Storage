# Storage
This is a set of .Net Standard libraries designed to provide access to file (and other) storage in a very generic _storage-agnostic_ way. This makes it available to both .Net Core and .Net Framework projects. Where specific providers cannot use .Net Standard multiple framework versions of libraries will be provided in the packages.

## Deployment Via NuGet
Deployment will be via a set of related NuGet packages, so that you only need to include the packages that you require (i.e.  just include the storage providers you wish to support in a given application).

## File specification
The Schema part of a stndard path name (e.g. drive letter in the old DOS parlence) is used to decide which Storage provider to use. 

This means all file access, regardless of actual storage, is via paths like:
- `TempFiles:\folder\subfolder\somefile.pdf` where `TempFiles` is specific named Storage configuration entry
- `UploadPath:\folder\subfolder\` where `UploadPath` is specific named Storage configuration entry

## Streams
To allow generic file access, Storage uses the lowest common denominator for any Storage Provider, which is the basic `Stream`. More complex operations, like `Copy`, are build on top of the basic Stream-driven methods.

## Configuration Driven
Configuration alone decides which Storage provider to use, for any given Schema name in a file path, so you can redirect you storage to a new location as/when you require without a code change.

## Features
- Copy from multiple sources, to multiple sources assuming a simple delimited set of paths. e.g. the following will copy all files from 2 sources to 2 destination paths storage.Copy("Temp1:\files\;Temp2:\files\", "Storage1:\files;Storage2:\files\"

## Initial design includes support for the following Storage environments
- Local file storage
- Embedded resource files (read only for obvious reasons)
- Azure blob storage

## Additional storage providers could include
- Email servers
- FTP & SFTP servers
- Inline Compression/Decompression (e.g. Zip/UnZip)

## Factory Pattern Decides Storage provider
In order to decide which Storage Provider library to implement against a given configuration Storge Type, it was felt a simple Factory pattern would be the simplest to implement while allowing granular inclusion of Storage libraries. A simple Factory class is created for each of:
- `IStorageProviderFactory`
- `IMaDataProviderFactory`
- `IEncryptionProviderFactory`
