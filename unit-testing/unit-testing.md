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

Unittests should give near immediate feedback to changes to the production code.
Hundreds of tests should be executable within seconds. A single test should not 
take longer than 10 ms.

**Isolated**

Tests must not depend on other tests to be executed. Each test must stand on its
own. Tests must also be isolated from external influences like time, file system
or any other form of resource!

**Repeatable**

Unless something changed in the production code a test must always produce the
same result when executed multiple times.

**Self-Verifying**

Unittest either *pass* OR *fail*. There must not be any human decision-making
involved!

**Timely**

If possible tests should be created before production code (Test Driven Development).

**Tests "one thing"**

There should be only on reason a test fails. If a single test tests more than
one thing, it has more than one reason to fail.

**Readable / Maintainable**

The only constant in software development is *change*. This means that tests
will change over time. If the test isn’t written in an easy to understand format
it will cost valuable time each time something changes.

In addition a well written and easy to understand test is the best documentation
a developer can get when trying to understand what a specific piece of software
does.

## Naming

Names are important! A good name tells you what something does without you
having to waste time analyzing it.

### Test Classes

The test class' name should reflect which class is tested (CUT: Class Under
Test). In general the following pattern applies: ${CUT_NAME}Test.

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

The test name should reflect it's context. Names like `test1()`, `test2()` and
`happyPath()` are meaningless because they don't say anything about what the test
actually does. A good test name helps the developer in case the test fails.
Nobody is looking at the name of a test that passed. But when a test fails it
should give a hint about what is wrong. As a general rule a test's name should
make sense when it's negated (in case of a failed test):

|  | Name | Interpretation in Case of Failure |  |
| --- | ------ | ----------------------------------- | --- |
| #1 | `test1()` | *"test 1 doesn't work"* | &#x1f44e; |
| #2 | `happyPath()` | *"happy path doesn't work"* | &#x1f44e; |
| #3 | `deletingAnEntityRemovesItFromTheCache()` | *"entity is not removed from cache"* | &#x1f44d; |

Try to imagine you are making some small changes to your production code and
when you execute your tests a whole bunch of #1 or #2 tests fail. Will you know
which of your changes broke the tests?

No imagine all of the failed tests have names like #3. Tests like these can
actually give you an understanding of the problem before even having to look at
the error message or the test code.

**Here some pointers for finding a good name:**

*  Be as precise as possible about what aspect of the method under test you are
validating.
*  Include the expected result inside you name. I.e. `nullIsAnInvalidArgument()`
, `deletingAnEntityAlsoRemovesItFromCache()` etc.
*  Don't shy away from longer names. Brevity is not always the way to go.
  
**Example:**

```java
@RunWith(Enclosed.class)
public class FooTest {
 
    public static class SomeMethod extends FooTest {
 
        @Test
        void canBeInvokedByAuthenticatedUser() {
            ...
        }
 
        @Test(expected=AuthenticationException.class)
        void cannotBeInvokedByUnauthenticatedUser() {
            ...
        }
 
    }
 
}
```

## Scoping

One test should always test only one thing! Deciding what "one thing" means is
harder than it sounds...

The test in the following example has a very ambiguous name. It has the name
because it actually tests two aspects of persisting the entity and that name is
the best fit for both:

* It verifies the save operation on the entity manager
* and the cleanup of the used entity cache.

Verifying both of these aspects of "persisting" in a single test makes it test
two separate things.

```java
@Test
void entityCanBePersisted() {
 
    Entity mockEntity = mock(Entity.class)
 
    cut.persist(mockEntity);
 
    verify(entityManager).save(mockEntity);
    verify(entityCache).remove(mockEntity);
 
}
```

This test can very easily be split into two distinct tests, each verifying a
single aspect of the greater whole.

```java
@Test
void persistingAnEntitySavesItToTheEntityManager() {
    Entity mockEntity = mock(Entity.class)
    cut.persist(mockEntity);
    verify(entityManager).save(mockEntity);
}
 
@Test
void persistingAnEntityClearsItToFromTheCache() {
    Entity mockEntity = mock(Entity.class)
    cut.persist(mockEntity);
    verify(entityCache).remove(mockEntity);
}
```

In combination with very precise naming this split allows for an easier
identification of the source of a failed test. Each of this test can only fail
because of one single reason. (unexpected exceptions aside)

