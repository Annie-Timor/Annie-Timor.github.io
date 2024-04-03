---
title: WAV格式详解
author: lixinghui
date: 2023-2-10 12:00:00 +0800
categories: [Speech, WAV]
tags: [语音杂谈]
---


> 本文由简悦 SimpRead转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904051964903431)
{: .prompt-tip }

概述
--

  WAV 即 WAVE，是经典的 Windows 音频数据封装格式，由 Microsoft 开发。数据本身格式为 PCM，也可以支持一些编码格式的数据，比如最近流行的 AAC 编码。如果是 PCM，则为无损格式，文件会比较大，并且大小相对固定，可以使用以下公式计算文件大小。

```
FileSize = HeadSize + TimeInSecond * SampleRate * Channels * BitsPerSample / 8
FileSize = HeadSize + DataChunkSize
```

> 其中 HeadSize，为 WAV 文件头部长度；
>
> TimeInSecond，为音频时长
>
> SampleRate，为采样率，可选 8000、16000、32000、44100 或 48000；
>
> Channels，表示声道数量，通常为 1 或 2；
>
> BitsPerSample，代表单个 Sample 的位深，可选 8、16 以及 32，其中 32 位时可以是 float 类型。 
>
> 例如一个WAV文件，FileSize=274044,ChunkSize=274036, DataChunkSize=274000。音频时长为8.5625秒，采样率为16000，声道数为单声道，位深为16bit
>
> 计算为：274044=44+8.5625\*16000\*1\*16/8=274044

WAV 是一种极其简单的文件格式，如果对其结构足够熟悉，完全可以自己通过代码写入 WAV 文件，从而免去引入一些复杂中间库。特别是在对音频进行调试的时候，能提高效率，降低复杂度。   

WAV 格式遵循 RIFF 规范，所有 WAV 都有一个文件头，记录着音频流的采样和编码信息。数据块的记录方式是小尾端 (little-endian)。

RIFF
----

  RIFF，全称 Resource Interchange File Format，是一种按照标记区块存储数据的通用文件存储格式，多用于存储音频、视频等多媒体数据。Microsoft 在 Windows 下的 WAV、AVI 等都是基于 RIFF 实现的。 一个标准的 RIFF 规范规范文件，最小存储单位为 “块”(Chunk)，每个块(Chunk) 包含以下三个信息：

<table><thead><tr><th>名称</th><th>大小</th><th>类型</th><th>端序</th><th>含义</th></tr></thead><tbody><tr><td>FOURCC</td><td>4</td><td>字符</td><td>大端</td><td>用于标识 Chunk ID 或 chunk 类型，通常为 Chunk ID</td></tr><tr><td>Data Field Size</td><td>4</td><td>整形</td><td>小端</td><td>特别注意，该长度不包含其本身，以及 FOURCC</td></tr><tr><td>Data Field</td><td>-</td><td>-</td><td>-</td><td>数据域，如果 Chunk ID 为 "RIFF" 或 "LIST"，则开始四个字节为类型码</td></tr></tbody></table>

  只有 ID 为 "RIFF" 或者 "LIST" 的块允许拥有子块 (SubChunk)。RIFF 文件的第一个块的 ID 必须是 "RIFF"，也就是说 ID 为 "LIST" 的块只能是子块 (SubChunk)，他们和各个子块形成了复杂的 RIFF 文件结构。RIFF 数据域的的起始位置四个字节为类型码 (Form Type)，用于说明数据域的格式，比如 WAV 文件的类型码为 "WAVE"。 "LIST" 块的数据域的起始位置也有一个四字节类型码 (List Type)，用于说明 LIST 数据域的数据内容。比如，类型码为 "INFO" 时，其数据域可能包括 "ICOP"、"ICRD" 块，用于记录文件版权和创建时间信息。

WAV
---

  以最简单的无损 WAV 格式文件为例，此时文件的音频数据部分为 PCM，比较简单，重点在于 WAV 头部。一个典型的 WAV 文件头部长度为 44 字节，包含了采样率，通道数，位深等信息，如下表所示。

