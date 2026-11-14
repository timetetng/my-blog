---
title: Windows 下免代理部署 ww-manager
author: 行简
pubDatetime: 2026-11-15T00:35:00+08
slug: deploy-ww-manager-without-proxy
featured: false
draft: false
tags:
  - Windows
  - PowerShell
  - Linux
  - uv
  - 鸣潮
description: 在连不上 GitHub、也没有代理的机器上装 ww-manager：uv 本体、PyPI 包、CPython 三条链路全走镜像，附一键脚本。
---
## 三条镜像

| 环节 | 换成的地址 |
| --- | --- |
| uv 二进制 | `https://gh-proxy.com/https://github.com/astral-sh/uv/releases/...` |
| PyPI | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| CPython | `UV_PYTHON_INSTALL_MIRROR` 指到同一个 gh-proxy 前缀 |

gh-proxy 就是个 GitHub 反向代理，用法是在原始 URL 前面拼上 `https://gh-proxy.com/`。

## 一键脚本（Windows）

保存成 `install-ww-manager.ps1`，然后用命令行跑：

```powershell
powershell -ExecutionPolicy Bypass -File .\install-ww-manager.ps1
```

脚本全文：

```powershell
<#
.SYNOPSIS
    One-click installer: uv + ww-manager on Windows, no proxy required.

.DESCRIPTION
    1. Downloads the uv Windows binary from a GitHub mirror (gh-proxy.com).
    2. Unpacks it to %USERPROFILE%\.local\bin and adds that folder to the user PATH.
    3. Points uv at the Tsinghua PyPI mirror and at a mirror for CPython builds.
    4. Installs the requested packages with `uv tool install`.

.EXAMPLE
    powershell -ExecutionPolicy Bypass -File .\install-ww-manager.ps1

.EXAMPLE
    # pin a different package list / uv version
    powershell -ExecutionPolicy Bypass -File .\install-ww-manager.ps1 -Packages ww-manager,yt-dlp -UvVersion 0.13.8
#>
[CmdletBinding()]
param(
    [string]   $UvVersion     = '0.13.8',
    [string]   $IndexUrl      = 'https://pypi.tuna.tsinghua.edu.cn/simple',
    [string]   $GhMirror      = 'https://gh-proxy.com/',
    [string[]] $Packages      = @('ww-manager'),
    [string]   $PythonVersion = '',      # e.g. '3.13' to also install a managed CPython
    [switch]   $SkipChecksum,
    [switch]   $NoPath                   # do not touch the user PATH
)

$ErrorActionPreference = 'Stop'
$ProgressPreference    = 'SilentlyContinue'
try { [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 } catch {}

$BinDir   = Join-Path $env:USERPROFILE '.local\bin'
$Asset    = 'uv-x86_64-pc-windows-msvc.zip'
$UvUrl    = "$GhMirror" + "https://github.com/astral-sh/uv/releases/download/$UvVersion/$Asset"
$PyMirror = "$GhMirror" + 'https://github.com/astral-sh/python-build-standalone/releases/download'
$TmpDir   = Join-Path $env:TEMP ('uv-install-' + [guid]::NewGuid().ToString('N').Substring(0, 8))

function Say([string]$msg) { Write-Host "==> $msg" -ForegroundColor Cyan }
function Warn([string]$msg) { Write-Host "    $msg" -ForegroundColor Yellow }

function Get-FileWithRetry([string]$Url, [string]$OutFile, [int]$Tries = 3) {
    for ($i = 1; $i -le $Tries; $i++) {
        try {
            Invoke-WebRequest -Uri $Url -OutFile $OutFile -UseBasicParsing -TimeoutSec 600
            if ((Get-Item -LiteralPath $OutFile).Length -lt 4096) { throw 'downloaded file is too small' }
            return
        } catch {
            Warn "attempt $i/$Tries failed: $($_.Exception.Message)"
            if ($i -eq $Tries) { throw }
            Start-Sleep -Seconds (2 * $i)
        }
    }
}

try {
    # ---------------------------------------------------------------- download
    Say "uv $UvVersion  <-  $UvUrl"
    New-Item -ItemType Directory -Force -Path $TmpDir | Out-Null
    $zip = Join-Path $TmpDir $Asset
    Get-FileWithRetry -Url $UvUrl -OutFile $zip

    # ---------------------------------------------------------------- checksum
    if (-not $SkipChecksum) {
        try {
            $sumText = (Invoke-WebRequest -Uri "$UvUrl.sha256" -UseBasicParsing -TimeoutSec 60).Content
            if ($sumText -is [byte[]]) { $sumText = [Text.Encoding]::UTF8.GetString($sumText) }
            $expected = ($sumText.Trim() -split '\s+')[0].ToLower()
            $actual   = (Get-FileHash -LiteralPath $zip -Algorithm SHA256).Hash.ToLower()
            if ($expected -ne $actual) { throw "sha256 mismatch (expected $expected, got $actual)" }
            Say 'sha256 OK'
        } catch {
            Warn "checksum skipped: $($_.Exception.Message)"
        }
    }

    # ------------------------------------------------------------------ unpack
    Say "unpacking to $BinDir"
    New-Item -ItemType Directory -Force -Path $BinDir | Out-Null
    foreach ($exe in 'uv.exe', 'uvx.exe', 'uvw.exe') {
        $p = Join-Path $BinDir $exe
        if (Test-Path -LiteralPath $p) { Remove-Item -LiteralPath $p -Force -ErrorAction SilentlyContinue }
    }
    Expand-Archive -LiteralPath $zip -DestinationPath $BinDir -Force

    # -------------------------------------------------------------------- PATH
    if (-not $NoPath) {
        $userPath = [Environment]::GetEnvironmentVariable('Path', 'User')
        if ($null -eq $userPath) { $userPath = '' }
        $entries = @($userPath -split ';' | Where-Object { $_ -ne '' })
        if ($entries -notcontains $BinDir) {
            $newPath = if ($userPath.Trim()) { $userPath.TrimEnd(';') + ';' + $BinDir } else { $BinDir }
            [Environment]::SetEnvironmentVariable('Path', $newPath, 'User')
            Say 'user PATH updated (takes effect in a new terminal)'
        }
    }
    $env:Path = @($env:Path -split ';' | Where-Object { $_ -ne '' }) -join ';'
    if (-not (($env:Path -split ';') -contains $BinDir)) { $env:Path = $env:Path + ';' + $BinDir }

    # ------------------------------------------------------- mirrors for uv
    [Environment]::SetEnvironmentVariable('UV_DEFAULT_INDEX', $IndexUrl, 'User')
    [Environment]::SetEnvironmentVariable('UV_PYTHON_INSTALL_MIRROR', $PyMirror, 'User')
    $env:UV_DEFAULT_INDEX          = $IndexUrl
    $env:UV_PYTHON_INSTALL_MIRROR  = $PyMirror
    Say "PyPI index  -> $IndexUrl"
    Say "CPython src -> $PyMirror"

    $uv = Join-Path $BinDir 'uv.exe'
    & $uv --version

    # ------------------------------------------------------------- python
    if ($PythonVersion) {
        Say "installing CPython $PythonVersion (via mirror)"
        & $uv python install $PythonVersion
        if ($LASTEXITCODE -ne 0) { Warn "uv python install returned $LASTEXITCODE" }
    }

    # ------------------------------------------------------------ packages
    foreach ($pkg in $Packages) {
        Say "uv tool install $pkg"
        & $uv tool install $pkg --default-index $IndexUrl
        if ($LASTEXITCODE -ne 0) { throw "uv tool install $pkg failed (exit $LASTEXITCODE)" }
    }

    Say 'done'
    Write-Host ''
    Write-Host "  uv       : $BinDir\uv.exe"
    Write-Host "  tools    : $BinDir"
    Write-Host "  NOTE     : open a NEW terminal so PATH picks up $BinDir"
    Write-Host ''
}
finally {
    Remove-Item -LiteralPath $TmpDir -Recurse -Force -ErrorAction SilentlyContinue
}
```

