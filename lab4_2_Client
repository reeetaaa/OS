#include <iostream>
#include "Windows.h"

using namespace std;

const int messageSize = 128;
char message[messageSize];

void CALLBACK ReadCompletionRoutine(DWORD errorCode, DWORD bytestransfered, LPOVERLAPPED lpOverlapped) {
	cout << message << endl;
}

int main() {
    cout << endl;
	OVERLAPPED overlapped;
	memset(&overlapped, 0, sizeof(overlapped));

	HANDLE hPipe = CreateFile(TEXT("\\\\.\\pipe\\Pipe"), GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);
	if (hPipe == INVALID_HANDLE_VALUE) cout << " [Connection error to the pipe due to an error]:" << GetLastError() << endl;
	else cout << " [Connected to pipe succesfully!]" << endl;
    if (hPipe != INVALID_HANDLE_VALUE)  cout <<" [Data received from the server]:"<< endl;
	bool done = true;
	while (done && hPipe != INVALID_HANDLE_VALUE) {
		done = ReadFileEx(hPipe, message, messageSize, &overlapped, ReadCompletionRoutine);
		SleepEx(INFINITE, TRUE);
	}
	CloseHandle(hPipe);
	return 0;
}