<table><thead><tr><th>偏移位置</th><th>大小</th><th>类型</th><th>端序</th><th>含义</th></tr></thead><tbody><tr><td>0x00-0x03</td><td>4</td><td>字符</td><td>大端</td><td>"RIFF" 块 (0x52494646)，标记为 RIFF 文件格式</td></tr><tr><td>0x04-0x07</td><td>4</td><td>整型</td><td>小端</td><td>块数据域大小（Chunk Size），即从下一个地址开始，到文件末尾的总字节数，或者文件总字节数 - 8。从 0x08 开始一直到文件末尾，都是 ID 为 "RIFF" 块的内容，其中会包含两个子块，"fmt" 和 "data"</td></tr><tr><td>0x08-0x0B</td><td>4</td><td>字符</td><td>大端</td><td>类型码 (Form Type)，WAV 文件格式标记，即 "WAVE" 四个字母</td></tr><tr><td>0x0C-0x0F</td><td>4</td><td>字符</td><td>大端</td><td>"fmt" 子块 (0x666D7420)，注意末尾的空格</td></tr><tr><td>0x10-0x13</td><td>4</td><td>整形</td><td>小端</td><td>子块数据域大小（SubChunk Size）定义了音频的编码格式</td></tr><tr><td>0x14-0x15</td><td>2</td><td>整形</td><td>小端</td><td>编码格式 (Audio Format)，1 代表 PCM 无损格式</td></tr><tr><td>0x16-0x17</td><td>2</td><td>整形</td><td>小端</td><td>声道数 (Channels)，1 或 2</td></tr><tr><td>0x18-0x1B</td><td>4</td><td>整形</td><td>小端</td><td>采样率 (Sample Rate)</td></tr><tr><td>0x1C-0x1F</td><td>4</td><td>整形</td><td>小端</td><td>传输速率 (Byte Rate)，每秒数据字节数，SampleRate * Channels * BitsPerSample / 8</td></tr><tr><td>0x20-0x21</td><td>2</td><td>整形</td><td>小端</td><td>每个采样所需的字节数 BlockAlign，BitsPerSample*Channels/8</td></tr><tr><td>0x22-0x23</td><td>2</td><td>整形</td><td>小端</td><td>单个采样位深 (Bits Per Sample)，可选 8、16 或 32</td></tr><tr><td>0x24-0x27</td><td>4</td><td>字符</td><td>大端</td><td>"data" 子块 (0x64617461)</td></tr><tr><td>0x28-0x2B</td><td>4</td><td>整形</td><td>小端</td><td>子块数据域大小（SubChunk Size）</td></tr><tr><td>0x2C-eos</td><td>N</td><td></td><td></td><td>PCM</td></tr></tbody></table>

  上表为典型的 WAV 头部格式，从 0x00 到 0x2B 总共 44 字节，从 0x2C 开始一直到文件末尾都是 PCM 音频数据。所以如果你已经知道了 PCM 的采样信息，那么可以直接跳过头部的解析，直接从 0x2C 开始读取 PCM 即可，但是对于另一些无损的 WAV 文件却是不行的。

WAV 扩展
------

  有一些 WAV 的头部并不仅仅只有 44 个字节，比如通过 FFmpge 编码而来的 WAV 文件头部信息通常大于 44 个字节。这是因为根据 WAV 规范，其头部还支持携带附加信息，所以只按照 44 个字节的长度去解析 WAV 头部信息是不一定正确的，还需要考虑附加信息。那么如何知道一个 WAV 文件头部是否包含附加信息呢？根据 "fmt" 子块长度来判断即可。

> 如果 fmt SubChunk Size 等于 0x10(16)，表示头部不包含附加信息，即 WAV 头部信息长度为 44；如果等于 0x12(18)，则包含附加信息，此时头部信息长度大于 44。

  当 WAV 头部包含附加信息时，fmt SubChunk Size 长度为 18，并且紧随是另一个子块，这个包含了一些自定义的附加信息，接着往下才是 "data" 子块，格式如下：

<table><thead><tr><th>偏移位置</th><th>大小</th><th>类型</th><th>端序</th><th>含义</th></tr></thead><tbody><tr><td>0x00-0x03</td><td>4</td><td>字符</td><td>大端</td><td>"RIFF" 块 (0x52494646)，标记为 RIFF 文件格式</td></tr><tr><td>0x04-0x07</td><td>4</td><td>整型</td><td>小端</td><td>块数据域大小（Chunk Size），即从下一个地址开始，到文件末尾的总字节数，或者文件总字节数 - 8。从 0x08 开始一直到文件末尾，都是 ID 为 "RIFF" 块的内容，其中会包含两个子块，"fmt" 和 "data"</td></tr><tr><td>0x08-0x0B</td><td>4</td><td>字符</td><td>大端</td><td>类型码 (Form Type)，WAV 文件格式标记，即 "WAVE" 四个字母</td></tr><tr><td>0x0C-0x0F</td><td>4</td><td>字符</td><td>大端</td><td>"fmt" 子块 (0x666D7420)，注意末尾的空格</td></tr><tr><td>0x10-0x13</td><td>4</td><td>整形</td><td>小端</td><td>子块数据域大小（SubChunk Size），这里为 0x12</td></tr><tr><td>0x14-0x15</td><td>2</td><td>整形</td><td>小端</td><td>编码格式 (Audio Format)，1 代表 PCM 无损格式</td></tr><tr><td>0x16-0x17</td><td>2</td><td>整形</td><td>小端</td><td>声道数 (Channels)，1 或 2</td></tr><tr><td>0x18-0x1B</td><td>4</td><td>整形</td><td>小端</td><td>采样率 (Sample Rate)</td></tr><tr><td>0x1C-0x1F</td><td>4</td><td>整形</td><td>小端</td><td>传输速率 (Byte Rate)，每秒数据字节数，SampleRate * Channels * BitsPerSample / 8</td></tr><tr><td>0x20-0x21</td><td>2</td><td>整形</td><td>小端</td><td>每个采样所需的字节数 BlockAlign，BitsPerSample*Channels/8</td></tr><tr><td>0x22-0x23</td><td>2</td><td>整形</td><td>小端</td><td>单个采样位深 (Bits Per Sample)，可选 8、16 或 32</td></tr><tr><td>0x24-0x25</td><td>2</td><td></td><td></td><td></td></tr><tr><td>0x26 - 不定</td><td>-</td><td>-</td><td>-</td><td>可选附加信息，标准 RIFF Chunk</td></tr><tr><td>不定</td><td>4</td><td>字符</td><td>大端</td><td>"data" 子块 (0x64617461)</td></tr><tr><td>不定</td><td>4</td><td>整形</td><td>小端</td><td>子块数据域大小（SubChunk Size）</td></tr><tr><td>不定</td><td>N</td><td></td><td></td><td>PCM</td></tr></tbody></table>

  如果一个无损 WAV 文件头部包含了附加信息，那么 PCM 音频所在的位置就不确定了，但由于附加信息也是一个子块 (SubChunk)，根据 RIFF 规范，该子块也必然记录着其长度信息，所以我们还是有办法能够动态计算出其位置，下面是计算步骤：

