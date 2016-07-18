# Unit Testing

## What makes a good Unit-Test?

1. F.I.R.S.T. Principles
   * Fast
   * Isolated
   * Repeatable
   * Self-Verifying
   * Timely
2. Tests "one thing"
3. Readable / Maintainable

**Fast**
> Unittests should give near immediate feedback to changes to the production code.
> Hundreds of tests should be executable within seconds.

**Isolated**
> Tests must not depend on other tests to be executed.
> Each test must stand on its own.
> The order of execution should be non-deterministic.
> Tests must also be isolated from external influences!
> File system or Operating System must not change the tests behaviour.

**Repeatable**
> Unless something changed in the production code a test must always get the same result when executed.

**Self-Verifying**
> Unittest either *pass* OR *fail*.
> There must not be a manual decision involved!

**Timely**
> Tests should be created before production code (TDD).

**Tests "one thing"**
> There should be only on reason a unit test fails.
> If a single test tests more than one thing, it has more than one reason to fail.

**Readable / Maintainable**
> The only constant in software development is *change*.
> This means that tests will change over time.
> If the test isn’t written in an easy to understand format it will cost valuable time each time something changes.

## Naming

Names are important! A good name tells you what something does without you having to waste time analysing it.

### Test Classes

The test class' name should reflect which class is tested (CUT: Class Under Test).
In general the following pattern applies: ${CUT_NAME}Test.

```java
// The class under test
public class Foo {
    ...
}
 
// This class contains tests for 'Foo'
public class FooTest {
    Foo cut = new Foo();
    ...
}
```

### Test Methods

The test name should reflect it's context.
Names like test1(), test2() and happyPath() are meaningless because they don't say anything about what the test actually does.
A good test name helps the developer in case the test fails.
Nobody is looking at the name of a test that passed.
But when a test fails it should give a hint about what is wrong.
As a general rule a test's name should make sense when it's negated (in case of a failed test):

|  | Name | Interpretation in Case of Failure |  |
| --- | ------ | ----------------------------------- | --- |
| #1 | `test1()` | *"test 1 doesn't work"* | &#x1f44e; |
| #2 | `happyPath()` | *"happy path doesn't work"* | &#x1f44e; |
| #3 | `deletingAnEntityRemovesItFromTheCache()` | *"entity is not removed from cache"* | &#x1f44d; |

Try to imagine you are making some small changes to your production code and when you execute your tests a whole bunch of #1 or #2 tests fail.
Will you know which of your changes broke the tests?

No imagine all of the failed tests have names like #3.
Tests like these can actually give you an understanding of the problem before even having to look at the failure or inside the test.

**Here some pointers for finding a good name:**

*  Be as precise as possible about what aspect of the method under test you are validating.
*  Include the expected result inside you name. I.e. `nullIsAnInvalidArgument()`, `deletingAnEntityAlsoRemovesItFromCache()` etc.
*  Don't shy away from longer names. Brevity is not always the way to go.
  
**Example:**

```java
@RunWith(Enclosed.class)
public class FooTest {
 
    public static class SomeMethod extends FooTest {
 
        @Test
        public void canBeInvokedByAuthenticatedUser() {
            ...
        }
 
        @Test(expected=AuthenticationException.class)
        public void cannotBeInvokedByUnauthenticatedUser() {
            ...
        }
 
    }
 
}
```

## Scoping

One test should always test only one thing! Deciding what "one thing" means is harder than it sounds...

The test in the following example has a very ambiguous name. It has the name because it actually tests two aspects of persisting the entity and that name is the best fit for both:

* It verifies the save operation on the entity manager
* and the cleanup of the used entity cache.

Verifying both of these aspects of "persisting" in a single test makes it test two separate things.

```java
@Test
public void entityCanBePersisted() {
 
    Entity mockEntity = mock(Entity.class)
 
    cut.persist(mockEntity);
 
    verify(entityManager).save(mockEntity);
    verify(entityCache).remove(mockEntity);
 
}
```

This test can very easily be split into two distinct tests, each verifying a single aspect of the greater whole.

```java
@Test
public void persistingAnEntitySavesItToTheEntityManager() {
    Entity mockEntity = mock(Entity.class)
    cut.persist(mockEntity);
    verify(entityManager).save(mockEntity);
}
 
@Test
public void persistingAnEntityClearsItToFromTheCache() {
    Entity mockEntity = mock(Entity.class)
    cut.persist(mockEntity);
    verify(entityCache).remove(mockEntity);
}
```

In combination with very precise naming this split allows for an easier identification of the source of a failed test. Each of this test can only fail because of one single reason. (unexpected exceptions aside)

In this example the decision to split the single test into two separate tests is rather easy to make. In most cases it isn't that easy to known when and where to split. Developers should try to identify different aspects of the method under test which could be validated separately. In case these aspects are not obvious, writing a single test is the safer bet.
 
A few things that might signal a splitable test:
 
* Ambiguous test name: If a precise test name is hard to find, it might test more than one thing
* Test name containing "and": "persistingAnEntityStoresItInTheEntityManagerAndClearsCache"
* Many assertion / verify  operations on a number of different objects / mocks.
 
This does not mean there should be only one assert / verify per test! It is important to split tests logically, based on different functional aspects. How these aspects are validated is of no consequence!
