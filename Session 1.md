# Note 1: " Process Identifier "
Ever executable file has a unique Process ID (PID), But What is a process?
It's basically a program in execution, it is an instance of an executing program that has a unique id.
If I compiled a program,
```bash
gcc main.c -o main
```
then executed it
```shell
./main
```
then tried to show everything in my folder
```shell
ls
```
![[Screenshot 2025-08-14 161356.png]]
(Attachments/Screenshot%2025-08-14%161356.png)
We can notice that I have `main.exe` that is an executable program
If I made it run, every time I make it run I am creating an instance from the original executable program, which has a unique id.
![[Screenshot 2025-08-14 161613.png]]
(Attachments/Screenshot%2025-08-14%161613..png)
If I made my code print out the process id (PID) you will see something like this.
![[Screenshot 2025-08-14 162756.png]]
(Attachments/Screenshot%2025-08-14%162756.png)
You will notice that every one had a unique id.
What is the syntax?
```c
#include <stdio.h>
#include <unistd.h> //this is a built-in header in the sys files

int main() {
	pid_t pid = getpid();
	printf("PID is %u", pid);
	
	return 0;
}
```
`pid_t` is an unsigned integer data type
`getpid()` is a function that returns the process id of the executable

# Note 2: " Parent Process Identification "
A **parent process** is simply the process that **created (spawned)** another process.
If you open **Command Prompt** and type:
```bash
notepad
```
- **Command Prompt** is the parent process.
- **Notepad** is the child process.
When a process ends, its child processes are usually terminated too or get “adopted” by a special system process.
**In operating systems**:
- On **Linux/macOS**, you can get the parent process ID (PPID) with `getppid()`.
- On **Windows**, the OS also keeps track of the parent, but there’s no standard C function to get it, you need Windows-specific APIs or you can make a specific function like this.
```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>

#if defined(_WIN32) || defined(_WIN64) // Windows
#include <windows.h>
#include <tlhelp32.h>
#else // Unix/Linux/Mac
#include <unistd.h>
#endif

pid_t get_parent_pid() {
#if defined(_WIN32) || defined(_WIN64)
    DWORD pid = GetCurrentProcessId();
    DWORD ppid = 0;
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe;a
    pe.dwSize = sizeof(PROCESSENTRY32);

    if (Process32First(hSnapshot, &pe)) {
        do {
            if (pe.th32ProcessID == pid) {
                ppid = pe.th32ParentProcessID;
                break;
            }
        } while (Process32Next(hSnapshot, &pe));
    }
    CloseHandle(hSnapshot);
    return (pid_t)ppid;
#else
    return getppid();
#endif
}

int main() {
    pid_t ppid = get_parent_pid();
    printf("Parent PID: %d\n", ppid);
    return 0;
}
```
It will get something like that:
![[Screenshot 2025-08-14 172317.png]]
So the PPID doesn't change by changing the executable or the instance, it changes when you change the parent of the process (e.g. powershell)
![[Screenshot 2025-08-14 173129.png]]
(Attachments/Screenshot%2025-08-14%173129.png)

In Unix systems and Mac, you can type that command in your terminal and it will show your PPID
```shell
echo $$
```

