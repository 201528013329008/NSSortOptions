# NSSortOptions
转自http://blog.csdn.net/miscellaner/article/details/41870183

数组排序中NSSortOptions两个参数详解（NSSortConcurrent、NSSortStable）

我们在进行数组排序的时候，很多时候可以根据参数理解意思，并且给定自己要传递的参数。在OC中数组排序的时候，其它方法都有对应的解释，但是唯独

    “-sortedArrayWithOptions:usingComparator：”这个方法的参数没有解释清楚，网上其它博客和技术文章都给出了一些“正序”、“稳定”、“并发”的错误解释，我研究一段时间，在此给出较为正确地解释 ，下面先给大家一个例子，来让大家看一下使用场景。

     

[objc] view plain copy
NSArray *array = [NSArray arrayWithObjects:@"123",@"21",@"33",@"69",@"108",@"256", nil nil];  
NSArray *resultArray = [array sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(id obj1, id obj2) {  
    int nu1 = [obj1 intValue];  
    int nu2 = [obj2 intValue];  
    if (nu1 > nu2) {  
        return NSOrderedDescending;  
    }else if (nu1 == nu2){  
        return NSOrderedSame;  
    }else{  
        return NSOrderedAscending;  
    }  
}];  
NSLog(@"%@",resultArray);  

 

   上面的例子中，我们使用了给定的排序方法，这个时候我们能正确地排列数组，如果我们不写“int nu1 = [obj1 intValue]和int nu2 = [obj2 intValue]”排序出来的结果不是按照大小，而是按照ASCII码表排序，这个大家都能理解。然后我们试着把方法中的Options参数改变一下，讲NSSortConcurrent换成，NSSortStable。发现结果是一样的，那么我们来甄别一下到底有什么区别。


      当然，代码熟练的同学可以直接使用下面这种方式来进行比较，结果和作用是一样的。

[objc] view plain copy
NSMutableArray *array = [@[@1,@3,@9,@4,@2]mutableCopy];  
  
[array sortWithOptions:NSSortStable usingComparator:^NSComparisonResult(id obj1, id obj2) {  
   
    return [obj1 compare:obj2];  
      
}];  
  
NSLog(@"%@",array);  

   首先concurrent，是“并发的，一致的，同时发生的”，stable，是“稳定的”。那意思就是前者的算法比后者执行速度快，后者比前者执行的方法更稳定？我们参阅一下说明来解决一下问题，先给大家说一下概念性的区别。
   我们知道后面使用的是一个Block，Block是代码块。 如果我们的参数选择的是NSSortConcurrent，那么我们的代码块用的是并行排序操作，这个选项仅仅是一个提示，并且可能在某些情况下地实现会被忽略。对NSSortConcurrent的并发操作，必须保证代码的安全调用。如果我们的参数选择的是NSSortStable，指定的排序结果应该返回的比较项，是对初值（这里指顺序）进行了改变。

   懵了么？好了亲，其实就是并行和串行的排序。并行就是多线程的排序方式，串行就是单步执行，没有多线程。如果你还没学多线程，那么简单解释起来就是两个顺序结构的语句同时执行，并行的话，有可能出现还没排好顺序，你就使用数组的情况，而且为了安全性还不一定执行这个多线程。如果你学了多线程，那就是说，排序的时候在子线程操作，所以可能出现的情况就是子线程还没执行完，主线程执行好了，那你用数组的时候...你懂。所以我们一般情况下应该使用的是NSSortStabel，就是没有特殊操作的排序，对原始数据操作之后返回排序好的结果。

   现在我们明白了，两者并不会影响排序中是按照正序还是反序排，只影响效率和稳定性，我们根据自己的取舍就OK，我建议使用NSSortStable，除非计算偏大量的时候我们采用NSSortConcurrent。
    
enum{
 
NSSortConcurrent=(1UL<<0),
   
NSSortStable=(1UL<<4),
 };

是不是还有朋友觉得这个枚举格式很奇怪，那我们就说一下，首先UL是无符号长整型的标志（unsigned
 long），<<和>>都是位运算，<<是左移运算符，是将原数的各位二进制位左移对应的操作数位数，操作数就是后面的数。其实理解起来也不难，就是写成二进制，根据箭头方向移动值，缺位补零。至于具体的使用，到UI的开发阶段大家会频繁接触类似的枚举，理解起来也更为的方便。
