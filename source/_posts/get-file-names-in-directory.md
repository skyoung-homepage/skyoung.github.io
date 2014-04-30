title: 利用Boost获取文件夹下所有文件的文件名
date: 2014-04-30 09:58:12
tags: [Boost, 获取文件名]
categories: [Tips]
---

之前在做视频跟踪的时候，读取视频主要有三种方式，一种是直接从摄像头读取视频，一种是直接读取视频格式的文件，第三种就是读取图像序列，即一系列的图片。其中第三种方式需要获取文件夹下所有图像文件的文件名，虽然这些文件名称是顺序命名的，但有时候不同数据集提供的命名方式不同，导致读取很不方便，所以这就需要一种通用的读取方式，即先获取文件夹下所有图像文件的名称，然后排序名称，最后按照排好序的名称逐帧读取图像。 这里就用到了Boost库中的[Filesystem](http://www.boost.org/doc/libs/1_55_0/libs/filesystem/doc/index.htm)。它的优势在于可移植性，windows和linux下都是可以的，不需要针对不同系统的单独编写程序。

闲话少说，先贴一个实例程序：

```
#include <iostream>
#include <string>
#include <boost/filesystem.hpp>
#include <opencv2/opencv.hpp>

using namespace boost::filesystem;
using namespace std;
using namespace cv;

void getFileNames(FileStorage configFile,  vector<string> &fileNames)
{
    path inputPath ((string)configFile["path"]);
    try
    {
        if (exists(inputPath))
        {
            if (is_regular_file(inputPath))
            {
                cout << inputPath << " is not a directory, its size is " 
		<< file_size(inputPath) << '\n';
            }
            else if (is_directory(inputPath))
            {
                typedef vector<path> VecPath;
                VecPath paths;

                copy(directory_iterator(inputPath), directory_iterator(), back_inserter(paths));
                sort(paths.begin(), paths.end());

                for(VecPath::const_iterator it = paths.begin(); it != paths.end(); it++)
                {
                    string fileNam = it->filename().string();
                    if(fileNam.find("."+(string)configFile["fileFormat"]) != string::npos)
                        fileNames.push_back(fileNam);
                    //cout<< *it<<endl;
                }
            }
            else
                cout << inputPath << " exists, but is neither a regular file nor a directory\n";
        }
        else
            cout << inputPath << " does not exist\n";
  }

  catch (const filesystem_error& ex)
  {
        cout << ex.what() << '\n';
  }
}

int main()
{
    FileStorage configFile("FileInfo.yml", FileStorage::READ);
    vector<string> fileNames;
    getFileNames(configFile, fileNames);

    for(size_t i = 0; i < fileNames.size(); i++)
        cout<<fileNames[i]<<endl;

    return 0;
}

```

Filesystem主要是提供了一个`path`类，如示例程序12行所示，初始化一个`path`类的对象inputPath，第15行使用`exists()`判断输入的路径是否存在，17行使用`is_regular_file()`判断是否是文件，22行使用`is_directory()`判断是否是文件夹。28行使用`directory_iterator`迭代器读取路径下所有的文件的`path`，其中`path`类下对应的`filename()`成员函数对应`string`类型的文件名。


注意这里用到了OpenCV的FileStorage类来读取yml文件的配置信息(程序第53行)，主要包括路径和文件格式(可参考另一篇博客[OpenCV读取XML/YML文件的方法](http://skyoung.github.io/2014/04/25/write-and-read-xml-yml-file/))，FileInfo.yml存储的内容如下：

	%YAML:1.0
	path: ./
	fileFormat: yml

更多关于Filesystem的一些操作及应用可以参考[Filesystem Reference](http://www.boost.org/doc/libs/1_55_0/libs/filesystem/doc/reference.html)。

