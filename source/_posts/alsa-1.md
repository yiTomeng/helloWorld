title: alsaの勉強（1）ーー OSS
date: 2015-02-20 09:59:19
tags: linux
---
最近linux上の音声再生方法を集中しているけど、未だに全部わかっているわけではないけど、とりあえずメモしておく。
最初の時、linux上の音声再生方法を調べたところ、OSSというものがある。OSSは「Open Sound System」の略称である。
この方法はlinuxのシステムコールを呼び出す必要だ。その他、デバイス名も必要になる。OSSに関するデバイス名は/dev/dspとか、
/dev/mixerなどたくさんある。他のファイル（/dev/sndや/dev/audioなど）も音声と関わっている。一番重要なのは/dev/dsp
と/dev/mixerだと思う。まずこの2つデバイスを簡単に説明しましょう。

/dev/dspは放送・録音用のデバイスである。
OSSを使うと、システムコールopen()、close()、ioctl()、read()、write()が必要だ。
普通にはopen()→ioctl()→close()のように音声ファイルを再生する。open（）とclose()はファイル操作と同じくように使ったら特に問題ないだが、音声放送に対して一番重要なのはデバイスの設定だと思う。
どういうふうに設定すればいいのか？何かを設定必要なのか？

まずは、音声ファイルを放送するとき一番重要なものを書いときましょう。
    
    ①チャンネル：音声ファイルの特性の一つであるチャンネルというものがある。今の時点で、たくさんモードがある。monoとstereoはよくみえるが、surround 5.1やsurround7.1もよく耳に入る。monoは1チャンネルしかない。stereoは2つチャンネルがあって、左右2つのスピーカーで音声を再生する方式のことである。ステレオフォニック(stereo)再生はモノフォニック(mono)再生に比較して、音像定位や音場感が加わり、再生音の臨場感が増す効果がある。
  
    ②サンプリング周波数：サンプリング周波数（サンプリングしゅうはすう）は、音声等のアナログ波形を、デジタルデータにするために必要な処理である標本化（サンプリング）で、単位時間あたりに標本を採る頻度。ふつう使われる単位はHz。
    
    ③フォーマット：音データをコンピュータシステム上で格納する際のコンテナフォーマットである。たくさんの分類方法があり、一つは圧縮するかしないかを基準として、分けている。今回使っているのはMicrosoftのwavフォーマットである。このフォーマットは圧縮フォーマットと非圧縮フォーマット両方あるということだ。非圧縮フォーマットの方はPCMというものだ。
今頭に入っているのは以上の三つしかない。（今後思いついたら、追加する）

以上の説明の通り、OSSを使うと、以上の三つを設定しなければならない。（OSSの使い方は簡単だから、直ちに例を挙げる）
###ⅰ.sound_lib.c:

