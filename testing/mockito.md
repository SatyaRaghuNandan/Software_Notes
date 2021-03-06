Mockito
---

# Mock vs. Spy in Mockito
 
When Mockito creates a mock – it does so from the Class of an Type, not from an actual instance. The mock simply creates a bare-bones shell instance of the Class, entirely instrumented to track interactions with it.

On the other hand, the spy will wrap an existing instance. It will still behave in the same way as the normal instance – the only difference is that it will also be instrumented to track all the interactions with it.

In the following example – we create a mock of the ArrayList class:

```
@Test
public void whenCreateMock_thenCreated() {
    List mockedList = Mockito.mock(ArrayList.class);
 
    mockedList.add("one");
    Mockito.verify(mockedList).add("one");
 
    assertEquals(0, mockedList.size());
}
```

As you can see – adding an element into the mocked list doesn’t actually add anything – it just calls the method with no other side-effect.

A spy on the other hand will behave differently – it will actually call the real implementation of the add method and add the element to the underlying list:

```
@Test
public void whenCreateSpy_thenCreate() {
    List spyList = Mockito.spy(new ArrayList());
 
    spyList.add("one");
    Mockito.verify(spyList).add("one");
 
    assertEquals(1, spyList.size());
}
```
 
In a mock all methods are stubbed and return "smart return types". This means that calling any method on a mocked class will do nothing unless you specify behaviour.

In a spy the original functionality of the class is still there but you can validate method invocations in a spy and also override method behaviour.

---


# PowerMockito 

PowerMockito is a PowerMock’s extension API to support Mockito. It provides capabilities to work with the Java Reflection API in a simple way to overcome the problems of Mockito, such as the lack of ability to mock final, static or private methods.

## Mock private method call with PowerMockito

```java

public class PowerMockDemo {
 
    public Point callPrivateMethod() {
        return privateMethod(new Point(1, 1));
    }
 
    private Point privateMethod(Point point) {
        return new Point(point.getX() + 1, point.getY() + 1);
    }
}
```


### Unit test

```java

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
 
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.mockito.Matchers.anyObject;
import static org.mockito.Mockito.mock;
 
@RunWith(PowerMockRunner.class)
@PrepareForTest(PowerMockDemo.class)
public class PowerMockDemoTest {
 
    private PowerMockDemo powerMockDemoSpy;
 
    @Before
    public void setUp() {
        powerMockDemoSpy = PowerMockito.spy(new PowerMockDemo());
    }
 
    @Test
    public void testMockPrivateMethod() throws Exception {
        Point mockPoint = mock(Point.class);
 
        PowerMockito.doReturn(mockPoint)
            .when(powerMockDemoSpy, "privateMethod", anyObject());
 
        Point actualMockPoint = powerMockDemoSpy.callPrivateMethod();
 
        assertThat(actualMockPoint, is(mockPoint));
    }
}
```

---

## Deep Stubs in Mockito

`Answer<Object> org.mockito.Mockito.RETURNS_DEEP_STUBS`

Optional Answer to be used with Mockito.mock(Class, Answer).

Example that shows how deep stub works:

```
   Foo mock = mock(Foo.class, RETURNS_DEEP_STUBS);

   // note that we're stubbing a chain of methods here: getBar().getName()
   when(mock.getBar().getName()).thenReturn("deep");

   // note that we're chaining method calls: getBar().getName()
   assertEquals("deep", mock.getBar().getName());
```

WARNING: This feature should rarely be required for regular clean code! Leave it for legacy code. Mocking a mock to return a mock, to return a mock, (...), to return something meaningful hints at violation of Law of Demeter or mocking a value object (a well known anti-pattern).

Good quote I've seen one day on the web: every time a mock returns a mock a fairy dies.

Please note that this answer will return existing mocks that matches the stub. This behavior is ok with deep stubs and allows verification to work on the last mock of the chain.