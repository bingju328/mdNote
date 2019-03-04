

2）变量不需要与其他的状态变量共同参与不变约束。

例：

volatile static int start = 3;

volatile static int end = 6;

线程A执行    while(start<end){/*do something*/}

线程B执行    start+=3: end+=3;