```C
/******************************************************************************
* SYSTEM       :   音声放送LIB
* PROGRAM      :   放送を行うライブラリ
* MODULE       :   sound_lib.c  
* REMARKS      :
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.2.2   晏       初期作成
* [V01.00.01]  2015.2.6   晏        
*****************************************************************************/

#include <fcntl.h> 
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include <linux/soundcard.h>
#include "sound_lib.h"

//#define HEADER_BUFSIZE		36											//ヘッダファイルのバッファサイズ
#define BUFSIZE				64000											//ファイル読み込む用バッファサイズ
#define ERR_MSG_SIZE		64
//#define FILENAME_SIZE		32												//ファイル名バッファサイズ
#define DSP_DEVICE			"/dev/dsp"										//音声再生用デバイス

//WAVファイルのヘッダ
typedef struct
{
char	chunk_id[4];													//識別子"RIFF"
int		chunk_size;														//ファイルのサイズ(識別子"RIFF"と自分を除く)
char 	format[4];														//フォーマット部("WAVE")
char 	sub_chunk_id[4];												//識別子"fmt"
int  	sub_chunk_size;													//この後のファイルのサイズ
short 	audio_format;													//音声のフォーマット(1:PCM 　1以外は他の圧縮ファイル)
short 	num_channels;													//チャンネル
int 	sample_rate;													//サンプリング周波数
int 	byte_rate;														//1秒に使用するバイト数
short 	block_align;													//ブロックのアラインメント(今使っていない)
short 	bits_per_sample;												//量子化のビット数
}wave_file_header_info;

//WAVファイルのインフォーメーション
typedef struct
{
unsigned int play_start_pos;											//データ開始位置
unsigned int play_pos;													//停止場所(保留　今使っていない)
FILE 	*fp;															//音声ファイル識別子
char 	play_filename[FILENAME_SIZE];									//音声ファイル名
int 	stop;															//中止状態(保留　今使っていない)
int 	refresh_times;													//再生回数
wave_file_header_info wave_header;										//WAVファイルヘッダ
int 	data_size;	 													//データ部のサイズ
}play_wave_info;


static const char err_msg[][ERR_MSG_SIZE] = {
[ERR_ZERO] 					= "エラー無し。",
[ERR_FILE_OPEN] 			= "ファイルopen失敗しました。",
[ERR_FILE_READ] 			= "ファイルの読み込みは失敗しました。",
[ERR_FILE_WRITE] 			= "",
[ERR_DEVICE_OPEN] 			= "デバイスopen失敗しました。",
[ERR_DEVICE_READ] 			= "",
[ERR_DEVICE_WRITE] 			= "DSPデバイスに書き込むことが失敗しました。",
[ERR_DEVICE_SET] 			= "DSPデバイス設定失敗しました。",
[ERR_DEVICE_DESCRIPTOR]		= "ファイル識別子異常",
[ERR_BROADCAST] 			= "放送失敗しました。",
[ERR_SIZE] 					= "ファイルサイズはヘッダサイズより小さい。",
[ERR_NOT_RIFF] 				= "Specified file is not RIFF file.\n",
[ERR_NOT_WAVE] 				= "Specified file is not WAVE file.\n",
[ERR_NOT_FIND_FMT_CHUNK] 	= "Failed to find fmt chunk.\n",
[ERR_NOT_FIND_DATA_CHUNK] 	= "Failed to find data chunk.\n",
[ERR_NOT_FIND_DATA] 		= "Failed to find offset of PCM data.\n",
[ERR_NOT_PCM] 				= "Specified file's sound data is not PCM format.\n",
[ERR_BITS_PER_SAMPLE] 		= "Specified file's sound data is %d bits, not 8 nor 16 bits.\n",
[ERR_SET_FMT] 				= "ioctl( SOUND_PCM_SETFMT )",
[ERR_WRITE_RATE] 			= "ioctl( SOUND_PCM_WRITE_RATE )",
[ERR_WRITE_CHANNEL] 		= "ioctl( SOUND_PCM_WRITE_CHANNELS )"
};


static void wave_init(const char *filename, play_wave_info *wave_info, int refresh_count);		//WAVファイルインフォーメーション初期化
static int wave_read_file_header(play_wave_info *wave_info);									//WAVファイルヘッダを読み込む
static int set_dsp(int *dsp, play_wave_info *wave_info);										//DSPデバイスを設定
static int dsp_play(int *dsp, int *stop_cmd, play_wave_info *wave_info);						//放送


/*SOH*************************************************************************
* NAME         : open_voice_device
* FUNCTION     : デバイスを開く
* VERSION      : 01.00.00
* IN/OUT       : (i/o) dsp: 音声放送デバイスの識別子
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.06  晏         
*************************************************************************EOH*/
int open_voice_device(int *dsp)							
{
if ( -1 == ( *dsp = open( DSP_DEVICE, O_WRONLY ) ) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s:%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_OPEN], DSP_DEVICE);
return ERR_DEVICE_OPEN;
}
return ERR_ZERO;
}

/*SOH*************************************************************************
* NAME         : close_voice_device
* FUNCTION     : デバイスをクロス
* VERSION      : 01.00.00
* IN/OUT       : (i/o) dsp: 音声放送デバイスの識別子
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.06  晏         
*************************************************************************EOH*/
int close_voice_device(int *dsp)										
{
if (*dsp <= 0)
{
fprintf(stderr, "[%s:%d:%s()]%s:%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_DESCRIPTOR], DSP_DEVICE);
return ERR_DEVICE_DESCRIPTOR;
}

close(*dsp);

return ERR_ZERO;
}

/*SOH*************************************************************************
* NAME         : start_play_wave
* FUNCTION     : WAVファイル放送インターフェス
* VERSION      : 01.00.00
* IN/OUT       : (i/ ) dsp				: 音声放送デバイスの識別子
*				: (i/ ) filename		: 音声ファイル名
*              : (i/ ) refresh_times 	: 再生回数
*				: (i/ ) play_status		: 再生状態
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
* [V01.00.01]  2015.02.06  晏         
*************************************************************************EOH*/

int start_play_wave(int *dsp, const char *filename, int refresh_times, play_stat *play_status)
{

//	int dsp = 0;						//DSPデバイスの識別子

play_wave_info wave_info;			//音声ファイルインフォーメーション

wave_init(filename, &wave_info, refresh_times);					//音声ファイルインフォーメーションを初期化する


//音声ファイルを開く
if( (wave_info.fp = fopen(filename, "r")) == NULL )
{
fprintf(stderr, "[%s:%d:%s()]%s:%s\n", __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_FILE_OPEN], filename);
return (int)ERR_FILE_OPEN;
}

//音声ファイルのヘッダを読み込む、チェック処理を行う
if ( ERR_ZERO != wave_read_file_header(&wave_info) )
{
fclose(wave_info.fp);
return -1;
}
#if 0		//もう一つ関数に移動
//DSPデバイスを開く
if ( -1 == ( dsp = open( DSP_DEVICE, O_WRONLY ) ) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s:%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_OPEN], DSP_DEVICE);
fclose(wave_info.fp);
return ERR_DEVICE_OPEN;
}
#endif	
//音声ファイルヘッダデータにより、DSPデバイスを設定する
if ( ERR_ZERO != set_dsp(dsp, &wave_info) )
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_SET]);
//		close(dsp);
fclose(wave_info.fp);
return ERR_DEVICE_SET;
}

//放送を行う
if ( ERR_ZERO != dsp_play(dsp, &play_status->stop_cmd, &wave_info) )
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_BROADCAST]);
//		close(dsp);
fclose(wave_info.fp);
return ERR_BROADCAST;
}

//	close(dsp);
fclose(wave_info.fp);

return ERR_ZERO;
}

/*SOH*************************************************************************
* NAME         : stop_play_wave
* FUNCTION     : WAVファイル放送ストップインターフェス
* VERSION      : 01.00.00
* IN/OUT       : 
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.06  晏         
*************************************************************************EOH*/
void stop_play_wave(play_stat *play_status)
{
if (IS_RUNNING == play_status->status)
{
play_status->stop_cmd = STOP_PLAY;					//放送をストップフラグを設定する
}
else
{
fprintf(stderr, "[%s:%d:%s()]音声を再生していません。",  __FILE__, __LINE__, __FUNCTION__);
}
}														


/*SOH*************************************************************************
* NAME         : wave_init
* FUNCTION     : 音声ファイルインフォーメーションを初期化する
* VERSION      : 01.00.00
* IN/OUT       : (i/ ) filename: ファイル名
*              : (i/o) wave_info : 音声ファイルインフォーメーション
: (i/ )refresh_count:再生回数
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
*************************************************************************EOH*/

static void wave_init(const char *filename, play_wave_info *wave_info, int refresh_count)
{
wave_info->play_start_pos = 0;
wave_info->play_pos = 0;
wave_info->stop = 0;
wave_info->data_size = 0;
wave_info->refresh_times = refresh_count;
strncpy(wave_info->play_filename, filename, FILENAME_SIZE);
memset(&wave_info->wave_header, 0x00, sizeof(wave_file_header_info));
}


/*SOH*************************************************************************
* NAME         : wave_read_file_header
* FUNCTION     : 音声ファイルヘッダデータを読み込んで、チェックします
* VERSION      : 01.00.00
* IN/OUT       : (i/o) wave_info: 音声ファイルインフォーメーション　　
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
*************************************************************************EOH*/

static int wave_read_file_header(play_wave_info *wave_info)
{
int len = 0;							//臨時変数　サイズ
char buf[BUFSIZE];						//臨時バッファ

memset(buf, 0x00, BUFSIZE);				//臨時バッファを初期化する

//ファイルヘッダをバッファに読み込む
//fread( buf, 1, HEADER_BUFSIZE, wave_info->fp );

//バッファからファイルヘッダを構造体にコピーする
//memcpy(&wave_info->wave_header, buf, HEADER_BUFSIZE);

//ファイルヘッダをバッファに読み込む
len = fread( &wave_info->wave_header, 1, sizeof(wave_file_header_info), wave_info->fp );

if (len < sizeof(wave_file_header_info))
{
if (feof(wave_info->fp))
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_SIZE]);
return ERR_SIZE;
}
else
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_FILE_READ]);
return ERR_FILE_READ;
}
}
/*チェック処理を行う*/
//"RIFF"識別子をチェックする
if ( strncmp( wave_info->wave_header.chunk_id, "RIFF", 4 ) != 0 ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_RIFF] );
return ERR_NOT_RIFF;
}

//フォーマット"WAVE"をチェックする
if ( strncmp( wave_info->wave_header.format, "WAVE", 4 ) != 0 ) {
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_WAVE] );
return ERR_NOT_WAVE;
}

//"fmt"識別子をチェックする
if ( strncmp(wave_info->wave_header.sub_chunk_id, "fmt ", 4) != 0)
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_FIND_FMT_CHUNK] );
return ERR_NOT_FIND_FMT_CHUNK;
}

//無限ループでデータ部を探す	
while ( 1 ) {
len = fread( buf, 8, 1, wave_info->fp );								//識別子とサイズを読み込む
if (len < 1)
{
if (feof(wave_info->fp))
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_SIZE]);
return ERR_SIZE;
}
else
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_FILE_READ]);
return ERR_FILE_READ;
}
}

len = *( int* )( &buf[4] );										//サイズをlenに保存する

//"data"識別子をチェックする、"data"でなければ、lenバイト内容を飛ばして、次のエリアをチェックする
if ( strncmp( buf, "data", 4 ) != 0 ) 
{
//位置lenバイト後を設定し、"data"を探し続く
if ( fseek( wave_info->fp, len, SEEK_CUR ) == -1 ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_FIND_DATA_CHUNK] );
return ERR_NOT_FIND_DATA_CHUNK;
}
}
else 
{
break;
}
}

//データ部サイズを設定する
wave_info->data_size = len;

//データの開始位置を保存しながら、チェックします
if ( ( wave_info->play_start_pos = ftell( wave_info->fp ) ) == -1 ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_FIND_DATA] );
return ERR_NOT_FIND_DATA;
}

return ERR_ZERO;
}


/*SOH*************************************************************************
* NAME         : set_dsp　　
* FUNCTION     : DSPデバイスを設定する
* VERSION      : 01.00.00
* IN/OUT       : (i/ ) dsp: DSPデバイスの識別子
*              : (i/ ) wave_info : 音声ファイルインフォーメーション
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
*************************************************************************EOH*/

static int set_dsp(int *dsp, play_wave_info *wave_info)
{
int format = 0;					//フォーマットの初期化
int backup_fmt = 0;				//フォーマットのバックアップ
int rate = 0;					//サンプリング周波数
int backup_rate = 0;			//サンプリング周波数のバックアップ

//メディアフォーマットは"1"でなければ、PCMフォーマットではないと判断します。(この状態は別の圧縮ファイルになってしまう)
if ( 1 != wave_info->wave_header.audio_format)
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_NOT_PCM] );
//fclose( wave_info->fp );
return ERR_NOT_PCM;
}

//量子化ビット数をチェックする
//8bitの場合：フォーマットに"AFMT_U8"を設定する　　
if ( wave_info->wave_header.bits_per_sample == 8 ) 
{
format = AFMT_U8;
backup_fmt = format;
}

//16bitの場合：フォーマットに"AFMT_S16_LE"を設定する
else if ( wave_info->wave_header.bits_per_sample == 16 ) 
{
format = AFMT_S16_LE;
backup_fmt = format;
}

//その以外の場合、エラーとします
else 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_BITS_PER_SAMPLE], wave_info->wave_header.bits_per_sample );
//fclose( wave_info->fp );
return ERR_BITS_PER_SAMPLE;
}

int channel = ( int )wave_info->wave_header.num_channels;					//チャンネルを設定する

rate = wave_info->wave_header.sample_rate;
backup_rate = rate;

//デバイスのフォーマットを設定する
if ( (ioctl( *dsp, SNDCTL_DSP_SETFMT, &format ) == -1) || (backup_fmt != format) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_SET_FMT] );
return ERR_SET_FMT;
}

//デバイスのサンプリングの周波数を設定
if ( (ioctl( *dsp, SOUND_PCM_WRITE_RATE, &wave_info->wave_header.sample_rate ) == -1 ) || (backup_rate != rate) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_WRITE_RATE] );
return ERR_WRITE_RATE;
}

//デバイスのチャンネルを設定する
if ( ioctl( *dsp, SOUND_PCM_WRITE_CHANNELS, &channel ) == -1 ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_WRITE_CHANNEL] );
return ERR_WRITE_CHANNEL;
}

return ERR_ZERO ;
}



/*SOH*************************************************************************
* NAME         : dsp_play　　
* FUNCTION     : 音声を放送する
* VERSION      : 01.00.00
* IN/OUT       : (i/ ) dsp			: DSPデバイスの識別子
*				: (i/ ) stop_cmd	: 再生を中止する識別子
*              : (i/ ) wave_info 	: 音声ファイルインフォーメーション
* RETURN       : 0:OK >0:NG　
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
* [V01.00.01]  2015.02.06  晏         
*************************************************************************EOH*/

static int dsp_play(int *dsp, int *stop_cmd, play_wave_info *wave_info)
{
int len;								//臨時変数サイズ
char buf[BUFSIZE];						//臨時バッファ(ファイル読み込むとデバイスに書き込む用)
int rebroadcast_cnt = 0;				//再生回数
int ret = ERR_ZERO;						//リータン値

printf( "Now playing specified wave file %s ...\n", wave_info->play_filename );
fflush( stdout );

//データの開始場所に設定する
fseek( wave_info->fp, wave_info->play_start_pos, SEEK_SET );

//ループで再生する
while ( 1 ) {

if ( STOP_PLAY == *stop_cmd )
{
fprintf(stderr, "[%s:%d:%s()]中止指示があるため、中止する。",  __FILE__, __LINE__, __FUNCTION__);
break;
}

len = fread( buf, 1, BUFSIZE, wave_info->fp );

//ファイルを再生完了場合や読み込み失敗の場合
if ( len < BUFSIZE ) 
{
//ファイルを再生完了のチェック
if ( feof( wave_info->fp ) ) 
{
//書込み失敗
if ( -1 == write( *dsp, buf, len ) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_WRITE] );
ret = ERR_DEVICE_WRITE;
}
//正常の場合、再生回数により、終了するかもう一度再生するかを決める
else 
{
if ( (++rebroadcast_cnt) < wave_info->refresh_times)
{
fseek( wave_info->fp, wave_info->play_start_pos, SEEK_SET );
continue;
}
}
}
else 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_BROADCAST] );
ret = ERR_BROADCAST;
}

break;
}

if ( -1 == write( *dsp, buf, len ) ) 
{
fprintf(stderr, "[%s:%d:%s()]%s。",  __FILE__, __LINE__, __FUNCTION__, err_msg[ERR_DEVICE_WRITE] );
ret = ERR_DEVICE_WRITE;
break;
}

}


return ret;
}

```

