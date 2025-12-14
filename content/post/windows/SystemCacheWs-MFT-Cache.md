---
title: 'SystemCacheWs - MFT Cache'
summary: 'How to purge Metafile cache'
date: '2025-09-04T00:00:00+00:00'
---

# SystemCacheWs - MFT Cache 

Recently I worked on a very interesting issue, where Windows system consumed 99% of available RAM, but no proccess table via proccess explorer / task manager was showing no particular proccess, consuming this amount of RAM.

By taking WPR trace I see that most of the RAM is consumed by an entity called SystemCacheWs:

![WPR](https://cdn.porotnikov.com/media/2025/9/4/wpr.jpg)

When viewing the system from RAMMAP, this is how consumption looked like:

![RAMMAP](https://cdn.porotnikov.com/media/2025/9/4/RAMMAP.png)

What is this mystery Metafile entry? It is a cache for [MFT table](https://learn.microsoft.com/en-us/windows/win32/fileio/master-file-table).

      
Back in a day, (windows server 2003/2008 era) there was a way to control the metafile usage with an official [Microsoft tool Dyncache](http://www.microsoft.com/en-us/download/details.aspx?id=9258), but this tool wont work on modern OS:  

![Dyncache tool](https://cdn.porotnikov.com/media/2025/9/4/dyncache_tool.png)

In modern OS the cache control should be fully automatic, and proper solution is to change the way on how files are opened/handled:

[Troubleshoot Cache and Memory Manager Performance Issues | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot)

![Cache issue solution](https://cdn.porotnikov.com/media/2025/9/4/cache_solution.png)

However, is still possible to flush / limit the cache by calling [Windows API](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-setsystemfilecachesize) directly:

Below is a simple POC on how to flush the cache via WinAPI in C++:

```C++
      
#include <windows.h>
#include <iostream>

#define SE_INCREASE_QUOTA_NAME TEXT("SeIncreaseQuotaPrivilege")
#define SE_PROFILE_SINGLE_PROCESS_NAME TEXT("SeProfileSingleProcessPrivilege")

// For NtSetSystemInformation (looked up from ntdll.dll)
typedef NTSTATUS(NTAPI* pNtSetSystemInformation)(
    INT SystemInformationClass,
    PVOID SystemInformation,
    ULONG SystemInformationLength);

// System info classes
const INT SystemFileCacheInformation = 0x15;
const INT SystemMemoryListInformation = 0x50;
const INT MemoryPurgeStandbyList = 4;

BOOL EnablePrivilege(LPCTSTR privilegeName) {
    HANDLE hToken;
    TOKEN_PRIVILEGES tp;
    if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken)) return FALSE;
    LookupPrivilegeValue(NULL, privilegeName, &tp.Privileges[0].Luid);
    tp.PrivilegeCount = 1;
    tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    AdjustTokenPrivileges(hToken, FALSE, &tp, 0, NULL, NULL);
    CloseHandle(hToken);
    return (GetLastError() == ERROR_SUCCESS);
}

BOOL PurgeStandbyList() {
    HMODULE hNtDll = LoadLibrary(TEXT("ntdll.dll"));
    if (!hNtDll) {
        std::cerr << "Failed to load ntdll.dll: " << GetLastError() << std::endl;
        return FALSE;
    }

    pNtSetSystemInformation NtSetSystemInformation = (pNtSetSystemInformation)GetProcAddress(hNtDll, "NtSetSystemInformation");
    if (!NtSetSystemInformation) {
        std::cerr << "Failed to get NtSetSystemInformation: " << GetLastError() << std::endl;
        FreeLibrary(hNtDll);
        return FALSE;
    }

    if (!EnablePrivilege(SE_PROFILE_SINGLE_PROCESS_NAME)) {
        std::cerr << "Failed to enable SeProfileSingleProcessPrivilege: " << GetLastError() << std::endl;
        FreeLibrary(hNtDll);
        return FALSE;
    }

    INT command = MemoryPurgeStandbyList;
    NTSTATUS status = NtSetSystemInformation(SystemMemoryListInformation, &command, sizeof(INT));
    FreeLibrary(hNtDll);

    if (status != 0) {
        std::cerr << "NtSetSystemInformation failed for standby purge: " << status << std::endl;
        return FALSE;
    }

    std::cout << "Standby list purged successfully." << std::endl;
    return TRUE;
}

int main() {
    if (!EnablePrivilege(SE_INCREASE_QUOTA_NAME)) {
        std::cerr << "Failed to enable SeIncreaseQuotaPrivilege: " << GetLastError() << std::endl;
        return 1;
    }

    // Getting current cache limits
    SIZE_T minCurrent, maxCurrent;
    DWORD flagsCurrent;
    GetSystemFileCacheSize(&minCurrent, &maxCurrent, &flagsCurrent);

    // Setting low limits to force file cache flush
    SIZE_T lowMin = 1024 * 1024;  // 1 MB
    SIZE_T lowMax = 1024 * 1024;  // 1 MB
    DWORD flags = 0;
    if (!SetSystemFileCacheSize(lowMin, lowMax, flags)) {
        std::cerr << "Failed to set low cache: " << GetLastError() << std::endl;
        return 1;
    }
    std::cout << "File cache shrunk. Wait 10s for eviction..." << std::endl;
    Sleep(10000);  // Give time for system to adjust

    // Now purge the standby list (EmptyStandbyList equivalent)
    PurgeStandbyList();

    // Restore original or default (e.g., max to -1 for unlimited)
    if (!SetSystemFileCacheSize(minCurrent, maxCurrent, flagsCurrent)) {
        std::cerr << "Failed to restore: " << GetLastError() << std::endl;
    }
    std::cout << "Cache flushed and restored." << std::endl;
    return 0;
}

```

If you want to intentionally populate cache:

1. Create a test data:


```powershell
# --- CONFIG ---
$Root     = "D:\CacheTest-MFT"
$Dirs     = 2000        # dirs per level
$FilesPer = 4500        # files per dir
$Levels   = 2           # directory depth (increase to 2 if you want more dirs)
$NamePad  = 8           # longer names => bigger dir indexes

New-Item -ItemType Directory -Path $Root -Force | Out-Null

function New-Chunk($base, $dirs, $files) {
  # Create subdirs
  0..($dirs-1) | ForEach-Object {
    $d = Join-Path $base ("d{0}" -f $_.ToString().PadLeft($NamePad,'0'))
    New-Item -ItemType Directory -Path $d -Force | Out-Null
    # Create zero-byte files (metadata stress, minimal data)
    0..($files-1) | ForEach-Object {
      New-Item -ItemType File -Path (Join-Path $d ("f{0}.dat" -f $_.ToString().PadLeft($NamePad,'0'))) -Force | Out-Null
    }
  }
}

$curr = $Root
for ($level=1; $level -le $Levels; $level++) {
  New-Chunk -base $curr -dirs $Dirs -files $FilesPer
  if ($level -lt $Levels) {
    $next = Join-Path $curr ("L{0}" -f $level)
    New-Item -ItemType Directory -Path $next -Force | Out-Null
    $curr = $next
  }
}

```

2. Hammer the test data:

```C++

#include <windows.h>
#include <processthreadsapi.h>  // For SetThreadInformation
#include <iostream>
#include <string>
#include <vector>
#include <thread>
#include <random>
#include <atomic>

#define DIR_PATH L"D:\\CacheTest-MFT\\"  // Wide literal; base path for recursive search
#define FILE_PATTERN L"*.dat"             // Wide literal; pattern to match
#define ACCESS_ITERATIONS 100000          // Per thread per file; increase for more pressure
#define CHUNK_SIZE 4096                   // Read 4KB chunks for more cache hit
#define NUM_THREADS 8                     // Threads per file; scale with CPU cores
#define USE_LOW_PRIORITY false            // Set to true to demo mitigation (requires Windows 10+)

BOOL SetThreadMemoryPriority(HANDLE hThread, ULONG priority) {
    MEMORY_PRIORITY_INFORMATION mpi = { priority };
    return SetThreadInformation(hThread, ThreadMemoryPriority, &mpi, sizeof(mpi));
}

void RandomAccessThread(void* mappedView, SIZE_T fileSize, std::atomic<bool>& stopFlag) {
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<SIZE_T> dist(0, fileSize - CHUNK_SIZE);

    HANDLE hThread = GetCurrentThread();
    if (USE_LOW_PRIORITY) {
        // Use 1 for very low priority (as per docs: 1=very low, 5=normal)
        if (!SetThreadMemoryPriority(hThread, 1)) {
            std::cerr << "Failed to set low memory priority: " << GetLastError() << std::endl;
        }
    }

    char buffer[CHUNK_SIZE];
    while (!stopFlag.load()) {
        for (int i = 0; i < ACCESS_ITERATIONS; ++i) {
            SIZE_T offset = dist(gen);
            memcpy(buffer, (char*)mappedView + offset, CHUNK_SIZE);  // Random read
            if (i % 10000 == 0) std::cout << "Accessed at offset " << offset << std::endl;
            Sleep(1);  // Throttle to observe
        }
        // Loop again for continuous access
    }
}

// Recursive function to collect files matching pattern
void CollectFilesRecursive(const std::wstring& baseDir, const std::wstring& pattern, std::vector<std::wstring>& files) {
    std::wstring searchPath = baseDir + L"*";  // First, find all files/dirs in current dir

    WIN32_FIND_DATAW findData;
    HANDLE hFind = FindFirstFileW(searchPath.c_str(), &findData);
    if (hFind == INVALID_HANDLE_VALUE) {
        std::cerr << "Failed to open directory " << std::string(baseDir.begin(), baseDir.end()) << ": " << GetLastError() << std::endl;
        return;
    }

    do {
        if (wcscmp(findData.cFileName, L".") == 0 || wcscmp(findData.cFileName, L"..") == 0) {
            continue;  // Skip current and parent
        }

        std::wstring fullPath = baseDir + findData.cFileName;

        if (findData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY) {
            // Recurse into subdirectory
            CollectFilesRecursive(fullPath + L"\\", pattern, files);
        } else {
            // Check if file matches pattern (simple wildcard match for *.dat)
            if (wcsstr(findData.cFileName, pattern.c_str() + 1) != nullptr) {  // Skip '*' in pattern
                files.push_back(fullPath);
            }
        }
    } while (FindNextFileW(hFind, &findData) != 0);

    FindClose(hFind);
}

int main() {
    std::wstring dir = DIR_PATH;
    std::wstring pattern = FILE_PATTERN;

    std::vector<std::wstring> files;
    CollectFilesRecursive(dir, pattern, files);

    if (files.empty()) {
        std::cerr << "No matching files found recursively." << std::endl;
        return 1;
    }

    std::cout << "Found " << files.size() << " files recursively." << std::endl;

    std::vector<HANDLE> fileHandles;
    std::vector<HANDLE> mappingHandles;
    std::vector<void*> mappedViews;
    std::vector<std::vector<std::thread>> threadGroups;
    std::vector<std::atomic<bool>> stopFlags(files.size());

    for (size_t i = 0; i < files.size(); ++i) {
        const std::wstring& file = files[i];
        HANDLE hFile = CreateFileW(file.c_str(), GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING,
                                   FILE_ATTRIBUTE_NORMAL | FILE_FLAG_RANDOM_ACCESS, NULL);
        if (hFile == INVALID_HANDLE_VALUE) {
            std::cerr << "Failed to open file: " << GetLastError() << std::endl;
            continue;
        }
        fileHandles.push_back(hFile);

        LARGE_INTEGER fileSize;
        GetFileSizeEx(hFile, &fileSize);
        if (fileSize.QuadPart < CHUNK_SIZE) continue;

        HANDLE hMapping = CreateFileMappingW(hFile, NULL, PAGE_READONLY, 0, 0, NULL);
        if (hMapping == NULL) {
            std::cerr << "Failed to map file: " << GetLastError() << std::endl;
            CloseHandle(hFile);
            continue;
        }
        mappingHandles.push_back(hMapping);

        void* mappedView = MapViewOfFile(hMapping, FILE_MAP_READ, 0, 0, 0);
        if (mappedView == NULL) {
            std::cerr << "Failed to view map file: " << GetLastError() << std::endl;
            CloseHandle(hMapping);
            CloseHandle(hFile);
            continue;
        }
        mappedViews.push_back(mappedView);

        // Spawn threads for this file
        stopFlags[i].store(false);
        threadGroups.emplace_back();
        for (int t = 0; t < NUM_THREADS; ++t) {
            threadGroups.back().emplace_back(RandomAccessThread, mappedView, (SIZE_T)fileSize.QuadPart, std::ref(stopFlags[i]));
        }

        std::cout << "Mapped and accessing file." << std::endl;  // Skip path print to avoid wcout for simplicity
    }

    std::cout << "All files mapped. Press Enter to stop..." << std::endl;
    std::cin.get();  // Wait for user input to stop

    // Stop all threads
    for (auto& flag : stopFlags) flag.store(true);
    for (auto& group : threadGroups) {
        for (auto& th : group) if (th.joinable()) th.join();
    }

    // Cleanup
    for (auto view : mappedViews) UnmapViewOfFile(view);
    for (auto mapping : mappingHandles) CloseHandle(mapping);
    for (auto file : fileHandles) CloseHandle(file);

    return 0;
}

```
