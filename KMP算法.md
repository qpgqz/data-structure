# 模式匹配的KMP算法（只考选择题，思想最重要）
## 基本背景
### 定义
串(string)是零个或多个字符组成的有限序列。  
串和一般线性表的最大差异在操作上，串一般是以子串为操作对象，而线性表是以一个元素为操作对象。
### 串的基本操作
```c
// 赋值
strAssign(&T,char *)
// 复制
strcppy(&T,S)
// 判空
strEmpty(S)
// 比较
strCompare(S,T)
// 求串长
strLength(S)
// 求子串
subString($sub,S,pos,len)
//串联接
concat(&T,S1,S2)
//定位（模式匹配）
index(S,T)
//清空
clearString(&S)
//销毁
destroyString(&S)
```
### 串的存储
1. 定长存储（在首位存储字符串的长度）
2. 堆分配存储（动态分配内存）
3. 块链存储

## 模式匹配算法
### 经典算法
```c
//采用定长存储
//算法思想：逐个比较，如果遇到不一样的就失败，返回最开始的下一个字符，重新开始比较。
int index(SString S, SString T, int pos){
    //返回子串T在主串S中第pos个字符之后的位置
    // 1 <= pos <= strlength(S)
    int i = pos, j = 1;// i是主串的序号，j是匹配过程的序号
    while(i <= S[0] && j <= T[0]){//退出循环的条件：遍历完还没找到或者找到了。
        if(S[i] == T[j]){
            ++i;
            ++j;
        }
        else{
            i = i - j + 2;
            j = 1;
        }
    }
    if(j > T[0]) return i - T[0];
    else return 0;
}
```
问题：假设T=0001, S=00000000001，这种情况下如果回溯，则前2次比较都没有意义，可以直接跳过。
### KMP算法（关键是next数组）
KMP算法由 D.E.Knuth, V.R.Pratt, J.H.Morris 首先发明。  
算法思想：当比较发现不对的时候，说明前面几个都是T的前几项，如果提前对T进行分析，跳过不需要比较的，确定下一个进行比较的位置，直接和现在的 i 进行比较，就可以避免回溯。  
传统算法可以理解为把所有的部分模式（不一定是部分匹配）都试一遍，而KMP算法可以理解为只考虑部分匹配的尚未比较的部分。因此，如果主串有很多部分与模式串部分匹配的，用KMP算法就可以省去不必要的比较。
#### next数组手算(指导下一个比较的模式)
next数组就是当T[j]比较发现不对，找下一个比较的序号。这等价于找前 j-1 个模式的部分匹配值，所谓部分匹配(partial match, PM)值，就是前缀等于后缀的模式的长度。当我们找到前j-1个模式的最长部分匹配值k，则下一个与主串比较的就是T[k+1]，总之有下面的公式（理论性很强）
$$\text{next}[j]=\text{PM}(j-1)+1$$
如果不用部分匹配的术语，也可以“心里默记有意义的回溯”，分情况得到下面的公式（这种方法最直观，是一个一个地得到next，但不够抽象。）
$$
\text{next}[j]=
\begin{cases}
0,&j=1\\
\text{Max } (k)&\exist 0<k-1<j-1:p_{j-k+1}\cdots p_{j-1}=p_1\cdots p_{k-1}\\
1&\text{j > 1 and there is no partial match }
\end{cases}
$$
#### KMP算法
```c
int index_KMP(SString S, SString T, int pos){
    //KMP算法
    int i = pos, j = 1;
    while(i <= S[0] && j<= T[0]){
        if(S[i] == T[j]|| j == 0){
            i++;
            j++;
        }
        else j = next[j];
    }
    if(j > T[0]) return i - T[0];
    else return 0;
}
```
#### next数组算法（太抽象，不适合手算）
算法思想：
- 如果next[j]=k，且 $p_{j}=p_{k}$，则next[j+1]=k+1
- 如果next[j]=k，但$p_{j}\neq p_{k}$，则考察k'=next[k]，是否有$p_j=p_{k'}$，重复上述步骤。
- 当k‘=0，则令next[j+1]=1
```c
void get_next(SString T, int next[]){
    // 求next数组
    next[1] = 0;
    int i = 1, j = 0;// i是已经求得next的序号, i+1为待求，j=next[i]
    while(i < T[0]){
        if(T[i] == T[j] || j == 0)
            next[++i] = ++j;
        else j = next[j];
    }
}
```
#### nextval数组算法
按照上面算法求得的next数组 k=next[j]说明如果比较到第j个位置发生不匹配，则下一个进行比较的在T的第k个位置，但是T[k]如果与T[j]相同，显然没有必要再比较，因此可以直接跳到第一个使得T[k']不等于T[j]的k'。
```c
void get_nextval(SString T, int nextval[]){
    // 求nextval数组
    nextval[1] = 0;
    // i是已经求得next的序号, i+1为待求，j=next[i]，注意j这里仍然是next数组的j，不是nextval的。
    int i = 1, j = 0;
    while(i < T[0]){
        if(T[i] == T[j] || j == 0){
            ++i;
            ++j;// j仍然是next数组中的next[i]！
            if(T[i] != T[j]) 
                nextval[i] = j;
            else //最关键一步！没有直接求，而是递归
                nextval[i]= nextval[j];
        }
        else j = nextval[j];
    }
}
```