*This project has been created as part of the 42 curriculum by syokota.*

# Description
ファイルディスクリプタが指すテキストファイルを、read関数で読み取り、静的変数(static)に代入しながら、テキストファイルの中身を読み取っていくことで、終端に達するまで、`'\n'`がテキスト内に現れるたびに、テキスト中の1行を呼び出す、get_next_line関数を作成するプロジェクトです。

# Instruction
すべての関数ファイル (*.c) は、
```c:
cc -Wall -Wextra -Werror *.c *.h
``` 
によりコンパイル可能です。  
read関数で読み込むバイト数を指定する際は、
```c:
cc -Wall -Wextra -Werror -D BUFFER_SIZE=n *.c *.h
``` 
(nは任意の整数)で、コンパイルすることで、読み込むバイト数を指定できます。


プロトタイプは以下のとおりです。
```c:
char *get_next_line(int fd);
```
本関数は、読み込んだ行を返します。読み取るデータがなくなったり、エラーが発生した場合は、NULLを返します。  
ファイルの末尾に達し、かつファイルが\n文字で終わらない場合を除き、返される行には終端文字\nが含まれています。     
必要なヘルパー関数はすべて、get_next_line_utils.c ファイルに含まれています。  
ファイル記述子に関連付けられたファイルが、前回の呼び出し後に変更され、かつ read() がまだファイルの末尾に到達していない場合、get_next_line() は未定義の挙動を示す。

共通のディレクトリ内に、以下の main.c と xxx.txt(xxxは任意) を作成し、まとめてコンパイルすることで、コマンドライン引数から、テキストファイルを取得し、テキストファイル内のすべての文章を、get_next_line関数を使いながら、出力することが可能です。

```main.c:

#include "get_next_line.h"
#include <fcntl.h>
#include <stdio.h>

int	main(int argc, char **argv)
{
	int		fd;
	char	*line;

	if (argc == 2)
	{
		fd = open(argv[1], O_RDONLY);
		if (fd == -1)
		{
			printf("ファイルのオープンに失敗しました\n");
			return (1);
		}
	}
	else
	{
		fd = 0;
		printf("標準入力モード\n");
	}
	while (1)
	{
		line = get_next_line(fd);
		if (line == NULL)
			break ;
		printf("%s\n", line);
	}
	if (fd != 0)
		close(fd);
	return (0);
}
```

# Resources

* man page(glibc) : 標準関数の仕様や挙動の主要なリファレンスとして使用。  
(https://man7.org/linux/man-pages/man2/read.2.html)  
* 第21章lseek()  
(https://mkguytone.github.io/allocator-navigatable/ch21.html)  
* Qiitaブログ : static変数の性質をかんたんに理解する  
(https://qiita.com/FR1SK_noob/items/a6b83916dbf5b3191619)  
* 【C言語】read関数の使い方、間違っていませんか？  
(https://progzakki.sanachan.com/program-lang/c/how-to-use-read/#google_vignette)  

### AIの仕様について
* static変数の使い方
* open, read, close, lseek 関数の使用方法と指定したメモリ領域の移動方法について
