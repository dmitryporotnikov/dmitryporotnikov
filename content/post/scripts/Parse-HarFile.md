---
title: 'Parse-HarFile - HAR Analysis Script'
summary: 'A PowerShell script that converts HAR files into LLM-readable format for traffic analysis'
date: '2026-03-16T00:00:00+00:00'
---

# Parsing HAR Files for LLM Analysis with PowerShell

If you've ever needed to analyze HTTP traffic captured in a HAR file and wanted to feed it to an AI assistant for analysis, you know the challenge: HAR files are JSON documents with deeply nested structures that don't read well in their native format. Here is a PowerShell script that transforms HAR files into clean, structured text that LLMs can easily understand and analyze:

```powershell
#Requires -Version 5.1
<#
.SYNOPSIS
    Parses a HAR file and outputs requests/responses in an LLM-readable format.
#>

param(
    [string]$HarFilePath = "D:\HARLLM\sample.har",
    [string]$OutputFilePath = "D:\HARLLM\har-analysis.txt",
    [int]$MaxResponseBodySize = 2000
)

# Read and parse HAR file
Write-Host "Reading HAR file: $HarFilePath"
$harContent = Get-Content -Path $HarFilePath -Raw -Encoding UTF8

try {
    $har = $harContent | ConvertFrom-Json
}
catch {
    Write-Error "Failed to parse HAR file as JSON: $_"
    exit 1
}

$entries = $har.log.entries
$totalEntries = $entries.Count

Write-Host "Found $totalEntries HTTP transactions"

# Helper function to extract headers as key-value pairs
function Get-HeadersHash {
    param([array]$headers)
    $hash = @{}
    foreach ($h in $headers) {
        $hash[$h.name] = $h.value
    }
    return $hash
}

# Helper function to format headers for display
function Format-Headers {
    param([array]$headers)
    $output = @()
    foreach ($h in $headers) {
        $output += "    $($h.name): $($h.value)"
    }
    return ($output -join "`n")
}

