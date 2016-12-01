# Tail recursion when function returns Future

**Intent**
Scala functional language feature allows user to write recursion rather than iteration. It allows user to overcome the limitations of the call stack.

It becomes challenging when a function computation returns Future in best scenario, and tail recursion is not allowed. There are multiple ways we can solve this problem, but we will leverage Akka actor to overcome this issue.

**Problem**

Let's go through sum problem using tail recusion:

```scala
import scala.annotation.tailrec
def sum(list:List[Int]):Int={
   @tailrec
   def sumNum(acc:Int,list: List[Int]): Int = list match{
       case Nil=>0
       case head::Nil => acc+head
       case head::tail=> sumNum(acc+head,tail)
   }
   sumNum(0,list)
}
```
If we rewrite same porblem (which is not IO bound) and still do with Future



**Solution**

