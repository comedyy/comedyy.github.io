## 数据压缩
1. 常用的是使用 `zipArchive`， 可以有目录结构。
```
// 压缩
bool CompressFileToZip(string fileName, string srcPath)
{
    try
    {
        if (!File.Exists(srcPath))
        {
            return false;
        }

        File.Delete(fileName);
        using (var compressedFileStream = new FileStream(fileName, FileMode.CreateNew))
        {
            //Create an archive and store the stream in memory.
            using (var zipArchive = new ZipArchive(compressedFileStream, ZipArchiveMode.Create, false)) {
                //Create a zip entry for each attachment
                var zipEntry = zipArchive.CreateEntry("proto", System.IO.Compression.CompressionLevel.Optimal);

                //Get the stream of the attachment
                using (var originalFileStream = new FileStream(srcPath, FileMode.Open))
                using (var zipEntryStream = zipEntry.Open()) {
                    //Copy the attachment stream to the zip entry stream
                    originalFileStream.CopyTo(zipEntryStream);
                }
            }
        }
    }
    catch(Exception e)
    {
        Debug.LogError($"压缩存档报错：{srcPath}\n{e.Message} \n{e.StackTrace}");
        return false;
    }
    finally
    {
        File.Delete(srcPath);
    }

    return true;
}

// 加载
byte[] LoadBytesFromZipOnDisk(string zipPath)
{
    try
    {
        using (var compressedFileStream = new FileStream(zipPath, FileMode.Open))
        {
            //Create an archive and store the stream in memory.
            using (var zipArchive = new ZipArchive(compressedFileStream, ZipArchiveMode.Read, false)) {
                //Create a zip entry for each attachment
                var zipEntry = zipArchive.GetEntry("proto");

                //Get the stream of the attachment
                using (var memorySteam = new MemoryStream())
                using (var zipEntryStream = zipEntry.Open()) {
                    //Copy the attachment stream to the zip entry stream
                    zipEntryStream.CopyTo(memorySteam);
                    return memorySteam.ToArray();
                }
            }
        }
    }
    catch(Exception e)
    {
        Debug.LogError($"读取存档失败 ：{zipPath}\n{e.Message} \n{e.StackTrace}");
        return null;
    }
}
```


2. 如果你只想要内存中压缩，使用`GZipStream`

```
public static byte[] Zip(string str) {
    var bytes = Encoding.UTF8.GetBytes(str);

    using (var msi = new MemoryStream(bytes))
    using (var mso = new MemoryStream()) {
        using (var gs = new GZipStream(mso, CompressionMode.Compress)) {
            msi.CopyTo(gs);
        }

        return mso.ToArray();
    }
}

public static string Unzip(byte[] bytes) {
    using (var msi = new MemoryStream(bytes))
    using (var mso = new MemoryStream()) {
        using (var gs = new GZipStream(msi, CompressionMode.Decompress)) {
            gs.CopyTo(mso);
        }

        return Encoding.UTF8.GetString(mso.ToArray());
    }
}
```