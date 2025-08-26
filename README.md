# DLL-INJECTOR
//open cd with project, compile ant start, before start edit path to your dll, and start.	
//after this you have argv 0 - name, argv 1 - name programm to inject.	

#include <Windows.h>
#include <stdio.h>
#include <tlhelp32.h>
#include <iostream>
#include <string>

using namespace std;

int main(int argc, wchar_t* argv[]) {

	PROCESSENTRY32 pe32;
	pe32.dwSize = sizeof(PROCESSENTRY32);
	string puffy;
	HANDLE SHAPSHOT = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	if (argc < 2) {
		cout << "<program name.exe> <what`s programm.exe>";
		return 1;
	}
	const wchar_t* copy = argv[1];
	Process32First(SHAPSHOT, &pe32);
	do{
		if (wcscmp(pe32.szExeFile, copy) == 0) {
			DWORD PID = pe32.th32ProcessID;
			LPCSTR dll_path = "path to dll";
			
			HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, PID);
			if (hProcess == NULL) {
				cout << "can`t open process: " << GetLastError();
				return 1;
			}
			HANDLE alloc_mem = VirtualAllocEx(hProcess, NULL, strlen(dll_path) + 1, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
			if (alloc_mem == NULL) {
				cout << "Can`t allocate memory: " << GetLastError();
				return 1;
			}
			WriteProcessMemory(hProcess, alloc_mem, dll_path, strlen(dll_path) + 1, NULL);
			cout << "Memory allocated at: " << alloc_mem;

			HMODULE Kernel = GetModuleHandleA("kernel32.dll");
			if (Kernel == NULL) {
				cout << "Can`t get kernel32: " << GetLastError();
				return 1;
			}

			FARPROC lLoadLib = GetProcAddress(Kernel, "LoadLibraryA");

			HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)lLoadLib, alloc_mem, NULL, 0);
			if (hThread == NULL) {
				cout << "Can`t create thread: " << GetLastError();
				return 1;
			}

			WaitForSingleObject(hProcess, INFINITE);
			VirtualFreeEx(hThread, alloc_mem, 0, MEM_RELEASE);
			CloseHandle(hThread);
			CloseHandle(hProcess);
		}
		

	
	} while (Process32Next(SHAPSHOT, &pe32));
	return 0;
}
