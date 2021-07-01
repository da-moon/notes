# rest-request

## curl

- download a file with progress bar

```bash
filepath="/path/on/disk/"
url="https://github.com/my/repo/releases/download/id/archive.tar.gz"
curl -LJO --progress-bar -o "${filepath}" "${url}"
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