###ⅱ.sound_lib.h:

```C
#ifndef SOUND_LIB
#define SOUND_LIB

#define DEFAULT_PLAY_TIMES 1														

//#define play_wave_default(filename) play_wave(filename, DEFAULT_PLAY_TIMES)		
#define FILENAME_SIZE		32														
#define STOP_PLAY			1														
#define IS_RUNNING			1														

enum ERROR_CODE
{
ERR_ZERO,
ERR_FILE_OPEN,
ERR_FILE_READ,
ERR_FILE_WRITE,
ERR_DEVICE_OPEN,
ERR_DEVICE_READ,
ERR_DEVICE_WRITE,
ERR_DEVICE_SET,
ERR_DEVICE_DESCRIPTOR,
ERR_BROADCAST,
ERR_SIZE,
ERR_NOT_RIFF,
ERR_NOT_WAVE,
ERR_NOT_FIND_FMT_CHUNK,
ERR_NOT_FIND_DATA_CHUNK,
ERR_NOT_FIND_DATA,
ERR_NOT_PCM,
ERR_BITS_PER_SAMPLE,
ERR_SET_FMT,
ERR_WRITE_RATE,
ERR_WRITE_CHANNEL		
};

typedef struct
{
int stop_cmd;
int status;
}play_stat;

extern int start_play_wave(int *dsp,  
const char *filename, 
int refresh_times,
play_stat *play_status);									
extern void stop_play_wave(play_stat *play_status);									
extern int open_voice_device(int *dsp);												
extern int close_voice_device(int *dsp);											

#endif

```
###ⅲ.sound_test.c

