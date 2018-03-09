本篇文章是对区块链开发中的Go语言中常用的io操作的库做一个梳理

## io，最基本的io
#### Reader
```
type Reader interface {
    Read(p []byte) (n int, err error)
}
```
实现了Reader接口的都可以用read方法，将数据读入到p字节数组，n表示读取了几个字节，err返回错误。
如果读到了文件尾EOF，则err返回EOF。  
注意，当文件最后一小段已经无法填满p这个字节数组时，不会产生EOF的错误，只会在下一次读取时产生n=0，err=io.EOF的错误

举例
```
func main() {
    file, _ := os.Open("main.go")
    var a [128]byte

    count:=0
    for {
        n, err := file.Read(a[:])
        count+=1
        if err != nil {
            if err == io.EOF {
                break
            } else {
                os.Exit(1)
            }
        }
        fmt.Printf("%s\n", a[:n])
    }
    fmt.Printf("%d\n", count)

}
```

#### Writer
```
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
Write 将 len(p) 个字节从 p 中写入到基本数据流中。它返回从 p 中被写入的字节数 n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。若 Write 返回的 n < len(p)，它就必须返回一个 非nil 的错误。  
常见错误原因有磁盘满了

#### ReaderAt 和 WriterAt 接口
和Reader，Writer类似，但是需要自己调控偏移量。  
注意：接近文件尾巴时，当n小于数组大小时也触发了err.EOF，需要自行把最后n小于数组大小的这点数据处理一下。

举例：
```
func main() {
    file, _ := os.Open("main.go")
    var a [128]byte

    count := 0
    var pos int64 = 0
    for {
        n, err := file.ReadAt(a[:], pos)
        count += 1
        pos += int64(n)
        if err != nil {
            if err == io.EOF {
                fmt.Printf("%s", a[:n]) //区别在这里
                break
            } else {
                os.Exit(1)
            }
        }
        fmt.Printf("%s", a[:n])
    }
    fmt.Println()
    fmt.Printf("%d", count)

}
```

#### ReaderFrom 和 WriterTo 接口
一次性读完直到EOF，或者写入全部数据

#### Seeker 接口
```
type Seeker interface {
    Seek(offset int64, whence int) (ret int64, err error)
}
```
用来设置偏移量，也就是从哪开始读，offset由whence解释。
* 0 表示相对于文件的起始处
* 1 表示相对于当前的偏移，
* 2 表示相对于其结尾处。 

#### ByteReader 和 ByteWriter
读或写一个字节

## ioutil — 方便的IO操作函数集
#### ReadAll 
一次性读取数据
#### ReadDir 
读取目录并返回排好序的文件和子目录名
#### ReadFile 和 WriteFile
```
func WriteFile(filename string, data []byte, perm os.FileMode) error
```
这里特别注意的是写文件的权限问题，perm的数值，和linux规则一致
 四位（777）：  
模式| 数字 
----|----
 rwx| 7 
 rw-| 6 
 r-x| 5 
 r--| 4 
 -wx| 3 
 -w-| 2 
 --x| 1 
 ---| 0
组合如0666，表示rw-rw-rw-

## bufio，带缓存的io
是io库的包装，提供带缓存的方法

#### ReadSlice、ReadBytes、ReadString 和 ReadLine 方法
后三个方法最终都是调用ReadSlice来实现的
##### ReadSlice
```
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```
示例：
```
reader := bufio.NewReader(strings.NewReader("http://studygolang.com. \nIt is the home of gophers"))
line, _ := reader.ReadSlice('\n')
fmt.Printf("the line:%s\n", line)
// 这里可以换上任意的 bufio 的 Read/Write 操作
n, _ := reader.ReadSlice('\n')
fmt.Printf("the line:%s\n", line)
fmt.Println(string(n))
```
输出：
```
the line:http://studygolang.com. 

the line:It is the home of gophers
It is the home of gophers
```
注意ReadSlice每次返回的line是指向同一个缓存数组，因此ReadSlice的实现是反复覆盖重写缓存数组。

如果ReadSlice在找到分界符前
1. 缓存数组就满了，则返回bufio.ErrBufferFull
2. 遇到EOF了，则返回ErrEOF

##### ReadBytes
```
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
```
返回的byte是copy的一份数组

从以下实验可看出来
```
reader := bufio.NewReader(strings.NewReader("http://studygolang.com. \nIt is the home of gophers"))
line, _ := reader.ReadBytes('\n')
fmt.Printf("the line:%s\n", line)
// 这里可以换上任意的 bufio 的 Read/Write 操作
n, _ := reader.ReadBytes('\n')
fmt.Printf("the line:%s\n", line)
fmt.Println(string(n))
```
输出
```
the line:http://studygolang.com. 

