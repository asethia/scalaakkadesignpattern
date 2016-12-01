<h1>Query and Update/Insert Design</h1>

**Intent**

Asyncronous Insert or update record if not found in the database. 


**Problem**

Most of time it is seen that we would like to insert/update record only if the given record is not presnent based on priary key constraint. This has been done as two steps approach:

1. Query the information 
2. If found then insert / update 
3. else Generate error "Record already exist"

User would like to perform these operations asyncronous non blocking process. In typical way using Akka it is been done using sample code snippet; where queryModel is model how to query the information and insertModel provides insert entity information.
 ```scala
 queryActor.ask(queryModel).mapTo[Boolean].onComplete{
   case Success(v)=> v match{
        case true=> insertQueryActor.ask(insertModel).mapTo[Boolean]
        case false => ResultError("Record already exist")
    }
   case Failure(ex)=>  SystemError("Not able to process")
 }   
```
**Solution**

We can create a model which can wrap query and insert model together, for example:

```scala
case class QueryInsertModel(query:QueryModel,insert:InsertModel)
```
this can help to generalize the find update/insert operations at actor level as unified way.

We can use Scala ```scala map ``` high order function to perform async processing:

```scala
 queryActor.ask(queryInsertModel.query).mapTo[Boolean].map{
     case true=> insertQueryActor.ask(queryInsertModel.insert).mapTo[Boolean]
     case false => Future.failed(ResultError("Record already exist"))
  }   
```
If this result should be sent to caller, we can use ```scala flatMap ``` highorder function along with ```scala akka.pattern.pipeTo ``` for the same as:

```scala
 queryActor.ask(queryInsertModel.query).mapTo[Boolean].map{
     case true=> insertQueryActor.ask(queryInsertModel.insert).mapTo[Boolean]
     case false => Future.failed(ResultError("Record already exist"))
  }.flatMap(identity) pipeTo (sender())
```



