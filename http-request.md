# rest-request

## curl

- download a file with progress bar

```bash
filepath="/path/on/disk/"
url="https://github.com/my/repo/releases/download/id/archive.tar.gz"
curl \
  --location \
  --remote-header-name \
  --progress-bar \
  --output "${filepath}" \
  "${url}"
```

- only return status code

```bash
uri="localhost:9991/ping";
curl -o /dev/null -s -w "%{http_code}\n" "$uri"
```

- only headers

```bash
uri="http://awesome-site.com" ;
curl -v -s "${uri}" 1> /dev/null
```

## wget

- download a file with progress bar

```bash
filepath="/path/on/disk/"
url="https://github.com/my/repo/releases/download/id/archive.tar.gz"
wget \
  --quiet \
  --show-progress \
  --no-check-certificate \
  --output-document \
  "${filepath}" --content-disposition "${url}"
```

## aria2

- download a file with optimal flags

```bash
dir="/path/on/disk/"
file="archive.tar.gz"
url="https://github.com/my/repo/releases/download/id/archive.tar.gz"
aria2c \
  --continue \
  --optimize-concurrent-downloads \
  --min-split-size=1M \
  --max-concurrent-downloads=16 \
  --max-connection-per-server=16 \
  --file-allocation=falloc \
  --dir="${dir}" \
  --out="${file}" \
  "${url}"
```

## powershell

- download a file with progressbar

```powershell
$dir="/path/on/disk/" ;
$targetFile="archive.tar.gz" ;
$url="https://github.com/my/repo/releases/download/id/archive.tar.gz" ;

[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls"
$dir = Split-Path -parent $targetFile
if (-not(Test-Path -Path $dir -PathType Container)) {
  $null = New-Item -ItemType Directory -Path $dir -Force -ErrorAction Stop
}
$uri = New-Object "System.Uri" "$url"
$request = [System.Net.HttpWebRequest]::Create($uri)
$request.set_Timeout(15000)
$response = $request.GetResponse()
$totalLength = [System.Math]::Floor($response.get_ContentLength() / 1024)
$responseStream = $response.GetResponseStream()
$targetStream = New-Object `
  -TypeName System.IO.FileStream `
  -ArgumentList $targetFile, Create
$buffer = New-Object byte[] 10KB
$count = $responseStream.Read($buffer, 0, $buffer.length)
$downloadedBytes = $count
while ($count -gt 0) {
  $targetStream.Write($buffer, 0, $count)
  $count = $responseStream.Read($buffer, 0, $buffer.length)
  $downloadedBytes = $downloadedBytes + $count;
  $download_bytes_floor=([System.Math]::Floor($downloadedBytes / 1024))
  $status_msg="Downloaded ($($download_bytes_floor)K of $($totalLength)K): ";
  $percent_complete=(($download_bytes_floor / $totalLength) * 100)l
  Write-Progress `
    -activity "Downloading file '$($url.split('/') `
  | Select -Last 1)'" `
    -status $status_msg `
    -PercentComplete $percent_complete ;
}
Write-Progress `
  -activity "Finished downloading file '$($url.split('/') | Select -Last 1)'"
$targetStream.Flush()
$targetStream.Close()
$targetStream.Dispose()
$responseStream.Dispose()
```