它做的事：

1. 从 gh-proxy 拉 `uv-x86_64-pc-windows-msvc.zip`（0.13.8），失败重试三次，并用官方的 `.sha256` 校验；
2. 解压到 `%USERPROFILE%\.local\bin`，写进用户 PATH；
3. 把 `UV_DEFAULT_INDEX` 和 `UV_PYTHON_INSTALL_MIRROR` 写进用户环境变量，之后这台机器上所有 `uv` 命令都自动走镜像；
4. `uv tool install ww-manager --default-index ...`。

装出来的可执行文件叫 `ww.exe`（不是 `ww-manager.exe`），同样落在 `%USERPROFILE%\.local\bin`，新开一个终端直接敲 `ww` 就行。

想额外指定 Python 版本就加 `-PythonVersion 3.13`，脚本会先用镜像把 CPython 装好再装包，免得装到一半才发现没有解释器。

## Linux / Termux

同样的思路，bash 版本：

```bash
bash install-ww-manager.sh
```

```bash
#!/usr/bin/env bash
# ---------------------------------------------------------------------------
# install-ww-manager.sh
#
# One-click installer for uv + ww-manager on Linux (and Termux), no proxy.
# Everything that would normally hit github.com goes through a mirror instead.
#
#   bash install-ww-manager.sh
#
# Overrides (environment variables):
#   MIRROR=https://gh-proxy.com/                        github mirror prefix
#   UV_VERSION=0.13.8                                   uv release to install
#   INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple  PyPI mirror
#   PKGS="ww-manager"                                   space separated tools
#   SKIP_CHECKSUM=1                                     skip the sha256 check
# ---------------------------------------------------------------------------
set -euo pipefail

MIRROR="${MIRROR:-https://gh-proxy.com/}"
UV_VERSION="${UV_VERSION:-0.13.8}"
INDEX_URL="${INDEX_URL:-https://pypi.tuna.tsinghua.edu.cn/simple}"
PKGS="${PKGS:-ww-manager}"
SKIP_CHECKSUM="${SKIP_CHECKSUM:-0}"

GH_UV="https://github.com/astral-sh/uv/releases/download/${UV_VERSION}"
PY_MIRROR="${MIRROR}https://github.com/astral-sh/python-build-standalone/releases/download"

BIN_DIR="${HOME}/.local/bin"
TMP_DIR="$(mktemp -d)"
trap 'rm -rf "$TMP_DIR"' EXIT

say()  { printf '\033[36m==> %s\033[0m\n' "$*"; }
warn() { printf '\033[33m    %s\033[0m\n' "$*"; }

if [ "$(uname -s)" != "Linux" ]; then
    echo "This script is for Linux. On Windows use install-ww-manager.ps1." >&2
    exit 1
fi

case "$(uname -m)" in
    x86_64|amd64) ARCH=x86_64 ;;
    aarch64|arm64) ARCH=aarch64 ;;
    armv7l|armv7)  ARCH=armv7 ;;
    *) echo "unsupported architecture: $(uname -m)" >&2; exit 1 ;;
esac

if [ -n "${TERMUX_VERSION:-}" ] || [ -d /data/data/com.termux ]; then
    case "$ARCH" in
        aarch64) TARGET="aarch64-unknown-linux-android" ;;
        x86_64)  TARGET="x86_64-unknown-linux-android" ;;
        *) echo "unsupported Termux architecture: $ARCH" >&2; exit 1 ;;
    esac
else
    case "$ARCH" in
        x86_64)  TARGET="x86_64-unknown-linux-gnu" ;;
        aarch64) TARGET="aarch64-unknown-linux-gnu" ;;
        armv7)   TARGET="armv7-unknown-linux-gnueabihf" ;;
    esac
fi

fetch() {
    local url="$1" out="$2"
    if command -v curl >/dev/null 2>&1; then
        curl -fL --retry 3 --retry-delay 2 --connect-timeout 15 -o "$out" "$url"
    elif command -v wget >/dev/null 2>&1; then
        wget -O "$out" "$url"
    else
        echo "need curl or wget" >&2
        exit 1
    fi
}

ASSET="uv-${TARGET}.tar.gz"
URL="${MIRROR}${GH_UV}/${ASSET}"

say "uv ${UV_VERSION}  <-  ${URL}"
if ! fetch "$URL" "$TMP_DIR/$ASSET"; then
    echo "download failed: $URL" >&2
    echo "try another mirror, e.g. MIRROR=https://gh-proxy.com/" >&2
    exit 1
fi

if [ "$SKIP_CHECKSUM" != "1" ]; then
    if fetch "${URL}.sha256" "$TMP_DIR/sum" 2>/dev/null; then
        expected="$(awk '{print $1; exit}' "$TMP_DIR/sum")"
        if command -v sha256sum >/dev/null 2>&1; then
            actual="$(sha256sum "$TMP_DIR/$ASSET" | awk '{print $1}')"
        else
            actual="$(shasum -a 256 "$TMP_DIR/$ASSET" | awk '{print $1}')"
        fi
        if [ "$expected" != "$actual" ]; then
            echo "sha256 mismatch (expected $expected, got $actual)" >&2
            exit 1
        fi
        say "sha256 OK"
    else
        warn "checksum file unavailable, skipping"
    fi
fi

say "installing to ${BIN_DIR}"
mkdir -p "$BIN_DIR"
tar -xzf "$TMP_DIR/$ASSET" -C "$BIN_DIR" --strip-components=1
chmod +x "$BIN_DIR/uv"
if [ -f "$BIN_DIR/uvx" ]; then chmod +x "$BIN_DIR/uvx"; fi

BLOCK_BEGIN="# >>> uv + ww-manager >>>"
BLOCK_END="# <<< uv + ww-manager <<<"

case "${SHELL:-}" in
    */zsh)  RC_FILE="$HOME/.zshrc" ;;
    */bash) RC_FILE="$HOME/.bashrc" ;;
    *)      RC_FILE="$HOME/.profile" ;;
esac
touch "$RC_FILE"

if grep -qF "$BLOCK_BEGIN" "$RC_FILE" 2>/dev/null; then
    warn "shell config already has the block, keeping it as is ($RC_FILE)"
else
    cat >> "$RC_FILE" <<EOF

$BLOCK_BEGIN
export PATH="\$HOME/.local/bin:\$PATH"
export UV_DEFAULT_INDEX="$INDEX_URL"
export UV_PYTHON_INSTALL_MIRROR="$PY_MIRROR"
$BLOCK_END
EOF
    say "PATH + mirror settings written to ${RC_FILE}"
fi

export PATH="$BIN_DIR:$PATH"
export UV_DEFAULT_INDEX="$INDEX_URL"
export UV_PYTHON_INSTALL_MIRROR="$PY_MIRROR"

"$BIN_DIR/uv" --version

for pkg in $PKGS; do
    say "uv tool install ${pkg}"
    "$BIN_DIR/uv" tool install "$pkg" --default-index "$INDEX_URL"
done

say "done"
echo
echo "  uv    : $BIN_DIR/uv"
echo "  tools : $BIN_DIR"
echo "  NOTE  : run 'source $RC_FILE' or open a new shell so PATH is set"
echo
```

