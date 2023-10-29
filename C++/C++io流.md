## C++IO流

![click on an element for detailed information](./C++io%E6%B5%81.assets/iostream.gif)

**流输入/输出 ：\<istream\>、\<ostream\>、\<iostream\>**

**文件流输入/输出 ：\<ifstream\>、\<ofstream\>、\<fstream\>**

**字符串流输入/输出 ：\<istringstream\>、\<ostringstream\>、\<sstream\>**

### 1. iostream

C++标准库提供了4个全局流对象cin、cout、cerr、clog，使用cout进行标准输出，即数据从内存流向控制台(显示器)。使用cin进行标准输入即数据通过键盘输入到程序中，同时C++标准库还提供了cerr用来进行标准错误的输出，以及clog进行日志的输出，从上图可以看出，cout、cerr、clog是ostream类的三个不同的对象，因此这三个对象现在基本没有区别，只是应用场景不同。

在使用时候必须要包含文件并引入std标准命名空间。

#### 特点：

1. cin为缓冲流。键盘输入的数据保存在缓冲区中，当要提取时，是从缓冲区中拿。如果一次输入过多，会留在那儿慢慢用，如果输入错了，必须在回车之前修改，如果回车键按下就无法挽回了。只有把输入缓冲区中的数据取完后，才要求输入新的数据。

2. 输入的数据类型必须与要提取的数据类型一致，否则出错。出错只是在流的状态字 state 中对应位置位(置1)，程序继续。

3. 空格和回车都可以作为数据之间的分格符，所以多个数据可以在一行输入，也可以分行输入。但如果是字符型和字符串，则空格（ASCI码为32）无法用cin输入，字符串中也不能有空格。回车符也无法读入。

4. cin 和 cout 可以直接输入和输出内置类型数据。

5. 对于自定义类型，如果要支持 cin 和 cout 的标准输入输出，需要对 << 和 >> 进行重载。

   ```c++
   class Date
   {
   public:
   	Date(int year = 1900, int month = 1, int day = 1)
   		: _year(year)
   		, _month(month)
   		, _day(day)
   	{}
       // 因操作中访问到了私有成员，在没有get()函数时，只能使用友元函数
   	friend ostream& operator<< (ostream& os, Date& d);
   private:
   	int _year;
   	int _month;
   	int _day;
   };
   
   ostream& operator<< (ostream& os, Date& d)
   {
   	os << d._year << "-" << d._month << "-" << d._day << endl;
   	return os;
   }
   
   int main()
   {
   	Date d;
   	cout << d << endl;	// 1900-1-1
       
       return 0;
   }
   ```

   > 特别注意：由于 fstream 和 sstream 中的流是 iostream 的子类，故 定义了`ostream& operator<<`
   >
   > 就会使用切片的思想自动实现了 `ostringstream& operator<<`  以及 `ofstream& operator<<` 

5. 在线OJ中的输入和输出：

   - 对于IO类型的算法，一般都需要循环输入.

   - 严格按照题目的要求进行，多一个少一个空格都不行。

   - 连续输入时，vs系列编译器下在输入 `ctrl+Z` 时结束

     ```c++
     // 单个元素循环输入
     while(cin>>a)
     {
         // ...
     }
     // 多个元素循环输入
     while(c>>a>>b>>c)
     {
         // ...
     }
     // string重载>>
     while(cin>>str)
     {
         // ...
     }
     
     // 在C语言中是这样的
     char a[128];
     while (scanf("%s", a) != EOF)
     {
         cout << a << endl;
     }
     ```

6. istream类型对象可以转换为逻辑条件判断值 （对第6点的补充）

   ```c++
   istream& operator>> (int& val);
   explicit operator bool() const;
   ```

   > 实际上我们看到使用while (cin >> i) 去流中提取对象数据时，调用的是 `operator>>`，返回值是 `istream`类型的对象，那么这里可以做逻辑条件值，源自于`istream`的对象又调用了`operator bool`，调用时如果接收流失败，或者有结束标志，则返回 `false`

#### 一些常用接口：

1. 获取单个字符/整行(以\n为结尾):

     `char a; cin.get(a);`   / `char buf[100]; cin.getline(buf,sizeof buf);`

