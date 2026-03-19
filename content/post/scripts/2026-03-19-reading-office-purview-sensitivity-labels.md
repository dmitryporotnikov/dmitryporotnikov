---
title: 'Reading Microsoft Purview Sensitivity Labels from Office Files'
summary: 'A PowerShell script that extracts Purview sensitivity label metadata from Office Open XML files'
date: '2026-03-19T00:00:00+00:00'
categories:
    - Scripts
    - Microsoft Purview
    - PowerShell
---

## Reading Microsoft Purview Sensitivity Labels from Office Files

When working with Microsoft Purview (now Microsoft 365 compliance), sensitivity labels applied to Office documents are embedded directly into the file structure. This can be useful for auditing, compliance verification, or automation scenarios where you need to extract label information from multiple files without opening each one.

Office Open XML files (.docx, .xlsx, .pptx, etc.) store label metadata in two locations within their ZIP structure: `docProps/custom.xml` and `docMetadata/LabelInfo.xml`. This script reads both locations and returns structured label information.

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem

function Get-XmlPropertyValue {
    param([System.Xml.XmlElement]$PropertyNode)

    foreach ($child in $PropertyNode.ChildNodes) {
        if ($child.NodeType -eq [System.Xml.XmlNodeType]::Element) {
            return $child.InnerText
        }
    }

    return $null
}

function Get-PurviewLabelFromOfficeFile {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Path
    )

    if (-not (Test-Path -LiteralPath $Path)) {
        throw "File not found: $Path"
    }

    $resolvedPath = (Resolve-Path -LiteralPath $Path).Path
    $results = @()

    try {
        $zip = [System.IO.Compression.ZipFile]::OpenRead($resolvedPath)
    }
    catch {
        throw "File is not a readable Office Open XML package (or it is encrypted/protected in a way this ZIP-based reader can't inspect): $resolvedPath"
    }

    try {
        $sensitivityProperty = $null
        $customPropsByLabel = @{}

        # -------- Read docProps/custom.xml --------
        $customEntry = $zip.Entries | Where-Object { $_.FullName -eq 'docProps/custom.xml' } | Select-Object -First 1
        if ($customEntry) {
            $stream = $customEntry.Open()
            try {
                $xml = New-Object System.Xml.XmlDocument
                $xml.Load($stream)

                $propertyNodes = $xml.SelectNodes("//*[local-name()='property']")
                foreach ($prop in $propertyNodes) {
                    $name = $prop.GetAttribute("name")
                    $value = Get-XmlPropertyValue -PropertyNode $prop

                    if ([string]::IsNullOrWhiteSpace($name)) {
                        continue
                    }

                    if ($name -eq 'Sensitivity') {
                        $sensitivityProperty = $value
                        continue
                    }

                    if ($name -match '^MSIP_Label_([0-9a-fA-F-]{36})_(.+)$') {
                        $labelId = $matches[1].ToLowerInvariant()
                        $attrName = $matches[2]

                        if (-not $customPropsByLabel.ContainsKey($labelId)) {
                            $customPropsByLabel[$labelId] = @{}
                        }

                        $customPropsByLabel[$labelId][$attrName] = $value
                    }
                }
            }
            finally {
                $stream.Dispose()
            }
        }

        # -------- Read docMetadata/LabelInfo.xml --------
        $labelInfoEntry = $zip.Entries | Where-Object { $_.FullName -eq 'docMetadata/LabelInfo.xml' } | Select-Object -First 1
        if ($labelInfoEntry) {
            $stream = $labelInfoEntry.Open()
            try {
                $xml = New-Object System.Xml.XmlDocument
                $xml.Load($stream)

                $labelNodes = $xml.SelectNodes("//*[local-name()='label']")
                foreach ($node in $labelNodes) {
                    $labelId  = $node.GetAttribute("id")
                    $tenantId = $node.GetAttribute("siteId")
                    $enabled  = $node.GetAttribute("enabled")
                    $removed  = $node.GetAttribute("removed")
                    $method   = $node.GetAttribute("method")

                    $customData = $null
                    if ($labelId -and $customPropsByLabel.ContainsKey($labelId.ToLowerInvariant())) {
                        $customData = $customPropsByLabel[$labelId.ToLowerInvariant()]
                    }

                    $results += [pscustomobject]@{
                        FilePath            = $resolvedPath
                        Source              = "LabelInfo.xml"
                        LabelId             = $labelId
                        TenantId            = $tenantId
                        Enabled             = $enabled
                        Removed             = $removed
                        Method              = $method
                        SetDate             = if ($customData) { $customData["SetDate"] } else { $null }
                        Name                = if ($customData) { $customData["Name"] } else { $null }
                        ContentBits         = if ($customData) { $customData["ContentBits"] } else { $null }
                        SensitivityProperty = $sensitivityProperty
                    }
                }
            }
            finally {
                $stream.Dispose()
            }
        }

        # -------- Fallback to custom.xml only --------
        if ($results.Count -eq 0 -and $customPropsByLabel.Count -gt 0) {
            foreach ($labelId in $customPropsByLabel.Keys) {
                $data = $customPropsByLabel[$labelId]

                $results += [pscustomobject]@{
                    FilePath            = $resolvedPath
                    Source              = "custom.xml"
                    LabelId             = $labelId
                    TenantId            = $data["SiteId"]
                    Enabled             = $data["Enabled"]
                    Removed             = $null
                    Method              = $data["Method"]
                    SetDate             = $data["SetDate"]
                    Name                = $data["Name"]
                    ContentBits         = $data["ContentBits"]
                    SensitivityProperty = $sensitivityProperty
                }
            }
        }

        # -------- Last fallback: only Sensitivity property --------
        if ($results.Count -eq 0 -and $sensitivityProperty) {
            $results += [pscustomobject]@{
                FilePath            = $resolvedPath
                Source              = "Sensitivity property only"
                LabelId             = $sensitivityProperty
                TenantId            = $null
                Enabled             = $null
                Removed             = $null
                Method              = $null
                SetDate             = $null
                Name                = $null
                ContentBits         = $null
                SensitivityProperty = $sensitivityProperty
            }
        }

        if ($results.Count -eq 0) {
            $results += [pscustomobject]@{
                FilePath            = $resolvedPath
                Source              = "None found"
                LabelId             = $null
                TenantId            = $null
                Enabled             = $null
                Removed             = $null
                Method              = $null
                SetDate             = $null
                Name                = $null
                ContentBits         = $null
                SensitivityProperty = $null
            }
        }

        return $results
    }
    finally {
        $zip.Dispose()
    }
}

Get-PurviewLabelFromOfficeFile -Path C:\Users\user\Desktop\ConfTest.docx
```

### Output

The function returns a PSCustomObject with:
- **FilePath** - Path to the analyzed file
- **Source** - Which XML location the label was found in
- **LabelId** - The unique identifier for the sensitivity label
- **TenantId** - The Microsoft 365 tenant identifier
- **Enabled** - Whether the label is enabled
- **Removed** - Whether the label was removed
- **Method** - How the label was applied (manual, automatic, etc.)
- **SetDate** - When the label was applied
- **Name** - The label display name
- **ContentBits** - Content watermark information
- **SensitivityProperty** - The raw sensitivity property value

### Usage

To use the script, call `Get-PurviewLabelFromOfficeFile` with the `-Path` parameter pointing to an Office file:

```powershell
Get-PurviewLabelFromOfficeFile -Path "C:\path\to\document.docx"
```

The script works with .docx, .xlsx, .pptx, and other Office Open XML format files. It will not work with encrypted or rights-protected files.