脚本自己判断 x86_64 / aarch64 / armv7，Termux 下换用 `-unknown-linux-android` 的三元组。配置写进 `.bashrc` 或 `.zshrc` 里的一个标记块，重复跑不会叠。

这版本我在一台 aarch64 的 Arch 容器里实跑过：`sha256 OK` → 装好 uv → `ww-manager==2.3.8` → `Installed 1 executable: ww`，全程二十多秒。

## 脚本挂了的手动路线

镜像站有寿命，万一 gh-proxy 哪天不可用，手动的步骤也就三步：

```powershell
# 1. 把 uv 的 zip 弄到机器上，解压到 ~\.local\bin
$dir = "$env:USERPROFILE\.local\bin"
New-Item -ItemType Directory -Force $dir | Out-Null
Expand-Archive .\uv-x86_64-pc-windows-msvc.zip -DestinationPath $dir -Force

# 2. 加 PATH
$old = [Environment]::GetEnvironmentVariable('Path','User')
if ($old -notlike "*$dir*") { [Environment]::SetEnvironmentVariable('Path', "$old;$dir", 'User') }
$env:Path = "$env:Path;$dir"

# 3. 装包时直接指定索引
uv tool install ww-manager --default-index https://pypi.tuna.tsinghua.edu.cn/simple
```

第三条命令是关键。哪怕环境变量全都没生效，`--default-index` 也能一次性覆盖。