2. 关闭和c语言中stdio的同步输入：`std::ios_base::sync_with_stdio(false);`, 在关闭下可以加快流输入，如果再使用 stdio 可能会导致意外交错字符。

3. 最多忽略n个字符，直到找到delim字符：`cin.ignore (streamsize n = 1, int delim = EOF);`

   返回值为 以n字符开始的输入流，一般配合`cin.get()`使用。

4. 清除错误状态[iostate](https://legacy.cplusplus.com/ios_base::iostate).：`cin.clear();` 当`cin>>a`失败时，清除状态使流输入继续。

   一般配合清空缓冲区的操作，使用 `cin.sync();` 或者 `ignore`;

5. 重定向：使用缓冲区指针`std::streambuf *cinbuf = std::cin.rdbuf();` `std::cin.rdbuf(新缓冲区指针)`

6. 查看下一个字符：`fs.peek()`

### 2. fstream

#### 特点：

1. 与iostream大致相同

2. C++根据文件内容的数据格式分为**二进制文件**和**文本文件**。

3. 采用文件流对象操作文件的一般步骤：

   1. 定义一个文件流对象
      - ifstream ifile(只输入用)
      - ofstream ofile(只输出用)
      - fstream iofile(既输入又输出用)

   2. 使用文件流对象的成员函数打开一个磁盘文件，使得文件流对象和磁盘文件之间建立联系
   3. 使用提取和插入运算符对文件进行读写操作，或使用成员函数进行读写
   4. 关闭文件

```c++
struct ServerInfo
{
    // 二进制读写非常不建议使用string
	char _ip[32];
	int _port;
};

class ConfigManager
{
public:
	ConfigManager(const char* filename)
		:_filename(filename)
	{}

	void WriteBin(const ServerInfo& info)//二进制 写
	{
		ofstream ofs(_filename.c_str(), ios_base::out | ios_base::binary);
		ofs.write((const char*)&info, sizeof(ServerInfo));
        ofs.close();
	}

	void ReadBin(ServerInfo& info)//二进制 读
	{
		ifstream ifs(_filename.c_str(), ios_base::in | ios_base::binary);
		ifs.read((char*)&info, sizeof(ServerInfo));
        ifs.close();
	}

	void WriteTest(const ServerInfo& info)//文本 写
	{
		ofstream ofs(_filename.c_str());
		ofs << info._ip << info._port;
        ofs.close();
	}

	void ReadTest(ServerInfo& info)//文本 读
	{
		ifstream ifs(_filename.c_str());
		ifs >> info._ip >> info._port;
        ifs.close();
	}

private:
	string _filename;
};

int main()
{
    // 网络
	、、ServerInfo info = { "127.0.0.1",80 };
	//ConfigManager cm("config.bin");
	////cm.WriteBin(info);
	//cm.ReadBin(info);
	//cout << info._ip << ":" << info._port << endl;

	ConfigManager cm("config.test");
	ServerInfo winfo;
    // 写入
	cm.ReadTest(winfo);
	cout << winfo._ip << winfo._port;
	return 0;
}
```

4. fstream的构造函数可选项

   ```c++
   explicit ifstream (const string& filename, ios_base::openmode mode = ios_base::in);
   ```

   |                        | member constant | stands for   | access                                                       |
   | ---------------------- | --------------- | ------------ | ------------------------------------------------------------ |
   | 可读                   | in              | **in**put    | File open for reading: the *[internal stream buffer](https://legacy.cplusplus.com/ifstream::rdbuf)* supports input operations. |
   | 可写                   | out             | **out**put   | File open for writing: the *[internal stream buffer](https://legacy.cplusplus.com/ifstream::rdbuf)* supports output operations. |
   | 二进制                 | binary          | **binary**   | Operations are performed in binary mode rather than text.    |
   | 追加                   | ate             | **at e**nd   | The *output position* starts at the end of the file.         |
   | 追加<br>(不可更改前文) | app             | **app**end   | All output operations happen at the end of the file, appending to its existing contents. |
   | 覆盖                   | trunc           | **trunc**ate | Any contents that existed in the file before it is open are discarded. |



#### 独特接口：

1. 打开/关闭 文件：`fs.open(filename, ...);`	/	`fs.close()`;
2. 判断文件是否打开：`fs.is_open()`