In this example the decision to split the single test into two separate tests is
rather easy to make. In most cases it isn't that easy to known when and where to
split. Developers should try to identify different aspects of the method under
test which could be validated separately. One possible starting point could be
to write a big test and then try to find assertions and verifications which
build logical units. Than split these logical units into separate tests.
 
A few things that might signal a splitable test:
 
* Ambiguous test name: If a precise test name is hard to find, it might test
more than one thing
* Test name containing "and": "persistingAnEntityStoresItInTheEntityManagerAndClearsCache"
* Many assertion / verify  operations on a number of different objects / mocks.
 
This does not mean there should be only one assert / verify per test! It is
important to split tests logically, based on different functional aspects.
How these aspects are validated is of no consequence!

## State vs. Behavior Testing

There is a difference between testing how a method behaves internally and what
its result is. The **how** is commonly referred to as **behavior** testing and 
the **what** as **state** testing.

**State Testing** is ideally done in cases where there are no additional classes
involved. But since this is rarely the case, take a look at the following example:

```java
public class FooService {

    private FooTransformer transformer;
    private FooPersistenceService fooPs;

    public List<FooDto> getFoos(){
        return fooPs.getFoos().stream()
            .map(foo -> transformer.getDto(foo))
            .collect(toList());
    }

}

public class FooServiceTest {

    @Mock
    FooTransformer transformer;
    @Mock
    FooPersistenceService fooPs;

    @InjectMocks
    FooService cut;

    @Test
    void allFoosAreReturnedAsDtos() {
 
        // difine test data
        Foo foo1 = mock(Foo.class);
        Foo foo2 = mock(Foo.class);
        FooDto fooDto1 = mock(FooDto.class);
        FooDto fooDto2 = mock(FooDto.class);

        // simulate (mock) service behavior
        doReturn(fooDto1).when(transformer).getDto(foo1);
        doReturn(fooDto2).when(transformer).getDto(foo2);
        doReturn(Arrays.asList(foo1, foo2)).when(fooPs).getFoos();
 
        // execute method and assert result
        List<FooDto> result = cut.getFoos();
        assertThat(result).containsExactly(fooDto1, fooDto2);

    }

}
```

In this example there are two services which are mocked `FooTransformer` and
`FooPersistenceService`. The behavior of both is defined regarding the input
data. The test is written without verifying any behavior of the method under test.
Just the result is considered, not how it was produced.

We don't need to verify the behavioral aspects of the method since we can safely
assume that this test will break if there are any significant changes to the
method under test.

In order to make this test more stable we could use the real implementation of
`FooTransformer` instead of a mock since it does not have any external dependencies
which would break our test's scope (Unit). This way only the database access is
simulated.

**Behavior Testing** should be used in cases where there are certain aspects of
the method under test's behavior that are important. These could be aspects like
order of invokation, logging, exception handling. In addition there are certain
constelations where state testing can't be performed: void methods, delegation 
and verifying that something didn't happen.

As an example of behavior testing, let's take a look at an event handler:

```java
public class EventSystem {

    private List<EventListener> listeners = new ArrayList<>();

    public void register(EventListener listener){
        listeners.add(listener);
    }

    public void fire(Event event){
        listeners.forEach(listener -> listener.eventWasFired(event));
    }

}

public class EventSystemTest {

    @InjectMocks
    EventSystem cut;

    @Test
    void eventListenersAreCalledInOrderOfRegistration(){
        
        // prepare input data
        EventListener listener1 = mock(EventListener.class);
        EventListener listener2 = mock(EventListener.class);
        EventListener listener3 = mock(EventListener.class);
        Event event = mock(Event.class);

        // register listeners in specific order
        cut.register(listener2);
        cut.register(listener3);
        cut.register(listener1);

        cut.fire(event);
        
        // verify order of execution is same as registration
        InOrder inOrder = inOrder(listener1, listener2, listener3);
        inOrder.verify(listener2).eventWasFired(event);
        inOrder.verify(listener3).eventWasFired(event);
        inOrder.verify(listener1).eventWasFired(event);
        inOrder.verifyNoMoreInteractions();
        
    }

}
````

This example demonstrates clearly that not everything can be tested by state testing.
There is no way of testing the order of execution in this scenario without mocking.
It also very nicely demonstrates how behavior tests can provide an addition level
of testing by allowing to verify very functional aspects / requirements of methods.

**Keep the following in mind when testing behavior:**
* Don't verify every method invocation made by the method under test - concentrate
on the relevant calls for what you want to test!
* Don't verify the same thing over and over again - if there is already a test 
for it you don't need to test it again!