1.  判断 fmt 块长度是否为 18。
2.  如果 fmt 长度为 18，那么必然从 0x26 位置开始为附加信息块，0x30-0x33 位置记录着该子块长度。
3.  根据步骤 2 获取的子块长度，假定为 N(16 进制)，那么 PCM 音频信息开始位置为：0x34 + N + 8。   以上步骤仅为逻辑推理得出，未经验证，但大致遵循以上步骤，如有错误，欢迎指正。

## C语言参考代码

> 以下代码为自行添加，非原文内容。仅是最简单的short型wav文件读取，如输入的文件不是该类型，请修改相关代码。

**wav_pcm16.h**

```c
#ifndef WAV_PCM16_H
#define WAV_PCM16_H

#include <stdio.h>

#define PRINT_WAV_HEADER 1
#define WAVE_HEADER_SIZE 44

#define ONLY_FOR_LITTLE_ENDIAN

/*little endian*/
typedef struct
{
    /*file_size=chunk_size+8=sub_chunk_size+44*/
    /*fmt_chunk_size=0x10,defines format information
    for audio data at address 0x14~0x23*/
    char file_format[4];   // "RIFF"
    int chunk_size;        // =sub_chunk_size+36
    char form_type[4];     // "WAVE"
    char fmt[4];           // "fmt "
    int fmt_size;          // fmt chunk size=0x10
    short audio_format;    // 01=PCM
    short channels;        // channel number,1/2
    int sample_rate;       // sample rate,8000/16000/44100
    int byte_rate;         // SampleRate * Channels * BitsPerSample / 8
    short byte_per_sample; // BitsPerSample*Channels/8
    short bit_per_sample;  // 8/16/32
    char data[4];          // "data"
    int sub_chunk_size;    // =file_size-44
} WAV_HEADER;

int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size);
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_ReadBuffer(FILE *fp_in, short *buf, int n);
short *WAV_ReadAll(FILE *fp_in, int *n);
short *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n);
int WAV_WriteBuffer(FILE *fp_in, const short *buf, int n);
#endif // !WAV_READ_H

```

**wav_pcm16.c**

