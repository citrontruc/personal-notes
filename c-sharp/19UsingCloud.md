# Azure Cloud

## Table of content

- [Azure Cloud](#azure-cloud)
  - [Table of content](#table-of-content)
  - [Storing documents in Azure Blob](#storing-documents-in-azure-blob)

## Storing documents in Azure Blob

Start with getting the appropriate dotnet packages:

```sh
dotnet add package Azure.Storage.Blobs
```

Add information to your appsettings.json file (only do that in development, for production, use Azure Key Vault):

```json
{
  "ConnectionStrings": {
    "AzureBlobStorage": "<YOUR_AZURE_BLOB_CONNECTION_STRING>"
  }
}
```

We can then write code to do our upload. During our upload, we can cut documents in chunks in order to optimize upload of large documents:

```cs
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Microsoft.Extensions.Options;
using UploadDownloadFilesAzureBlobStorage.Configuration;

namespace UploadDownloadFilesAzureBlobStorage.Services;

public sealed class AzureBlobStorageService : IBlobStorageService
{
    private readonly BlobContainerClient _containerClient;

    public AzureBlobStorageService(
        BlobServiceClient blobServiceClient, 
        IOptions<AzureBlobStorageConfiguration> options)
    {
        _containerClient = blobServiceClient.GetBlobContainerClient(options.Value.ContainerName);
    }

    public async Task<string> UploadSimpleAsync(
        string blobName, 
        Stream content, 
        string? contentType = null, 
        CancellationToken cancellationToken = default)
    {
        var blobClient = _containerClient.GetBlobClient(blobName);

        var uploadOptions = new BlobUploadOptions
        {
            HttpHeaders = new BlobHttpHeaders
            {
                ContentType = string.IsNullOrWhiteSpace(contentType) 
                    ? "application/octet-stream" 
                    : contentType
            }
        };

        await blobClient.UploadAsync(content, uploadOptions, cancellationToken);
        return blobClient.Name;
    }

        public async Task<string> UploadChunkedAsync(string blobName, Stream content, string? contentType = null,
        CancellationToken cancellationToken = default)
    {
        var blockBlobClient = _containerClient.GetBlockBlobClient(blobName);

        var blockIds = new List<string>();
        var buffer = ArrayPool<byte>.Shared.Rent(_uploadChunkSize);
        try
        {
            var index = 0;
            int bytesRead;
            
            while ((bytesRead = await content.ReadAsync(buffer.AsMemory(0, _uploadChunkSize), cancellationToken)) > 0)
            {
                var blockId = Convert.ToBase64String(BitConverter.GetBytes(index));
                using var blockStream = new MemoryStream(buffer, 0, bytesRead, writable: false, publiclyVisible: true);
                await blockBlobClient.StageBlockAsync(blockId, blockStream, cancellationToken: cancellationToken);
                blockIds.Add(blockId);
                index++;
            }

            var headers = new BlobHttpHeaders
            {
                ContentType = string.IsNullOrWhiteSpace(contentType) ? "application/octet-stream" : contentType
            };
            
            await blockBlobClient.CommitBlockListAsync(blockIds, httpHeaders: headers, cancellationToken: cancellationToken);
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(buffer);
        }

        return blockBlobClient.Name;
    }
}
```
