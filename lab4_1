#pragma comment (lib, "winmm.lib")
#include "windows.h"
#include <iostream>
#include "fstream"
#include "time.h"
#include <iomanip>

using namespace std;

LPVOID initBuffer(int size, bool read) {
	HANDLE file = CreateFile(L"C:\\buffer.txt", GENERIC_ALL, 0, NULL, read ? OPEN_EXISTING : CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
	HANDLE buffer = OpenFileMapping(read ? GENERIC_READ : GENERIC_WRITE, NULL, L"buffer");
	if (buffer == NULL) buffer = CreateFileMapping(file, NULL, PAGE_READWRITE, 0, size, L"buffer");
	LPVOID buffAddress = MapViewOfFile(buffer, read ? FILE_MAP_READ : FILE_MAP_WRITE, 0, 0, size);
	VirtualLock(buffAddress, size);
	return buffAddress;
}

void initSemaphores(HANDLE* arr, int numOfPages, int state, wchar_t name[]) {
	for (int i = 0; i < numOfPages; i++) {
		name[2] = i + '0';
		arr[i] = OpenSemaphore(SYNCHRONIZE | SEMAPHORE_MODIFY_STATE, false, name);
		if (arr[i] == NULL) arr[i] = CreateSemaphore(NULL, state, 1, name);
	}
}

int main(int argc, char* argv[]) {
	system("cls");
	srand(time(NULL));

	SYSTEM_INFO sysInfo;
	GetSystemInfo(&sysInfo);

	const int pages = 3 + 0 + 7 + 4 +0;
	const int pageSize = sysInfo.dwPageSize;
	const int buffSize = pageSize * pages;

	bool reader = !strcmp(argv[1], "reader");
	cout << (reader ? "Reader App" : "Writer App") << endl;

	char* message = new char[pageSize]();
	if (!reader) memset(message, 97 + rand() % 26, pageSize);

	fstream journal;
	journal.open(reader ? "C:\\ReaderJournal.txt" : "C:\\WriterJournal.txt", fstream::app);

	LPVOID buffAddress = initBuffer(buffSize, reader);

	HANDLE* sr = new HANDLE[pages];
	wchar_t srName[] = { L"reader " };
	initSemaphores(sr, pages, 0, srName);

	HANDLE* sw = new HANDLE[pages];
	wchar_t swName[] = { L"writer " };
	initSemaphores(sw, pages, 1, swName);

	HANDLE* wait = reader ? sr : sw;
	HANDLE* release = reader ? sw : sr;

	cout << std::setw(10) << "ADDRESS" << " | " << std::setw(10) << "PAGE NUMBER" << endl << "-----------+---------------" << endl;
	journal << endl << std::setw(10) << "PROCESS ID" << " | " << std::setw(10) << "TIME" << " | " << std::setw(12) << "STATE" << " | " << std::setw(10) << "PAGE NUMBER" << endl << "-----------+------------+--------------+------------" << endl;
	while (true) {
		journal << std::setw(10) << GetCurrentProcessId() << " | " << std::setw(10) << timeGetTime() << " | " << std::setw(12) << "Wait" << " | " << endl;
		int curPage = WaitForMultipleObjects(pages, wait, false, INFINITE);

		journal << std::setw(10) << GetCurrentProcessId() << " | " << std::setw(10) << timeGetTime() << " | " << std::setw(12) << (reader ? "Read begin" : "Write begin") << " | " << std::setw(10) << curPage << endl;
		cout << std::setw(10) << (int)buffAddress + curPage * pageSize << " | " << std::setw(5) << curPage << endl;

		if (reader) memcpy(message, (void*)((int)buffAddress + curPage * pageSize), pageSize);
		else memcpy((void*)((int)buffAddress + curPage * pageSize), message, pageSize);

		Sleep(500 + rand() % 1000);
		journal << std::setw(10) << GetCurrentProcessId() << " | " << std::setw(10) << timeGetTime() << " | " << std::setw(12) << (reader ? "Read end" : "Write end") << " | " << std::setw(10) << curPage << endl;
		ReleaseSemaphore(release[curPage], 1, NULL);
	}

	return 0;
}