```c
#include "wav_pcm16.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * @brief Analyze header file contents
 *
 * @param fp_in :input file point
 * @param h : wav head struct point
 * @return -1:input is null,0:success
 */
int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle error
        return -1;
    }

    fseek(fp_in, 0, SEEK_SET);
    /*both the little endian~*/
    fread(h, WAVE_HEADER_SIZE, 1, fp_in);

#if PRINT_WAV_HEADER
    printf("---------------------------------------------\n");
    printf("----------------Analyze start----------------\n");
    printf("1-File fomat:%.4s\n", h->file_format);
    printf("2-chunk size:%d bytes\n", h->chunk_size);
    printf("3-Form Type:%.4s\n", h->form_type);
    printf("4-%.4s size:", h->fmt);
    printf("%d bytes\n", h->fmt_size);
    printf("5-Audio Format:%s\n", h->audio_format == 1 ? "PCM" : "other");
    printf("6-Channels:%d\n", h->channels);
    printf("7-Sample Rate:%d\n", h->sample_rate);
    printf("8-Byte Rate(SampleRate * Channels * BitsPerSample / 8):%d\n", h->byte_rate);
    printf("9-Bytes Per Sample:%d\n", h->byte_per_sample);
    printf("10-Bits Per Sample:%d\n", h->bit_per_sample);
    printf("11-%.4s:", h->data);
    printf("%d bytes\n", h->sub_chunk_size);
    printf("-----------------Analyze end-----------------\n");
    printf("---------------------------------------------\n");
#endif

    return 0;
}

/**
 * @brief fill the wav header
 *
 * @param h : the wav head struct point
 * @param sample_rate : sample_rate
 * @param channels : number of channels
 * @param data_size : number of data count,not the bytes
 * @return 0 on success, -1 on failure
 */
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size)
{
    if (h == NULL)
    {
        // !handle empty
        return -1;
    }

    unsigned int total_size = data_size * 2;
    memcpy(h->file_format, "RIFF", 4);
    h->chunk_size = total_size + 36;
    memcpy(h->form_type, "WAVE", 4);
    memcpy(h->fmt, "fmt ", 4);
    h->fmt_size = 16;
    h->audio_format = 1;
    h->channels = channels;
    h->sample_rate = sample_rate;
    h->byte_rate = sample_rate * channels * 2;
    h->byte_per_sample = 2;
    h->bit_per_sample = 16;
    memcpy(h->data, "data", 4);
    h->sub_chunk_size = total_size;

    return 0;
}

/**
 * @brief write the header to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param h :the input WAV head
 * @return int -1:fp_in or head is NULL,-2:write error,0:correct
 */
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h)
{
    int ret = -2;
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty input
        return -1;
    }
    fseek(fp_in, 0, SEEK_SET);
    if (WAVE_HEADER_SIZE == fwrite(h, WAVE_HEADER_SIZE, 1, fp_in))
    {
        ret = 0;
    }

    return ret;
}

/**
 * @brief read N buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the buffer to read
 * @param n :N
 * @return int -1:read error,0:correct
 */
int WAV_ReadBuffer(FILE *fp_in, short *buf, int n)
{
    int read_size = 0;

    read_size = fread(buf, sizeof(short), n, fp_in);
    if (read_size != n)
    {
        return -1;
    }
    return 0;
}

#if 1
/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param n :the input WAV file buffers number
 * @return short* return the data
 */
short *WAV_ReadAll(FILE *fp_in, int *n)
{
    unsigned char head[4] = {0};
    unsigned int size = 0;
    unsigned char *audio_read = NULL;
    fseek(fp_in, WAVE_HEADER_SIZE - 4, SEEK_SET);
    fread(head, sizeof(char), 4, fp_in);
    size = head[0] + head[1] * 256 + head[2] * 256 * 256 + head[3] * 256 * 256 * 256;
    audio_read = (unsigned char *)malloc(size);
    memset(audio_read, 0, size);
    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);
    if (fread(audio_read, sizeof(char), size, fp_in) != size)
    {
        free(audio_read);
        return NULL;
    }
    *n = size / 2;
    return ((short *)audio_read);
}
#endif

/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in the input WAV file ponit
 * @param h the wav file head point
 * @param n the wav buffer read (short type)
 * @return short* data buffer or NULL
 */
short *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty
        return NULL;
    }

    fseek(fp_in, 0, SEEK_END);
    long file_size = ftell(fp_in);
    long audio_size = file_size - WAVE_HEADER_SIZE;
    long num_sample = audio_size / (h->channels * (h->bit_per_sample / 8));
    fseek(fp_in, 0, SEEK_SET);
    short *buf = (short *)malloc(num_sample * sizeof(short));
    if (fread(buf, sizeof(short), num_sample, fp_in) != num_sample)
    {
        free(buf);
        return NULL;
    }
    *n = num_sample;
    return buf;
}

/**
 * @brief write data to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the wav data
 * @param n :data number
 * @return int 0 on success, -1 on failure
 */
int WAV_WriteBuffer(FILE *fp_in, const short *buf, int n)
{
    if ((fp_in == NULL) || (buf == NULL))
    {
        // !handle empty input
        return -1;
    }
    fwrite(buf, sizeof(short), n, fp_in);
    return 0;
}
```

**wav_pcm24.h**

```c
#ifndef WAV_PCM24_H
#define WAV_PCM24_H

#include <stdio.h>

#define PRINT_WAV_HEADER 1
#define WAVE_HEADER_SIZE 44

#define ONLY_FOR_LITTLE_ENDIAN

/*little endian*/
typedef struct
{
    /*file_size=chunk_size+8=sub_chunk_size+44*/
    /*fmt_chunk_size=0x10,defines format information
    for audio data at address 0x14~0x23*/
    char file_format[4];   // "RIFF"
    int chunk_size;        // =sub_chunk_size+36
    char form_type[4];     // "WAVE"
    char fmt[4];           // "fmt "
    int fmt_size;          // fmt chunk size=0x10
    short audio_format;    // 01=PCM
    short channels;        // channel number,1/2
    int sample_rate;       // sample rate,8000/16000/44100
    int byte_rate;         // SampleRate * Channels * BitsPerSample / 8
    short byte_per_sample; // BitsPerSample*Channels/8
    short bit_per_sample;  // 8/16/32
    char data[4];          // "data"
    int sub_chunk_size;    // =file_size-44
} WAV_HEADER;

int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size);
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_ReadBuffer(FILE *fp_in, int *buf, int n);
short *WAV_ReadAll(FILE *fp_in, int *n);
short *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n);
int WAV_WriteBuffer(FILE *fp_in, const int *buf, int n);
#endif // !WAV_READ_H
```

**wav_pcm24.c**

