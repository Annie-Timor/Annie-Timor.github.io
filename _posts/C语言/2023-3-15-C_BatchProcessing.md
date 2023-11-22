---
title: C语言批量处理
author: lixinghui
date: 2023-2-20 12:00:00 +0800
categories: [C语言, Function]
tags: [C语言]
---


在测试算法的效果的时，很多时候一个文件的测试并不是很准确，就需要一批文件来做测试，但是对于一批文件如果手动的一个个输入势必非常麻烦，能够使用一些既有的小工具来做就会简单很多，这里基于C语言的读取文件夹内的所需内容及`system`函数就可以做到批量处理了。

## 使用python进行处理

`2023.4.8更新：使用python工具会方便很多，如下：`

```python
#coding=utf-8
import os

folder_in = 'F:\TEST_WAVE\AddNoise'  # 输入文件夹
folder_out = 'F:\TEST_WAVE\Tpython'  #输出文件夹

# os.chdir(folder_in)
count = 0
executable_file = '.\main.exe'

for filename in os.listdir(folder_in):
    if filename.endswith('.wav'):
        # abs_path = os.path.join(os.getcwd(), filename)
        file_in = os.path.join(folder_in, filename)
        file_out = os.path.join(folder_out, filename)
	    # 这里的命令根据需要执行的文件参数设定
        os.system(f'{executable_file} {file_in} {file_out}')
        count += 1
        if count == 10:
            break

```



## 使用C语言进行处理

参考代码如下：

```c
/*Read a file with a fixed suffix*/

#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NUM 200  /*dir file number*/
#define MAX_CHAR 200 /*max file name length */

char file_in[MAX_NUM][MAX_CHAR] = {0};  /*The specified file within the folder*/
char file_out[MAX_NUM][MAX_CHAR] = {0}; /*The write file within the folder*/
int file_cnt = 0;                       /*quantity of file*/

void MyFileRecursive(char *in_path, char *out_path);

int main(int argc, char *argv[])
{
    /*Folder Name*/
    char folder_in[MAX_NUM] = {0};
    char folder_out[MAX_NUM] = {0};

    char exe[100] = "stft.exe"; /*executable file name*/
    char in_params[2 * MAX_CHAR] = {0};

    if (argc < 2)
    {
        strcpy(folder_in, "F:/TEST_WAVE/voice_lib_1");
        strcpy(folder_out, "F:/TEST_WAVE/test_0315");
    }
    else
    {
        strcpy(folder_in, argv[1]);
        strcpy(folder_out, argv[2]);
    }

    printf("user batch process folder is:%s\n", folder_in);
    printf("user batch process output is:%s\n\n", folder_out);

    MyFileRecursive(folder_in, folder_out);

    for (int i = 0; i < file_cnt; i++)
    {
        // printf("%s\n", file_in[i]);
        // printf("%s\n", file_out[i]);
        /*stft.exe in_file out_file*/
        memset(in_params, 0, sizeof(in_params)); // clear string
        strcpy(in_params, exe);
        strcat(in_params, " ");
        strcat(in_params, file_in[i]);
        strcat(in_params, " ");
        strcat(in_params, file_out[i]);

        // printf("%s\n", in_params);
        system(in_params);
    }

    return 0;
}

void MyFileRecursive(char *in_path, char *out_path)
{
    /*file path*/
    char path[MAX_CHAR];
    struct dirent *dp;
    DIR *dir = opendir(in_path);

    /*If it is a file, end the call directly*/
    if (!dir)
    {
        return;
    }

    while ((dp = readdir(dir)) != NULL)
    {
        // skip "." and ".."
        if (strcmp(dp->d_name, ".") != 0 && strcmp(dp->d_name, "..") != 0 && strstr(dp->d_name, ".wav"))
        {
            /*Concatenate the file name and path to form a complete path*/
            memset(path, 0, sizeof(path));
            strcpy(path, out_path);
            strcat(path, "/");
            strcat(path, dp->d_name);
            // sprintf(file_out[file_cnt], "%s", path);
            memcpy(file_out[file_cnt], path, sizeof(path));

            memset(path, 0, sizeof(path));
            strcpy(path, in_path);
            strcat(path, "/");
            strcat(path, dp->d_name);
            // sprintf(file_in[file_cnt], "%s", path);
            memcpy(file_in[file_cnt], path, sizeof(path));
            file_cnt++;
            /*Recursive Traversal*/
            MyFileRecursive(path, out_path);
        }
    }
    closedir(dir);
}

```



## 扫描文件夹内的指定文件

读取文件夹中的对应文件为：

其中`MyFileRecursive`函数依赖`#include <dirent.h>`头文件，`TraverseFolder`函数依赖于`#include <windows.h>`

```c
#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <windows.h>

#define MAX_NUM 100  /*dir file number*/
#define MAX_CHAR 100 /*max file name length */

char file_in[MAX_NUM][MAX_CHAR] = {0}; /*The specified file within the folder*/
int file_cnt = 0;                      /*quantity of file*/

void MyFileRecursive(char *folder, char (*file)[MAX_CHAR])
{
    /*file path*/
    char path[MAX_CHAR];
    struct dirent *dp;
    DIR *dir = opendir(folder);

    /*If it is a file, end the call directly*/
    if (!dir)
    {
        return;
    }

    while ((dp = readdir(dir)) != NULL)
    {
        // skip "." and ".."
        if (strcmp(dp->d_name, ".") != 0 && strcmp(dp->d_name, "..") != 0 && strstr(dp->d_name, ".wav"))
        {
            /*Concatenate the file name and path to form a complete path*/
            //  / and \\ are ok
            sprintf(file[file_cnt], "%s\\%s", folder, dp->d_name);
            file_cnt++;
            /*Recursive Traversal*/
            MyFileRecursive(path, file);
        }
    }
    closedir(dir);
}

int TraverseFolder(char *folder, char (*file)[MAX_CHAR])
{
    WIN32_FIND_DATA fileData;
    HANDLE hFind;
    char path[MAX_PATH] = {0};
    strcpy(path, folder);
    strcat(path, "\\*.wav");

    hFind = FindFirstFile(path, &fileData);
    if (hFind != INVALID_HANDLE_VALUE)
    {
        do
        {
            if (!(fileData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY))
            {
                sprintf(file[file_cnt], "%s\\%s", folder, fileData.cFileName);
                file_cnt++;
            }
        } while (FindNextFile(hFind, &fileData) != 0);
        FindClose(hFind);
    }
    return 0;
}

int main(int argc, char **argv)
{
    char *folder = "F:\\TEST_WAVE\\PMI_TEST\\WAV_IN";
    TraverseFolder(folder, file_in);
    // MyFileRecursive(folder, file_in);
    for (int i = 0; i < file_cnt; i++)
    {
        printf("%s\n", file_in[i]);
    }

    return 0;
}

```