```
/******************************************************************************
* SYSTEM       :   音声放送LIB テスト
* PROGRAM      :   放送を行うライブラリ
* MODULE       :   sound_test.c  ※ｿｰｽﾌｧｲﾙ名
* REMARKS      :
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02   晏       初期作成
* [V01.00.01]  2015.02.06  晏
*****************************************************************************/


/**********INCLUDE AREA***********/
#include <stdio.h>
#include <unistd.h>
#include "sound_lib.h"


/*SOH*************************************************************************
* NAME         : main
* FUNCTION     : 単体テスト
* VERSION      : 01.00.00
* IN/OUT       : (i/ ) argc: パラメータ
*              : (i/ ) argv : パラメータ配列
* RETURN       : 0:OK <0:NG　※　リターン値の説明
* REMARKS      : 備考
* HISTORY      :
* ID-----------DATE--------NAME----------NOTE --------------------------------
* [V01.00.00]  2015.02.02  晏         
* [V01.00.01]  2015.02.06  晏         
*************************************************************************EOH*/

int main(int argc, char *argv[])
{
//再生回数の格納先
int refresh_times = 0;

printf("再生回数を入力してください:\n");

scanf("%d", &refresh_times);

printf("再生回数は%d回\n", refresh_times);
/*
//デフォールト再生回数を使う関数のテスト
if (DEFAULT_PLAY_TIMES == refresh_times)
{
if ( -1 == play_wave_default(argv[1]))
{
return -1;
}
}
//普通の関数のテスト
else
{
if ( -1 == play_wave(argv[1], refresh_times))
{
return -1;
}
}
*/
int dsp;

open_voice_device(&dsp);

if ( ERR_ZERO != play_wave(argv[1], refresh_times))
{
return -1;
}

printf("音声ファイル再生：OVER!\n");

close_voice_device(&dsp);

return 0;
}

```

以上の例は簡単で作ったものなので、よくないかもしれないが、中身をみて、OSSはどういう風に使えるかがわかるはずだ。