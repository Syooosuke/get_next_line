*This project has been created as part of the 42 curriculum by syokota.*

## Description
ファイルディスクリプタが指すテキストファイルを、read関数で読み取り、静的変数(static)に代入しながら、テキストファイルの中身を読み取っていくことで、終端に達するまで、`'\n'`がテキスト内に現れるたびに、テキスト中の1行を呼び出す、get_next_line関数を作成するプロジェクトです。

## Instruction
すべての関数ファイル (*.c) のオブジェクトファイル (*.o) を作成する場合は
```c:
cc -Wall -Wextra -Werror -c *.c
``` 
によりコンパイル可能です。  
また、read関数で読み込むバイト数を指定する際は、
```c:
cc -Wall -Wextra -Werror -D BUFFER_SIZE=n -c *.c
``` 
(nは任意の整数)で、コンパイルすることで、読み込むバイト数を指定できます。

オブジェクトファイル(*.o)を消去したい場合は、以下のコマンドを実行して下さい。
```rm.c:
rm *.o
```

実行ファイル(a.out)を作成したい場合は、以下の手順に従うことで、コンパイル可能です。  
共通のディレクトリ内に、以下の main.c と xxx.txt(xxxは任意) を作成します。  
それぞれのファイル例は、以下に記述しています。  
以下に記載の、main.cを作成した場合、コマンドライン引数を持たせなかった場合は、標準入力モードが起動し、コマンドライン引数にテキストファイル名(xxx.txt)を持たせることで、テキストファイル内のすべての文章を、get_next_line関数を使いながら、出力することが可能です。

ターミナルコマンドは以下のとおりです。
```c:
cc -Wall -Wextra -Werror *.c
``` 
また、read関数で読み込むバイト数を指定する際は、
```c:
cc -Wall -Wextra -Werror -D BUFFER_SIZE=n *.c
``` 
(nは任意の整数)で、コンパイルすることで、読み込むバイト数を指定できます。

### main.c
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

### example.txt
```xxx.txt:
Hello, world!
This is a get_next_line test.
End of file test.
```
## アルゴリズム設計
プロトタイプは以下のとおりです。
```c:
char *get_next_line(int fd);
```
本プロジェクトでは、静的変数（Static Variable）を用いた文字列の持ち越しを行っています。処理は、以下の3つのステップで構成されています。 
1. 読み込みと蓄積（read_to_stash）
read関数を用いて、ファイルから `BUFFER_SIZE`バイトずつ文字列を読み込みます。
読み込んだ文字列（bufferに格納）を、前回からの持ち越し文字列（stash）の末尾に `ft_strjoin`を用いて結合していきます。
新しく読み込んだ buffer の中に改行文字`\n`が見つかるか、ファイルの終端（EOF）に達するまでこのループを繰り返します。

2. 1行の抽出（extract_line）
完成した`stash`の先頭から文字を調べ、最初の`\n`まで（あるいは`\0`まで）の長さを計算します。
計算した長さ分のメモリを確保し、`stash`から1行分だけを新しい文字列（line）としてコピーし、最後に`\0`を付与して返します。

3. 余りの保存（clean_stash）
抽出した`\n`より後ろに格納されている「次回の行の一部になる文字」の長さを計算します。
その長さ分のメモリを新しく確保して文字列を移し替え、古い`stash`のメモリを`free`で解放します。
この新しい文字列を静的変数`stash`に代入し直すことで、次回の`get_next_line`呼び出しに状態を引き継ぎます。

読み取るデータがなくなったり、エラーが発生した場合は、`NULL`を返します。  
ファイルの末尾に達し、かつファイルが`\n`で終わらない場合を除き、返される行には`\0`が含まれています。  
また、必要なヘルパー関数はすべて、`get_next_line_utils.c`ファイルに含まれています。
ヘルパー関数には、文字列の長さを測る`ft_strlen`、文字列中から特定の文字を見つける`ft_strchr`、 文字列同士を結合する`ft_strjoin`を採用しています。

ファイル記述子に関連付けられたファイルが、前回の呼び出し後に変更され、かつ`read`関数がまだファイルの末尾に到達していない場合、`get_next_line()`は未定義の挙動を示します。

## 設計方針

 メモリリークを防ぐため、文字列の結合、切り詰めの際は、必ずその関数内で古いメモリを解放するというルールを徹底しました。
`ft_strjoin`の内部で第1引数（s1）を自動的に`free`する設計にしたことで、`read`ループが何万回回っても一時的な文字列がメモリ上に放置されることがなく、堅牢性を担保しています。
 ループの継続条件において、毎回巨大な`stash`全体から`\n`を探すのではなく、読み込んだ`buffer`の中に`\n`があるかを調べるように変更し、無駄な計算時間を大幅に削ぎ落としました。
また、関数の処理を、読見出し、切り出し、移し替えの役割ごとに、独立した関数に分割することで、コードの可読性と保守性を高めています。

## Resources

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
* main関数を作らず（実行ファイルを作らず）に、コンパイルする方法について