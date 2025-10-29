# 对KMP算法的理解

> 此处对KMP算法的理解为本人根据王道书、AI给出的解释综合得到，可能存在错误

## 1. 核心：PM（部分匹配表）

- 部分匹配表代表当前子串的最长公共前后缀的长度，例如`ababaaababaa`,其部分匹配表为`001231123456`，可用表格理解如下：

  | 当前子串     | 最长公共前后缀 | 最长公共前后缀长度 |
  | :----------- | :------------- | :----------------: |
  | a            | none           |         0          |
  | ab           | none           |         0          |
  | aba          | a              |         1          |
  | abab         | ab             |         2          |
  | ababa        | aba            |         3          |
  | ababaa       | a              |         1          |
  | ababaaa      | a              |         1          |
  | ababaaab     | ab             |         2          |
  | ababaaaba    | aba            |         3          |
  | ababaaabab   | abab           |         4          |
  | ababaaababa  | ababa          |         5          |
  | ababaaababaa | ababaa         |         6          |

把第三列视作数组即为部分匹配表

## 2. PM与next数组的关系

- 在王道书中，next数组的引出是基于PM表的，因此，我认为**PM表是next数组的前提**，同样以`ababaaababaa`为例，其对应关系如下：

  | **PM表**                          | 0 0 1 2 3 1 1 2 3 4 5 6      |
  | --------------------------------- | ---------------------------- |
  | **0-based next数组（0下标开始）** | **-1 0 0 1 2 3 1 1 2 3 4 5** |
  | **1-based next数组（1下标开始）** | **0 1 1 2 3 4 2 2 3 4 5 6**  |

不难发现：

- 将PM表中的数字整体往右移动，把最左边的位补上-1，便得到了王道书中的0-based next数组
- 把0-based next数组的每一个数字加1，便得到了1-based next数组

因此，在实际题目中，**只需要把PM表确定，next数组便随之确定了**

## 3. CPP下的KMP实现

以上对KMP的介绍已经能够让我们正确写出王道书中与KMP有关的题了，但是如何在代码中实现这一模式匹配算法呢？究竟是直接用PM表，还是用0-based next数组或1-based next数组来充当next数组呢？

答案是：**都可以**

但是，鉴于本人能力有限，个人认为使用PM表即可充当KMP中next数组的职责，且代码相对易读，无需对next数组进行其他变换

其核心代码如下：

```c++
//此函数用于计算模式串的部分匹配表
//即对于ababaaababaa,其输出应为0 0 1 2 3 1 1 2 3 4 5 6
vector<int>calcNext(string t)
{
    vector<int>nxt(t.size(),0);
    int j=0;
    nxt[0]=0;

    for(int i=1;i<t.size();i++)//i始终递增，不回头，用for循环
    {
        //当失配时，j回退到这之前已确定的最长公共前缀的前缀的下一个字符下标处
        //注意，j既表示当前最长的公共前后缀长度，也可以视作是最长公共前后缀中前缀的下一个字符的下标
        //比如aba，当前j=1，1表示最长公共前后缀长度为1（即a），1也表示a的后一个字符b的下标
        while(j!=0&&t[j]!=t[i])
        {
            j=nxt[j-1];        
        }
        
        //当相等时，说明最长前后缀可以增加了，加1后，将其设为当前的nxt数组的值
        //比如aba -> abab, j=1 -> j=2,nxt[3]-2
        if(t[j]==t[i])
        {
            j++;
            nxt[i]=j;
        }else{
            //当不相等时，上述while循环已经将j回退至对应的nxt值，
            //比如abababa -> abababac , j=3 -> 经过while循环 -> j=0 , 此时t[j]!=t[i]，（a!=c）因此nxt[7]=0
            nxt[i]=j;
        }
        
    }
    return nxt;
}
```

以及

```c++
bool kmp(string s,string t)
{
    vector<int>nxt=calcNext(t);//调用上述函数算出pm表（可理解为next数组）
    int i=0;
    int j=0;
    for(int i=0;i<s.size();i++)//同样i永不回退，用for循环
    {
        while(j!=0&&t[j]!=s[i])//回退逻辑，与calcNext的判断语句很像，只是把t[i]换成s[i]而已
        {
            j=nxt[j-1];
        }

        if(t[j]==s[i])//相等时递增模式串的指针j
        {
            j++;
        }

        //其实上述代码就是kmp的核心实现，即利用next数组存储的信息避免回退过多，下面就是单纯判断是否匹配成功
        if(j==t.size())
        {
            return 1;
        }
    }
    return 0;
}
```

再随便写个入口

```c++
int main()
{
    string s;string t;
    cin>>s;
    cin>>t;

    vector<int>ans=calcNext(t);

    for(int i=0;i<t.size();i++)
    {
        cout<<ans[i]<<" ";
    }
    cout<<endl;

    cout<<kmp(s,t);

    return 0;
}
```

```shell
//输入aabaabaccabacab
//   acab

//输出
0 0 1 0    //acab的部分匹配表
1          //是否匹配

```

可以看到，只需要把PM表求出来，便可以写出KMP的代码了

至于0-based、1-based的那些next数组，我感觉只对做题有用

如果要把代码改为基于0-based、1-based的next数组实现也可以，比如把j=nxt[j-1]改为j=nxt[j]这样，但相比此解法或许较为繁琐（或许也不会，因为我只会这种）