```c
#include "wav_pcm24.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct
{
    int value : 24;
} Int24;

/**
 * @brief Analyze header file contents
 *
 * @param fp_in :input file point
 * @param h : wav head struct point
 * @return -1:input is null,0:success
 */
int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle error
        return -1;
    }

    fseek(fp_in, 0, SEEK_SET);
    /*both the little endian~*/
    fread(h, WAVE_HEADER_SIZE, 1, fp_in);

#if PRINT_WAV_HEADER
    printf("---------------------------------------------\n");
    printf("----------------Analyze start----------------\n");
    printf("1-File fomat:%.4s\n", h->file_format);
    printf("2-chunk size:%d bytes\n", h->chunk_size);
    printf("3-Form Type:%.4s\n", h->form_type);
    printf("4-%.4s size:", h->fmt);
    printf("%d bytes\n", h->fmt_size);
    printf("5-Audio Format:%s\n", h->audio_format == 1 ? "PCM" : "other");
    printf("6-Channels:%d\n", h->channels);
    printf("7-Sample Rate:%d\n", h->sample_rate);
    printf("8-Byte Rate(SampleRate * Channels * BitsPerSample / 8):%d\n", h->byte_rate);
    printf("9-Bytes Per Sample:%d\n", h->byte_per_sample);
    printf("10-Bits Per Sample:%d\n", h->bit_per_sample);
    printf("11-%.4s:", h->data);
    printf("%d bytes\n", h->sub_chunk_size);
    printf("-----------------Analyze end-----------------\n");
    printf("---------------------------------------------\n");
#endif

    return 0;
}

/**
 * @brief fill the wav header
 *
 * @param h : the wav head struct point
 * @param sample_rate : sample_rate
 * @param channels : number of channels
 * @param data_size : number of data count,not the bytes
 * @return 0 on success, -1 on failure
 */
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size)
{
    if (h == NULL)
    {
        // !handle empty
        return -1;
    }

    unsigned int total_size = data_size * 3;
    memcpy(h->file_format, "RIFF", 4);
    h->chunk_size = total_size + 36;
    memcpy(h->form_type, "WAVE", 4);
    memcpy(h->fmt, "fmt ", 4);
    h->fmt_size = 16;
    h->audio_format = 1;
    h->channels = channels;
    h->sample_rate = sample_rate;
    h->byte_rate = sample_rate * channels * 24 / 8;
    h->byte_per_sample = 3;
    h->bit_per_sample = 24;
    memcpy(h->data, "data", 4);
    h->sub_chunk_size = total_size;

    return 0;
}

/**
 * @brief write the header to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param h :the input WAV head
 * @return int -1:fp_in or head is NULL,-2:write error,0:correct
 */
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h)
{
    int ret = -2;
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty input
        return -1;
    }
    fseek(fp_in, 0, SEEK_SET);
    if (WAVE_HEADER_SIZE == fwrite(h, WAVE_HEADER_SIZE, 1, fp_in))
    {
        ret = 0;
    }

    return ret;
}

/**
 * @brief read N buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the buffer to read
 * @param n :N
 * @return int -1:read error,0:correct
 */
int WAV_ReadBuffer(FILE *fp_in, int *buf, int n)
{
    int i;
    int read_data = 0;
    Int24 rd = {0};
    for (i = 0; i < n; i++)
    {
        if (fread(&rd, 3, 1, fp_in) != 1)
        {
            break;
        }
        
        buf[i] = rd.value;
    }

    if (i != n)
    {
        return -1;
    }
    return 0;
}

#if 0
/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param n :the input WAV file buffers number
 * @return short* return the data
 */
short *WAV_ReadAll(FILE *fp_in, int *n)
{
    unsigned char head[4] = {0};
    unsigned int size = 0;
    unsigned char *audio_read = NULL;
    fseek(fp_in, WAVE_HEADER_SIZE - 4, SEEK_SET);
    fread(head, sizeof(char), 4, fp_in);
    size = head[0] + head[1] * 256 + head[2] * 256 * 256 + head[3] * 256 * 256 * 256;
    audio_read = (unsigned char *)malloc(size);
    memset(audio_read, 0, size);
    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);
    if (fread(audio_read, sizeof(char), size, fp_in) != size)
    {
        free(audio_read);
        return NULL;
    }
    *n = size / 2;
    return ((short *)audio_read);
}


/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in the input WAV file ponit
 * @param h the wav file head point
 * @param n the wav buffer read (short type)
 * @return short* data buffer or NULL
 */
short *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty
        return NULL;
    }

    fseek(fp_in, 0, SEEK_END);
    long file_size = ftell(fp_in);
    long audio_size = file_size - WAVE_HEADER_SIZE;
    long num_sample = audio_size / (h->channels * (h->bit_per_sample / 8));
    fseek(fp_in, 0, SEEK_SET);
    short *buf = (short *)malloc(num_sample * sizeof(short));
    if (fread(buf, sizeof(short), num_sample, fp_in) != num_sample)
    {
        free(buf);
        return NULL;
    }
    *n = num_sample;
    return buf;
}
#endif

/**
 * @brief write data to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the wav data
 * @param n :data number
 * @return int 0 on success, -1 on failure
 */
int WAV_WriteBuffer(FILE *fp_in, const int *buf, int n)
{
    if ((fp_in == NULL) || (buf == NULL))
    {
        // !handle empty input
        return -1;
    }
    Int24 wd = {0};
    for (int i = 0; i < n; i++)
    {
        wd.value = buf[i];
        fwrite(&wd, 3, 1, fp_in);
    }
    return 0;
}
```

