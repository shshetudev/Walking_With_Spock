## Specification

* A specification is represented as a Groovy class that extends from `spock.lang.Specification`.
* Class `Specification` contains a number of useful methods for writing specifications.
* Furthermore, it instructs JUnit to run specification with `Sputnik`, Spock's JUnit runner

## Fields

> def obj = new ClassUnderSpecification()
>
> def coll = new Collaborator()

* Initializing instant fields ay the point of declaration is a good practice.
* Semantically it's good practice to initialize them right at the point of declaration.
* We can share a very expensive resource using `@Shared` field.

> @Shared res = new VeryExpensiveResource()

* Static fields should only be used for constants, Otherwise shared fields are preferable.

> static final PI = 3.141592654

## Fixture Methods

> def setupSpecs() {} // runs once - before the first feature method
>
> def setup() {} // runs before every feature method
>
> def cleanup() {} // runs after every feature method
>
> def cleanupSpec() {} // runs once - after the last feature method

* These methods are responsible for setting up and cleaning up the environment in which feature methods are run.
* It's a good idea to use a fresh fixture for every feature method, which is what the `setup()` and `cleanup()` methods
  are for.
* All fixture methods are optional.
* The `setupSpec()` and `cleanupSpec()` may not reference instance fields unless they are annotated with `@Shared`.

## Invocation Order

`super.setupSpec > sub.setupSpec > super.setup > sup.setup > feature method > sub.cleanup > super.cleanup > sub.cleanupSpec > super.cleanupSpec`
.

## Feature Methods

> def "pushing an element on the stack"() {
>
> //block go here
>
> }

* By convention, feature methods are named with String literals.
* Conceptually, a feature method consists of four phases:
    * Set up the feature's fixture.
    * Provide a stimulus to the system under specification.
    * Describe the response expected from the system.
    * Clean up the feature's fixture.
* Whereas the first and last phases are optional, the stimulus and response phases are always present, and may occur
  more than once.

## Blocks

* There are six kinds of blocks: `given`, `when`, `then`, `expect`, `cleanup`, `where`.
* Any statements between the beginning of the method and the first explicit block belong to implicit given block.
* A feature method must have at least one explicit block.
* Blocks divide a method into distinct sections and can not be nested.
  > given: Setup
  >
  > when: Stimulus
  >
  > then: Response
  >
  > expect: Stimulus, Response
  >
  > cleanup: Cleanup
  >
  > where: over all blocks

## Given Blocks

> given:
>
> def stack = new Stack()
>
> def elem = "push me"

* The block where we do any setup work for the feature that we are describing.
* It is an optional or implicit , no repeated block.

## When and Then Blocks

* The `when-then` blocks always occur together. They describe stimulus and expected response.
* Whereas `when` blocks may contain arbitrary code, `then` blocks are restricted
  to `conditions, expection conditions, interactions and variable definitions`.
* A feature method may contain multiple pairs of `when-then` blocks.

## Conditions

> when: <br/>
> stack.push(elem) <br/>
> then: <br/>
> !stack.empty <br/>
> stack.size() == 1 <br/>
> stack.peek() == elem <br/>

* Conditions describe an expected state, much like JUnit's assertions.
* However, conditions are written as plain boolean expression, eliminating the need for an assertion API.
* Try to keep the number of conditions per feature method small. One to five conditions is a good guideline.
* If conditions define multiple unrelated features at once, breaking up into small methods is better.
* If conditions differ only in their values, consider `data table` is better.

## Implicit and explicit conditions

* Conditions are an essential ingredient of `then` blocks and `except` blocks.
* Except for calls to void methods and expressions classified as interactions, all top-level expressions in these blocks
  are implicitly treated as conditions.
* To use conditions in other places, we need to designate them with Groovy's assert keyword.
  > def setup() { <br>
  stack = new Stack() <br>
  assert stack.empty <br>
  } <br>

## Exception Conditions

> when: <br>
> stack.pop() <br>
> then: <br>
> def e = thrown(EmptyStackException) <br>
> e.cause == null
> stack.empty <br>

* Exceptions conditions are used to describe that a `when` block should throw and exception.
* They are defined using the `thrown()` method, passing along the expected exception type.
* Exception conditions my be followed by other condtions.
    * We can also write down like this:
  > when:<br>
  stack.pop()<br>
  then:<br>
  EmptyStackException e = thrown()<br>
  e.cause == null<br>
    * This syntax has two small advantage, first exception variable is strongly typed.
    * Second, the condition reads a bit more like a sentence ("then an EmptyStackException is thrown").
    * Sometimes we need to convey that an exception should not be thrown:
  > def "HashMap accepts null key"() {<br>
  > given:<br>
  > def map = new HashMap()<br>
  >
  >  when:<br>
  > map.put(null, "elem")<br>
  >
  >  then:<br>
  > notThrown(NullPointerException)<br>
  > }
    * By using `notThrown()`, we make it clear that in particular a NullPointerException should not be thrown.

