# Tail recursion with Akka Future

**Intent**

Akka is used for asyncronous communication using akka actor system. There are cases where we have to make recursive call based on result. It is known that we should try to avoid Await.result, or Thread.sleep in akka programming.

This design approach will provide alternative way of Await.result or Thread.sleep specially when you are using Akka "ask" to wait for result/responses.

**Problem**

Let's first understand how we can do tail recusion using Scala from simple sum of list of integers:

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
If we rewrite same problem; with assumtion of that sum is time consuming process and may take time to complete, so perform same using Asyncronous way using Future: 

```scala
import scala.annotation.tailrec
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
def sum(list:List[Int]):Future[Int]={
  @tailrec
  def sumNum(acc:Future[Int],list: List[Int]): Future[Int] = list match{
    case Nil=>Future.successful{0}
    case head::Nil => acc.map[Int](x=>x+head)
    case head::tail=> sumNum(acc.map[Int](x=>x+head),tail)
  }
  sumNum(Future.successful[Int]{0},list)
}

sum(List(1,2,3)).onComplete{
  case Success(v)=> Console.println(v)
  case Failure(ex)=>Console.println(ex.printStackTrace())
}

```


**Solution**