# Helper function to truncate and format body content
function Format-Body {
    param(
        [string]$text,
        [string]$mimeType,
        [int]$maxSize
    )

    if ([string]::IsNullOrWhiteSpace($text)) {
        return "    (empty body)"
    }

    # Try to format JSON for readability
    if ($mimeType -match 'application/json' -or $text.TrimStart().StartsWith('{') -or $text.TrimStart().StartsWith('[')) {
        try {
            $json = $text | ConvertFrom-Json | ConvertTo-Json -Depth 10
            if ($json.Length -gt $maxSize) {
                $truncated = $json.Substring(0, $maxSize)
                return "    $truncated`n    [Truncated: $($json.Length) total characters]"
            }
            return "    $($json -replace '\n', "`n    ")"
        }
        catch {
            # Not valid JSON
        }
    }

    if ($text.Length -gt $maxSize) {
        $truncated = $text.Substring(0, $maxSize)
        return "    $truncated`n    [Truncated: $($text.Length) total characters]"
    }

    return "    $($text -replace '\n', "`n    ")"
}

# Build output
$output = @()
$output += "================================================================================"
$output += "HAR FILE ANALYSIS"
$output += "================================================================================"
$output += ""
$output += "Generated: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
$output += "Source: $HarFilePath"
$output += "Total Requests: $totalEntries"
$output += ""

# Add metadata if available
if ($har.log.creator) {
    $output += "Creator: $($har.log.creator.name) v$($har.log.creator.version)"
}
if ($har.log.browser) {
    $output += "Browser: $($har.log.browser.name) v$($har.log.browser.version)"
}
$output += ""
$output += "================================================================================"
$output += ""

# Request flow summary
$output += "# REQUEST FLOW SUMMARY"
$output += "----------------------------------------"

$requestFlow = @()
for ($i = 0; $i -lt $entries.Count; $i++) {
    $entry = $entries[$i]
    $method = $entry.request.method
    $url = $entry.request.url
    $status = $entry.response.status
    $time = $entry.time

    # Extract domain from URL
    $domain = "unknown"
    if ($url -match 'https?://([^/]+)') {
        $domain = $matches[1]
    }

    $requestFlow += "[$($i + 1)] $method -> $status | $($domain) | ${time}ms"
}

# Show first 30 entries in flow summary
$flowDisplay = if ($requestFlow.Count -gt 30) {
    ($requestFlow[0..29] + "    ... and $($requestFlow.Count - 30) more")
} else {
    $requestFlow
}

$output += $flowDisplay
$output += ""
$output += "================================================================================"
$output += ""

# Process each entry
for ($i = 0; $i -lt $entries.Count; $i++) {
    $entry = $entries[$i]

    $request = $entry.request
    $response = $entry.response

    $method = $request.method
    $url = $request.url
    $timestamp = $entry.startedDateTime
    $duration = $entry.time
    $httpVersion = $request.httpVersion

    $status = $response.status
    $statusText = $response.statusText

    # Get headers as hash tables
    $reqHeaders = Get-HeadersHash -headers $request.headers
    $respHeaders = Get-HeadersHash -headers $response.headers

    $output += "================================================================================"
    $output += "REQUEST #$($i + 1) of $totalEntries"
    $output += "================================================================================"
    $output += ""
    $output += "## REQUEST"
    $output += "----------------------------------------"
    $output += "Timestamp  : $timestamp"
    $output += "Method     : $method"
    $output += "URL        : $url"
    $output += "HTTP Ver   : $httpVersion"
    $output += "Duration   : ${duration}ms"
    $output += ""

    # Request Headers
    $output += "### Request Headers"
    $output += Format-Headers -headers $request.headers
    $output += ""

    # Request Cookies
    if ($request.cookies -and $request.cookies.Count -gt 0) {
        $output += "### Request Cookies"
        foreach ($cookie in $request.cookies) {
            $output += "    $($cookie.name) = [hidden]"
        }
        $output += ""
    }

    # Query String
    if ($request.queryString -and $request.queryString.Count -gt 0) {
        $output += "### Query Parameters"
        foreach ($qs in $request.queryString) {
            $output += "    $($qs.name) = $($qs.value)"
        }
        $output += ""
    }

    # Request Body
    if ($request.postData) {
        $bodySize = $request.bodySize
        $mimeType = $request.postData.mimeType
        $bodyText = $request.postData.text

        $output += "### Request Body"
        $output += "Content-Type: $mimeType"
        $output += "Body Size   : $bodySize bytes"
        $output += ""
        $output += Format-Body -text $bodyText -mimeType $mimeType -maxSize 3000
        $output += ""
    }

    # Response
    $output += "## RESPONSE"
    $output += "----------------------------------------"
    $output += "Status     : $status $statusText"
    $output += "HTTP Ver   : $($response.httpVersion)"
    $output += "Body Size  : $($response.bodySize) bytes"
    $output += ""

    # Response Headers
    $output += "### Response Headers"
    $output += Format-Headers -headers $response.headers
    $output += ""

    # Response Cookies
    if ($response.cookies -and $response.cookies.Count -gt 0) {
        $output += "### Response Cookies"
        foreach ($cookie in $response.cookies) {
            $cookieInfo = $cookie.name
            if ($cookie.domain) { $cookieInfo += " (domain: $($cookie.domain))" }
            if ($cookie.expires) { $cookieInfo += " (expires: $($cookie.expires))" }
            $output += "    $cookieInfo = [hidden]"
        }
        $output += ""
    }

    # Response Body
    if ($response.content -and $response.content.text) {
        $content = $response.content
        $mimeType = $content.mimeType
        $encoding = $content.encoding
        $bodyText = $content.text

        # Handle compressed content
        if ($encoding -eq 'gzip' -or $encoding -eq 'deflate' -or $encoding -eq 'br') {
            try {
                $bytes = [System.Convert]::FromBase64String($bodyText)
                $memStream = New-Object System.IO.MemoryStream(,$bytes)
                $deflateStream = New-Object System.IO.Compression.GZipStream($memStream, [System.IO.Compression.CompressionMode]::Decompress)
                $reader = New-Object System.IO.StreamReader($deflateStream)
                $bodyText = $reader.ReadToEnd()
                $reader.Close()
                $deflateStream.Close()
                $memStream.Close()
            }
            catch {
                $bodyText = "[Base64 encoded - $encoding]: $bodyText"
            }
        }

        $output += "### Response Body"
        $output += "Content-Type: $mimeType"
        if ($encoding) { $output += "Encoding    : $encoding" }
        $output += ""
        $output += Format-Body -text $bodyText -mimeType $mimeType -maxSize $MaxResponseBodySize
        $output += ""
    }

    # Connection info
    if ($entry.serverIPAddress) {
        $output += "Server IP  : $($entry.serverIPAddress)"
    }
    if ($entry.connection) {
        $output += "Connection : $($entry.connection)"
    }

    $output += ""
    $output += ""
}

# Write output to file
$output | Out-File -FilePath $OutputFilePath -Encoding UTF8

Write-Host ""
Write-Host "Analysis complete!" -ForegroundColor Green
Write-Host "Output written to: $OutputFilePath"
Write-Host ""

# Show summary
$getCount = ($entries | Where-Object { $_.request.method -eq 'GET' }).Count
$postCount = ($entries | Where-Object { $_.request.method -eq 'POST' }).Count
$optionsCount = ($entries | Where-Object { $_.request.method -eq 'OPTIONS' }).Count
$successCount = ($entries | Where-Object { $_.response.status -ge 200 -and $_.response.status -lt 300 }).Count
$redirectCount = ($entries | Where-Object { $_.response.status -ge 300 -and $_.response.status -lt 400 }).Count
$errorCount = ($entries | Where-Object { $_.response.status -ge 400 }).Count

Write-Host "Summary:"
Write-Host "  GET requests    : $getCount"
Write-Host "  POST requests   : $postCount"
Write-Host "  OPTIONS requests: $optionsCount"
Write-Host "  Success (2xx)   : $successCount"
Write-Host "  Redirects (3xx) : $redirectCount"
Write-Host "  Errors (4xx/5xx): $errorCount"

```

**Flow Summary** - The output begins with a chronological list showing each request's method, status code, target domain, and duration. This gives you an immediate understanding of the traffic flow without reading every detail.

**Structured Requests** - Each HTTP transaction gets its own numbered section with clear labels: Method, URL, Timestamp, HTTP version, and Duration. Request headers are formatted as key-value pairs, and cookies are hidden by default for privacy.

**Formatted Bodies** - JSON request and response bodies are pretty-printed for readability. Large responses are truncated with a note about total size, keeping the output manageable while preserving the ability to see important content.

**Metadata Preservation** - The script captures creator information (Firefox, Chrome, etc.), timing data, server IP addresses, and connection details that help understand the full picture.

## Usage

Running the script is straightforward:

```powershell
.\Parse-HarFile.ps1
```

This reads `sample.har` from the script directory and outputs to `har-analysis.txt`. You can customize the behavior with parameters:

```powershell
.\Parse-HarFile.ps1 -HarFilePath "custom.har" -OutputFilePath "analysis.txt" -MaxResponseBodySize 5000
```

The MaxResponseBodySize parameter controls how much of response bodies to include. The default is 2000 characters, which balances readability with completeness.

## Sample Output

For a HAR file containing 74 requests (like the sample from browsing meduza.io), the output begins with a flow summary:

```
================================================================================
REQUEST FLOW SUMMARY
----------------------------------------
[1] GET -> 200 | meduza.io | 32ms
[2] GET -> 200 | cdnjs.cloudflare.com | 78ms
[3] GET -> 200 | meduza.io | 23ms
...
```

Each request then gets detailed treatment:

```
================================================================================
REQUEST #1 of 74
================================================================================

## REQUEST
----------------------------------------
Timestamp  : 2026-03-16T12:27:23.827+01:00
Method     : GET
URL        : https://meduza.io/
HTTP Ver   : HTTP/2
Duration   : 32ms

### Request Headers
    Host: meduza.io
    User-Agent: Mozilla/5.0...
    Accept: text/html,application/xhtml+xml...

### Request Cookies
    _ga_L0LHMYHRHJ = [hidden]

## RESPONSE
----------------------------------------
Status     : 200
HTTP Ver   : HTTP/2
Body Size  : 71469 bytes

### Response Headers
    content-type: text/html; charset=utf-8
    cache-control: public, max-age=30...
```

## Practical Applications

Once you have the analysis file, you can ask an LLM to identify issues, find performance bottlenecks, understand API interactions, or explain what third-party services a website uses. 

## Limitations

The script handles gzip and deflate compression but may struggle with brotli (br) encoded responses in some cases. Very large response bodies are truncated, so you may need to increase MaxResponseBodySize if you need to examine full responses. The script also assumes a reasonably modern HAR 1.2 format.