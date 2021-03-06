# 题目描述

请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配

## 解题思路

首先考虑两种特殊情况：

- 匹配串和模式串都成功到达尾部，说明匹配成功，返回true
- 只有模式串到达了尾部，说明匹配失败，返回false
- 模式串未结束，匹配串有可能结束也有可能未结束，还是有可能匹配成功的，因为"*"可以出现0次

接下来开始匹配第一个字符，这里有两种可能匹配成功或者匹配失败，但考虑到pattern下一个字符不为可能是"*"，因此分为两种情况：

- pattern所指示的下一个字符不为"*"

  ​	直接匹配当前字符，如果匹配成功直接返回下一个字符，如果匹配失败，直接返回false

- pattern所指示的下一个字符为"*"

  ​	**需要考虑*代表0个、1个、多个的情况**

  ​	a> 当"*"匹配0个字符时，匹配串不变，模式串向后移两位

   	b>当"*"匹配1个字符时，匹配串后移1个，模式串后移2个

  ​	c>当"*"匹配多个字符时，匹配串后移1个，模式串不变

  

  ## 代码实现

  ~~~
  public class Test19 {
      /**
       * 面试题19
       * 题目：请实现一个函数用来匹配包含‘.’和‘*’的正则表达式。模式中的字符'.'表示任意一个字符，
       * 而‘*’表示它前面的字符可以出现任意次（含0次）。本题中，匹配是指字符串的所有字符匹配整个模式。
       */
      public static boolean match(char[] str, char[] pattern) {
          if (str == null || pattern == null) {
              return false;
          }
          return matchCore(str, 0, pattern, 0);
      }
  
      /**
       * 递归进行字符串和模式的匹配
       *
       * @param str     字符串
       * @param i       字符串的匹配指示位置
       * @param pattern 模式
       * @param p       模式的指示位置
       * @return
       */
      public static boolean matchCore(char[] str, int i, char[] pattern, int p) {
          //str到尾，pattern到尾，匹配成功
          if (i >= str.length && p >= pattern.length){
              return true;
          }
          //pattern到尾，str未到尾，匹配失败
          if (i != str.length && p >= pattern.length){
              return false;
          }
  
          //str到尾，pattern未到尾(不一定匹配失败，因为a*可以匹配0个字符)
          if (i == str.length && p != pattern.length){
              //只有pattern剩下的部分类似a*b*c*的形式，才匹配成功
              if (p+1 < pattern.length && pattern[p+1] == '*'){
                  return matchCore(str,i,pattern,p+2);
              }
  
              return false;
          }
  
          //str未到尾，pattern未到尾
          //pattern的下一个字符是*
          if (p+1 < pattern.length && pattern[p+1] == '*'){
              //此处表示第1个字符匹配，且注意匹配串不能结束，匹配串结束的情况上面已经讨论了
              if (str[i] == pattern[p] || (pattern[p]=='.' && i != str.length) ){
                  return matchCore(str,i,pattern,p+2) //*匹配0个，跳过
                          ||matchCore(str,i+1,pattern,p+2) //*匹配1个，跳过
                          ||matchCore(str,i+1,pattern,p); //*匹配1个，再匹配str中的下一个
              }else {
                  //第1个字符不匹配，*只能匹配0次
                  return matchCore(str,i,pattern,p+2);
              }
          }else {
              //pattern的下一个字符不是*
              if (str[i] == pattern[p] || pattern[p] == '.' && i != str.length){
                  return matchCore(str,i+1,pattern,p+1);
              }
          }
  
  
          return false;
      }
  
      public static void main(String[] args) {
          char[] str = {'a','b'};
          char[] pattern = {'a','b','a','*','a','*'};
          System.out.println(match(str, pattern) + "[" + true + "]");
      }
  }
  
  ~~~

  

  ## 测试结果

  ~~~
  true[true]
  ~~~

  









- 





