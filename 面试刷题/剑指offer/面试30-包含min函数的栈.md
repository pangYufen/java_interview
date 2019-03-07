# 题目描述

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））

# 解题思路

利用一个辅助栈来存放最小值，把每次的最小元素（之前的最小元素和新压入栈的元素两者的较小值）都保存起来放到另外一个辅助栈里面。

![img](https://img-blog.csdn.net/20150803190225864?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![img](https://img-blog.csdn.net/20150803190236502?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

如果每次都把最小元素压入辅助栈，那么就能保证辅助栈的栈顶一直都是最小元素。当最小元素从数据栈内弹出之后，同时弹出辅助栈的栈顶元素，此时辅助栈的新栈顶元素就是下一个最小值。

# 代码实现

```java
import org.junit.Test;
import org.omg.PortableInterceptor.INACTIVE;

import java.util.Stack;

public class Test30 {
    /**
     * 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数
     */

    //数据栈
    Stack<Integer> datastack = new Stack<Integer>();
    //辅助栈
    Stack<Integer> minstack = new Stack<Integer>();


    public  void push(int node) {


        datastack.push(node);

        if (minstack.size() == 0){
            minstack.push(node);
        }else {
            //如果插入的数据比栈中的最小元素小
            if (node < minstack.peek()){
                minstack.push(node);
            }else {
                //如果插入的数据比栈中的最小元素大，复制最小栈栈顶元素，将其入栈
                minstack.push(minstack.peek());
            }
        }
    }

    public void pop() {
        // 如果栈已经为空，再出栈抛出异常
        if (datastack.isEmpty()){
            throw new RuntimeException("The stack is ready empty");
        }

        //两个栈同时出栈
        datastack.pop();
        minstack.pop();

    }

    public int top() {
        return datastack.peek();
    }

    public int min() {
        // 如果最小数公位置栈已经为空（数据栈中已经没有数据了），则抛出异常
        if (minstack.isEmpty()){
            throw new RuntimeException("No element in stack.");
        }

        return minstack.peek();
    }

    public static void main(String[] args) {
        Test30 stack = new Test30();
        stack.push(4);
        System.out.println(stack.min() == 4);

        stack.push(3);
        System.out.println(stack.min() == 3);

        stack.push(2);
        System.out.println(stack.min() == 2);

        stack.push(4);
        System.out.println(stack.min() == 2);

        stack.pop();
        System.out.println(stack.min() == 2);

        stack.pop();
        System.out.println(stack.min() == 3);

        stack.pop();
        System.out.println(stack.min() == 4);

//        stack.pop();
//        System.out.println(stack.min() == 4);

        stack.push(2);
        System.out.println(stack.min() == 2);

        stack.push(0);
        System.out.println(stack.min() == 0);
     }
}
```

# 测试结果

~~~
true
true
true
true
true
true
true
true
true
~~~





