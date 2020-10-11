---
layout:     post
title:      gtest死亡测试代码覆盖率统计问题
subtitle:   
date:       2020-10-11
author:     Yannis 
catalog: true
tags:	
   - gtest
---

# 遇到的问题

前一段时间在处理代码覆盖率的问题，其中遇到这么一个场景，即需要测试代码的一些分支为死亡分支，比如下面这段代码，如果add函数输入的vector的维度不是一致的话程序就会报错。

```c++
#include <glog/logging.h>
template <typename Dtype>
void add(const vector<Dtype>& src1, const vector<Dtype>& src2,
         vector<Dtype>& des){
  if(src1.size() != src2.size()){
      LOG(FATAL) << "src1's size must equal to src2's size";
  }
  if(src2.size() != des.size()){
      LOG(FATAL) << "src's size must equal to des's size";
  }
  int size = src1.size();
  for(int i = 0; i < size; ++i){
    des[i] = src1[i] + src2[i];
  }
}
```

在用gtest测试的时候我们会采用死亡测试的手段来测试该add函数，相应的测试代码如下，测试代码构造了一个例子故意让运行add函数报错，然后用gtest的EXPECT_DEATH函数来捕捉该错误，这样按道理我们就可以会将add函数中关于LOG(FATAL)的代码运行一遍，然后在统计代码覆盖率的时候就会显示跑过上述LOG(FATAL)的代码。

```c++
TEST_F(UtilTest, TestAddDeathTest){
  std::vector<int> vec1;
  std::vector<int> vec2{1};
  std::vector<int> des(2);
  EXPECT_DEATH(add(vec1, vec2, des), "");
  vec1.resize(1, 2);
  EXPECT_DEATH(add(vec1, vec2, des), "");
}
```

然后我通过lcov这个工具来收集相应的代码覆盖率，但是却看到下图代码覆盖率显示面板上并没有覆盖到这两行代码。

![image-20201011170826378](https://github.com/yupeifengyannis/yupeifengyannis.github.io/blob/master/_posts/utils/coverage.png)

# 分析与解决

通过查找资料了解了一下lcov和gcov统计代码覆盖率的机制，这有一篇讲解gcov统计代码覆盖率原理的[博客](https://github.com/yanxiangyfg/gcov)，那这里简单的描述一下gcov统计代码覆盖率的基本原理。

## gcov基本原理

gcc中指定-ftest-coverage等代码覆盖率测试选项后，gcc会：

1、在输出目标文件中留出一段存储区保存统计数据。

2、在源代码中每行可执行语句生成的代码之后附加一段更新代码覆盖率统计结果的代码。

3、在可执行文件进入用户代码main函数之前会调用gcov_init内部函数初始化统计数据区，并将gcov_exit内部函数注册为exit_handlers函数。

4、用户代码调用**exit**正常结束时，gcov_exit函数会被调用，其继续调用__gcov_flush函数输出统计数据到*.gcda文件当中。

## 问题分析

有了gcov的基本知识，我们发现在前面的死亡测试过程中调用LOG(FATAL)函数时，被测程序并不是按照正常exit退出，而是发送一个信号给操作系统内核，然后强行将该程序终止掉，所以最后gcov_exit函数并未被调用到，因此最终的代码覆盖率统计数据中会显示这两段代码未被统计到。

## 解决措施

解决的方法也是比较简单，就是注册信号捕捉函数，信号捕捉函数捕捉到信号之后以exit的方式退出程序，退出码为信号编号，如果退出码不为0在gtest看来也是