**wav_pcm32.h**

```c
#ifndef WAV_PCM32_H
#define WAV_PCM32_H

#include <stdio.h>

#define PRINT_WAV_HEADER 1
#define WAVE_HEADER_SIZE 44

#define ONLY_FOR_LITTLE_ENDIAN

/*little endian*/
typedef struct
{
    /*file_size=chunk_size+8=sub_chunk_size+44*/
    /*fmt_chunk_size=0x10,defines format information
    for audio data at address 0x14~0x23*/
    char file_format[4];   // "RIFF"
    int chunk_size;        // =sub_chunk_size+36
    char form_type[4];     // "WAVE"
    char fmt[4];           // "fmt "
    int fmt_size;          // fmt chunk size=0x10
    short audio_format;    // 01=PCM
    short channels;        // channel number,1/2
    int sample_rate;       // sample rate,8000/16000/44100
    int byte_rate;         // SampleRate * Channels * BitsPerSample / 8
    short byte_per_sample; // BitsPerSample*Channels/8
    short bit_per_sample;  // 8/16/32
    char data[4];          // "data"
    int sub_chunk_size;    // =file_size-44
} WAV_HEADER;

int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size);
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h);
int WAV_ReadBuffer(FILE *fp_in, int *buf, int n);
int *WAV_ReadAll(FILE *fp_in, int *n);
int *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n);
int WAV_WriteBuffer(FILE *fp_in, const int *buf, int n);
#endif // !WAV_READ_H
```

**wav_pcm32.c**

```c
#include "wav_pcm32.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * @brief Analyze header file contents
 *
 * @param fp_in :input file point
 * @param h : wav head struct point
 * @return -1:input is null,0:success
 */
int WAV_AnalyzeHeader(FILE *fp_in, WAV_HEADER *h)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle error
        return -1;
    }

    fseek(fp_in, 0, SEEK_SET);
    /*both the little endian~*/
    fread(h, WAVE_HEADER_SIZE, 1, fp_in);

#if PRINT_WAV_HEADER
    printf("---------------------------------------------\n");
    printf("----------------Analyze start----------------\n");
    printf("1-File fomat:%.4s\n", h->file_format);
    printf("2-chunk size:%d bytes\n", h->chunk_size);
    printf("3-Form Type:%.4s\n", h->form_type);
    printf("4-%.4s size:", h->fmt);
    printf("%d bytes\n", h->fmt_size);
    printf("5-Audio Format:%s\n", h->audio_format == 1 ? "PCM" : "other");
    printf("6-Channels:%d\n", h->channels);
    printf("7-Sample Rate:%d\n", h->sample_rate);
    printf("8-Byte Rate(SampleRate * Channels * BitsPerSample / 8):%d\n", h->byte_rate);
    printf("9-Bytes Per Sample:%d\n", h->byte_per_sample);
    printf("10-Bits Per Sample:%d\n", h->bit_per_sample);
    printf("11-%.4s:", h->data);
    printf("%d bytes\n", h->sub_chunk_size);
    printf("-----------------Analyze end-----------------\n");
    printf("---------------------------------------------\n");
#endif

    return 0;
}

/**
 * @brief fill the wav header
 *
 * @param h : the wav head struct point
 * @param sample_rate : sample_rate
 * @param channels : number of channels
 * @param data_size : number of data count,not the bytes
 * @return 0 on success, -1 on failure
 */
int WAV_FillHeader(WAV_HEADER *h, int sample_rate, int channels, int data_size)
{
    if (h == NULL)
    {
        // !handle empty
        return -1;
    }

    unsigned int total_size = data_size * 4;
    memcpy(h->file_format, "RIFF", 4);
    h->chunk_size = total_size + 36;
    memcpy(h->form_type, "WAVE", 4);
    memcpy(h->fmt, "fmt ", 4);
    h->fmt_size = 16;
    h->audio_format = 1;
    h->channels = channels;
    h->sample_rate = sample_rate;
    h->byte_rate = sample_rate * channels * 32 / 8;
    h->byte_per_sample = 4;
    h->bit_per_sample = 32;
    memcpy(h->data, "data", 4);
    h->sub_chunk_size = total_size;

    return 0;
}

/**
 * @brief write the header to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param h :the input WAV head
 * @return int -1:fp_in or head is NULL,-2:write error,0:correct
 */
int WAV_WriteHeader(FILE *fp_in, WAV_HEADER *h)
{
    int ret = -2;
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty input
        return -1;
    }
    fseek(fp_in, 0, SEEK_SET);
    if (WAVE_HEADER_SIZE == fwrite(h, WAVE_HEADER_SIZE, 1, fp_in))
    {
        ret = 0;
    }

    return ret;
}

/**
 * @brief read N buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the buffer to read
 * @param n :N
 * @return int -1:read error,0:correct
 */
int WAV_ReadBuffer(FILE *fp_in, int *buf, int n)
{
    int read_size = 0;

    read_size = fread(buf, sizeof(int), n, fp_in);
    if (read_size != n)
    {
        return -1;
    }

    return 0;
}

#if 1
/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param n :the input WAV file buffers number
 * @return short* return the data
 */
int *WAV_ReadAll(FILE *fp_in, int *n)
{
    unsigned char head[4] = {0};
    unsigned int size = 0;
    unsigned char *audio_read = NULL;
    fseek(fp_in, WAVE_HEADER_SIZE - 4, SEEK_SET);
    fread(head, sizeof(char), 4, fp_in);
    size = head[0] + head[1] * 256 + head[2] * 256 * 256 + head[3] * 256 * 256 * 256;
    audio_read = (unsigned char *)malloc(size);
    memset(audio_read, 0, size);
    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);
    if (fread(audio_read, sizeof(char), size, fp_in) != size)
    {
        free(audio_read);
        return NULL;
    }
    *n = size / 4;
    return ((int *)audio_read);
}

/**
 * @brief read all buffers from the input WAV file
 *
 * @param fp_in the input WAV file ponit
 * @param h the wav file head point
 * @param n the wav buffer read (short type)
 * @return short* data buffer or NULL
 */
int *WAV_ReadAll2(FILE *fp_in, WAV_HEADER *h, int *n)
{
    if ((fp_in == NULL) || (h == NULL))
    {
        // !handle empty
        return NULL;
    }

    fseek(fp_in, 0, SEEK_END);
    long file_size = ftell(fp_in);
    long audio_size = file_size - WAVE_HEADER_SIZE;
    long num_sample = audio_size / (h->channels * (h->bit_per_sample / 8));
    fseek(fp_in, 0, SEEK_SET);
    int *buf = (int *)malloc(num_sample * sizeof(int));
    if (fread(buf, sizeof(int), num_sample, fp_in) != num_sample)
    {
        free(buf);
        return NULL;
    }
    *n = num_sample;
    return buf;
}
#endif

/**
 * @brief write data to the input WAV file
 *
 * @param fp_in :the input WAV file ponit
 * @param buf :the wav data
 * @param n :data number
 * @return int 0 on success, -1 on failure
 */
int WAV_WriteBuffer(FILE *fp_in, const int *buf, int n)
{
    if ((fp_in == NULL) || (buf == NULL))
    {
        // !handle empty input
        return -1;
    }
    fwrite(buf, sizeof(int), n, fp_in);
    return 0;
}
```

