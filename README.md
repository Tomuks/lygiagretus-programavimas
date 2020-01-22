
#include <iostream>
#include <iostream>
#include <fstream>
#include <thread>
#include <mutex>
#include <iomanip>

using namespace std;

int threadSk = 5;
int duomenuKiekis = 25;
int duomenuCount = 0;
int rez = 0;
int m[25]; // nuskaityti duomenis

class Monitor
{
private:

	mutex mutex;
	condition_variable cv;
	condition_variable cv1;
	int* Monitorskaiciai;
	int index = -1;

public:
	int arraySize = 10;
	int count = 0;
	Monitor() {
		//int Monitorskaiciai[arraySize];
		Monitorskaiciai = new int[arraySize];
	}
	~Monitor() {}
	int completion = 0;

	Monitor(int size)
	{
		arraySize = size;
		Monitorskaiciai = new int[arraySize];
	}

	void create(int sk)
	{
		count++;
		unique_lock<std::mutex> lk(mutex);

		if (index >= arraySize - 1) // tikrinam ar ne pilnas monitorius
		{
			cv.wait(lk, [&] {return index < arraySize-1; }); //jeigu pilnas laukiam kol atsiras vietos
		}

		index++;
		Monitorskaiciai[index] = sk;

		lk.unlock();

		cv1.notify_one();
	}

	int remove()
	{
		unique_lock<std::mutex> lk(mutex);
		completion++;
		if (index < 0)
		{
			cv1.wait(lk, [&] {return index >= 0; }); // tikrina ar ne tuscias
		}

		int temp;
		temp = Monitorskaiciai[index];
		index--;
		lk.unlock();
		cv.notify_one();
		
		return temp;
	}

};

Monitor monitor;
Monitor result(25);

void Insert()
{
	while (monitor.count < duomenuKiekis)
	{
		monitor.create(m[monitor.count]);
	}
}
void Calculate()
{
	while (monitor.completion < duomenuKiekis)
	{
		int ats = monitor.remove();
		result.create(ats);
	}
}

int main()
{

	//cout << m[2];

	vector<thread> threads;
	threads.reserve(threadSk);
	for (size_t i = 0; i < threadSk; i++)
	{
		thread t(Calculate);
		threads.push_back(move(t));
	}

	Insert();

	for (std::thread& th : threads)
	{
		if (th.joinable())
			th.join();
	}

	for (size_t i = 0; i < result.count; i++)
	{
		int a = result.remove();
		cout << a << endl;
	}

}