## Interactions

* Whereas conditions describe an object's state, interactions describe how objects communicate with each other.
    * For example, we suppose we want to describe the flow of events from a publisher to its subscribers:
  > def "events are published to all subscribers"() {<br><br>
  given:<br>
  def subscriber1 = Mock(Subscriber)<br>
  def subscriber2 = Mock(Subscriber)<br>
  def publisher = new Publisher()<br>
  publisher.add(subscriber1)<br>
  publisher.add(subscriber2)<br><br>
  when:<br>
  publisher.fire("event")<br><br>
  then:<br>
  1 * subscriber1.receive("event")<br>
  1 * subscriber2.receive("event")<br>
  }

## Expect Blocks

* In `expect` block, it may only contain conditions and variable definitions.
* It's useful in situations where it is more natural to describe stimulus and expected response is single expression.

> when:// 1st way to check<br>
> def x = Math.max(1, 2)<br>
> then:<br>
> x == 2<br><br>
> expect: // 2nd way to check<br>
> Math.max(1, 2) == 2<br>

## Cleanup Blocks

> given:<br>
> def file = new File("/some/path")<br>
> file.createNewFile()<br>
> // ...<br>
> cleanup:<br>
> file.delete()<br>

* A `cleanup` block may only be followed by a where block, and may not be repeated.

## Where Blocks

> def "computing the maximum of two numbers"() {<br>
> expect:<br>
> Math.max(a, b) == c<br>
> where:<br>
> a << [5, 3]<br>
> b << [1, 9]<br>
> c << [5, 9]<br>
> }

* A `where` block always comes last in a method, and may not be repeated.
* It is used to write data-driven feature methods.
* This `where` block effectivly creates two "versions" of the feature method: One where `a` is `5`, `b` is `1` and `c`
  is `5`.
* Another where `a` is `3`, `b` is `9` and `c` is `9`.
* Although it is declared last, the where block is evaluated before the feature method containing it runs.

## Helper methods

> def "offered PC matches preferred configuration"() {<br><br>
> when:<br>
> def pc = shop.buyPc()<br><br>
> then:<br>
> matchesPreferredConfiguration(pc)<br>
> }<br><br>
> def matchesPreferredConfiguration(pc) {<br>
> assert pc.vendor == "Sunny"<br>
> assert pc.clockRate >= 2333<br>
> assert pc.ram >= 4096<br>
> assert pc.os == "Linux"<br>
> }

## Using `with` for exceptions

* As an alternative to the above helper methods, you can use a with(target, closure) method to interact on the object
  being verified.

> def "offered PC matches preferred configuration"() {<br>
<br>when:<br>
> def pc = shop.buyPc()<br>
<br>then:<br>
> with(pc) {<br>
> vendor == "Sunny"<br>
> clockRate >= 2333<br>
> ram >= 406<br>
> os == "Linux"<br>
> }<br>
> }

* When verifying mocks, a with statement can also cut out verbose verification statements.

> def service = Mock(Service) // has start(), stop(), and doWork() methods<br>
> def app = new Application(service) // controls the lifecycle of the service<br>
<br>when:<br>
> app.run()<br>
<br>then:<br>
> with(service) {<br>
> 1 * start()<br>
> 1 * doWork()<br>
> 1 * stop()<br>
> }

## Using `verifyAll` to assert multiple expectations together

> def "offered PC matches preferred configuration"() {<br>
<br>when:<br>
> def pc = shop.buyPc()<br>
<br>then:<br>
> verifyAll(pc) {<br>
> vendor == "Sunny"<br>
> clockRate >= 2333<br>
> ram >= 406<br>
> os == "Linux"<br>
> }<br>
> }

## Extensions

| Anntoations   | Used for       |
|---------------|:---------------|
| @Timeout     | Sets a timeout for execution of a feature or fixture method.  |
| @Ignore     | Ignores any feature method carrying this annotation.       |
| @IgnoreRest |
Any feature method carrying this annotation will be executed, all others will be ignored. Useful for quickly running just a single method.     |  
| @FailsWith | Expects a feature method to complete abruptly. @FailsWith has two use cases: First, to document known bugs that cannot be resolved immediately. Second, to replace exception conditions in certain corner cases where the latter cannot be used (like specifying the behavior of exception conditions). In all other cases, exception conditions are preferable.     | 

## Reference:

* https://spockframework.org/spock/docs/2.1/index.html