**test_main.c**

```c
#include <math.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TEST_PCM16 0
#define TEST_PCM24 0
#define TEST_PCM32 1

#if TEST_PCM16
#include "wav_pcm16.h"
#endif

#if TEST_PCM24
#include "wav_pcm24.h"
#endif

#if TEST_PCM32
#include "wav_pcm32.h"
#endif

#define FRAME_SIZE 960
#define FS (48000)
#define CHANNELS (1)

#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

void pcm16_data_init(short *buffer, int len)
{
    for (int i = 0; i < len; i++)
    {
        buffer[i] = 1024 * sinf(2 * M_PI * 1000 * i / 48000);
    }
}

void pcm24_data_init(int *buffer, int len)
{
    for (int i = 0; i < len; i++)
    {
        buffer[i] = 1024 * 256 * sinf(2 * M_PI * 1000 * i / 48000);
    }
}

void pcm32_data_init(int *buffer, int len)
{
    for (int i = 0; i < len; i++)
    {
        buffer[i] = 1024 * 256 * 256 * sinf(2 * M_PI * 1000 * i / 48000);
    }
}

#if TEST_PCM16

int main(int argc, char **argv)
{
    char *file_in = NULL;
    char *file_out = NULL;
    FILE *fp_in = NULL;
    FILE *fp_out = NULL;

    /* only read 16bit integer */
    short *read_data = NULL;
    short *write_data = NULL;
    WAV_HEADER header = {0};
    int frame_cnt = 0;
    int n = 0;

    /*give the fixed parameter*/
    file_in = "TT.wav";
    file_out = "TT.wav";

    /*open the wav file*/
    fp_out = fopen(file_out, "wb");
    if (!fp_out)
    {
        printf("Error: Could not open file %s \n", file_out);
        return -1;
    }

    read_data = malloc(CHANNELS * FRAME_SIZE * sizeof(short));
    write_data = malloc(CHANNELS * FRAME_SIZE * sizeof(short));
    pcm16_data_init(write_data, CHANNELS * FRAME_SIZE);

    fseek(fp_out, WAVE_HEADER_SIZE, SEEK_SET);
    frame_cnt = 0;
    for (int i = 0; i < 10; i++)
    {

        WAV_WriteBuffer(fp_out, write_data, CHANNELS * FRAME_SIZE);
        frame_cnt++;
    }
    WAV_FillHeader(&header, FS, CHANNELS, frame_cnt * CHANNELS * FRAME_SIZE);
    WAV_WriteHeader(fp_out, &header);
    fclose(fp_out);

    fp_in = fopen(file_in, "rb");
    if (!fp_in)
    {
        printf("Error: Could not open file %s \n", file_in);
        return -1;
    }

    /*analyze the input wav head*/
    WAV_AnalyzeHeader(fp_in, &header);

    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);

    WAV_ReadBuffer(fp_in, read_data, CHANNELS * FRAME_SIZE);

    FILE *fp = fopen("1.txt", "w");
    for (int i = 0; i < FRAME_SIZE; i++)
    {
        fprintf(fp, "%5d,%5d\n", write_data[i], read_data[i]);
    }

    fclose(fp_in);
    fclose(fp);
    free(read_data);
    free(write_data);

    printf("process done\n");
    return 0;
}
#endif

#if TEST_PCM24

int main(int argc, char **argv)
{
    char *file_in = NULL;
    char *file_out = NULL;
    FILE *fp_in = NULL;
    FILE *fp_out = NULL;

    /* only read 16bit integer */
    int *read_data = NULL;
    int *write_data = NULL;
    WAV_HEADER header = {0};
    int frame_cnt = 0;
    int n = 0;

    /*give the fixed parameter*/
    file_in = "TT.wav";
    file_out = "TT.wav";

    /*open the wav file*/
    fp_out = fopen(file_out, "wb");
    if (!fp_out)
    {
        printf("Error: Could not open file %s \n", file_out);
        return -1;
    }

    read_data = malloc(CHANNELS * FRAME_SIZE * sizeof(int));
    write_data = malloc(CHANNELS * FRAME_SIZE * sizeof(int));
    pcm24_data_init(write_data, CHANNELS * FRAME_SIZE);

    fseek(fp_out, WAVE_HEADER_SIZE, SEEK_SET);
    frame_cnt = 0;
    for (int i = 0; i < 10; i++)
    {
        WAV_WriteBuffer(fp_out, write_data, CHANNELS * FRAME_SIZE);
        frame_cnt++;
    }
    WAV_FillHeader(&header, FS, CHANNELS, frame_cnt * CHANNELS * FRAME_SIZE);
    WAV_WriteHeader(fp_out, &header);
    fclose(fp_out);

    fp_in = fopen(file_in, "rb");
    if (!fp_in)
    {
        printf("Error: Could not open file %s \n", file_in);
        return -1;
    }

    /*analyze the input wav head*/
    WAV_AnalyzeHeader(fp_in, &header);

    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);

    WAV_ReadBuffer(fp_in, read_data, CHANNELS * FRAME_SIZE);

    FILE *fp = fopen("1.txt", "w");
    for (int i = 0; i < FRAME_SIZE; i++)
    {
        fprintf(fp, "%8d,%8d\n", write_data[i], read_data[i]);
    }

    fclose(fp_in);
    fclose(fp);
    free(read_data);
    free(write_data);

    printf("process done\n");
    return 0;
}

#endif

#if TEST_PCM32

int main(int argc, char **argv)
{
    char *file_in = NULL;
    char *file_out = NULL;
    FILE *fp_in = NULL;
    FILE *fp_out = NULL;

    /* only read 16bit integer */
    int *read_data = NULL;
    int *write_data = NULL;
    WAV_HEADER header = {0};
    int frame_cnt = 0;
    int n = 0;

    /*give the fixed parameter*/
    file_in = "TT.wav";
    file_out = "TT.wav";

    /*open the wav file*/
    fp_out = fopen(file_out, "wb");
    if (!fp_out)
    {
        printf("Error: Could not open file %s \n", file_out);
        return -1;
    }

    read_data = malloc(CHANNELS * FRAME_SIZE * sizeof(int));
    write_data = malloc(CHANNELS * FRAME_SIZE * sizeof(int));
    pcm32_data_init(write_data, CHANNELS * FRAME_SIZE);

    fseek(fp_out, WAVE_HEADER_SIZE, SEEK_SET);
    frame_cnt = 0;
    for (int i = 0; i < 10; i++)
    {
        WAV_WriteBuffer(fp_out, write_data, CHANNELS * FRAME_SIZE);
        frame_cnt++;
    }
    WAV_FillHeader(&header, FS, CHANNELS, frame_cnt * CHANNELS * FRAME_SIZE);
    WAV_WriteHeader(fp_out, &header);
    fclose(fp_out);

    fp_in = fopen(file_in, "rb");
    if (!fp_in)
    {
        printf("Error: Could not open file %s \n", file_in);
        return -1;
    }

    /*analyze the input wav head*/
    WAV_AnalyzeHeader(fp_in, &header);

    fseek(fp_in, WAVE_HEADER_SIZE, SEEK_SET);

    WAV_ReadBuffer(fp_in, read_data, CHANNELS * FRAME_SIZE);

    FILE *fp = fopen("1.txt", "w");
    for (int i = 0; i < FRAME_SIZE; i++)
    {
        fprintf(fp, "%12d,%12d\n", write_data[i], read_data[i]);
    }

    fclose(fp_in);
    fclose(fp);
    free(read_data);
    free(write_data);

    printf("process done\n");
    return 0;
}
#endif
```