the line:http://studygolang.com. 

It is the home of gophers
```

##### ReadString
是对ReadBytes的封装，将返回的line转换成string

##### ReadLine
```
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```
这里要说的是isPrefix，用于读取的一行超过了缓存大小，则isPrefix为true，下次还读这行余下的部分，直到读完这行才isPrefix返回false

ReadLine返回的文本不会包含行结尾（"\r\n"或者"\n"）


##### Peek
该方法只是“窥探”一下Reader中没有读取的n个字节。好比栈数据结构中的取栈顶元素，但不出栈。
```
func (b *Reader) Peek(n int) ([]byte, error)
```
同上面介绍的ReadSlice一样，返回的[]byte只是buffer中的引用。所以在并发的时候有可能就被别人给改了

#### Scanner 类型和方法
用于方便的按token读取数据，token的分词规则用SplitFunc定义。默认按行分词，会去掉末尾换行符。
了解Scanner前要先了解SplitFunc
##### SplitFunc
```
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```
SplitFunc 定义了 用于对输入进行分词的 split 函数的签名。

参数 
1. data 是还未处理的数据，
2. atEOF 标识 Reader是否还有更多数据（是否到了EOF）。
 
返回值 
1. advance data里下一个token开始位置
2. token 表示当前token的结果数据
3. err 则代表可能的错误。

举例
```
func main() {
    // Comma-separated list; last entry is empty.
    const input = "1,2,3,4,"
    scanner := bufio.NewScanner(strings.NewReader(input))
    // Define a split function that separates on commas.
    onComma := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
        for i := 0; i < len(data); i++ {
            if data[i] == ',' {
                return i + 1, data[:i], nil
            }
        }
        // There is one final token to be delivered, which may be the empty string.
        // Returning bufio.ErrFinalToken here tells Scan there are no more tokens after this
        // but does not trigger an error to be returned from Scan itself.
        return 0, data, bufio.ErrFinalToken
    }
    scanner.Split(onComma)
    // Scan.
    for scanner.Scan() {
        fmt.Printf("%q ", scanner.Text())
    }
    if err := scanner.Err(); err != nil {
        fmt.Fprintln(os.Stderr, "reading input:", err)
    }
}
```
输出
```
"1" "2" "3" "4" "5" 
```

你也可以用系统定义好的几个分割token的方法。
1. ScanBytes 返回单个字节作为一个 token。
2. ScanRunes 返回单个 UTF-8 编码的 rune 作为一个 token。返回的 rune 序列（token）和 range string类型 返回的序列是等价的，也就是说，对于无效的 UTF-8 编码会解释为 U+FFFD = "\xef\xbf\xbd"。

3. ScanWords 返回通过“空格”分词的单词。如：study golang，调用会返回study。注意，这里的“空格”是 unicode.IsSpace()，即包括：'\t', '\n', '\v', '\f', '\r', ' ', U+0085 (NEL), U+00A0 (NBSP)。

4. ScanLines 返回一行文本，不包括行尾的换行符。这里的换行包括了Windows下的"\r\n"和Unix下的"\n"。

#####  Scanner 的使用方法
1. NewScanner
2. Split设置分割token的方法
3. 循环scanner.Scan()
4. 在循环里用scanner.Text()取token
示例
```
const input = "This is The Golang Standard Library.\nWelcome you!"
scanner := bufio.NewScanner(strings.NewReader(input))
scanner.Split(bufio.ScanWords)
count := 0
for scanner.Scan() {
    count++
}
if err := scanner.Err(); err != nil {
    fmt.Fprintln(os.Stderr, "reading input:", err)
}
fmt.Println(count)
```

#### Writer
带缓存的writer，记得在最终的写入操作执行完后flush一下，确保全部缓存都真正写入。

### 参考  
1.[《Go语言标准库》The Golang Standard Library by Example](
https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter01/01.0.html)


关于我：

linxinzhe，全栈工程师，目前供职于某500强通信企业，擅长移动APP，服务器开发。人工智能，区块链爱好者。目前ALL in 区块链。

GitHub:https://github.com/linxinzhe

欢迎留言讨论，也欢迎关注我，收获更多区块链开发相关的知识。   

![公众号.jpg](http://upload-images.jianshu.io/upload_images/810998-7bc0510cf2a159cd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


