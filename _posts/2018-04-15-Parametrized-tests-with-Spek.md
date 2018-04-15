![Header](https://raw.githubusercontent.com/rozkminiacz/rozkminiacz.github.io/master/_posts/distracted-spek-junit.jpg)

Everyone knows the situations when we have to check our system for various input. 
A good example would be login/password validation performed both on front and backend. 
There are many approaches to create a test code which covers all necessary cases, 
we can just copy-paste methods (not recommended), make many assertions in one test method (we can lost some valuable error info) 
or use parametrized test functionality bundled within test framework we are using. Keep in mind that there are more options available.

Well, if you want have different input on your tests while using jUnit4, you have to use parametrized runner and add some methods with dedicated annotations.
And implement method returning list of parameters and expected output.

```java
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {     
                 { 0, 0 }, { 1, 1 }, { 2, 1 } 
           });
    }

```

Pretty ugly, huh? Full example can be found [here](https://github.com/junit-team/junit4/wiki/Parameterized-tests)


## We can do better
Imagine, if we could write our tests like we write our code. Clear output, no annotation driven development, and
no unnecessary static methods. **That's were Spek comes to help**.

>If you are not familiar with using Spek in unit testing check my [article](http://rozkmin.me/Model-View-Presenter-writing-Spek-specification/) or other blog posts.

Consider simple distance converter - we parse given distance in meters to desired unit. 

|value [m]|displayed string|
|:---: | :---: |
|753|753 m|
|1337|1.34 km|
|15888|15.9 km|
|31270|31 km|
|51000|>50 km|

It's simple case - implementation is trivial. It only takes one ```switch``` or ```when``` and function to round number with desired precision to solve this case.

Despite of the fact, that implementation is simple, we should prepare unit test it. Using Spek we can also write BDD style tests and prepare a specification at one approach.

```kotlin
class DistanceConverterSpecificiation : Spek({
    describe("distance converter") {
        val distanceConverter: DistanceConverter by memoized {
            DistanceConverter()
        }
        on("distance 61888.123m") {
            val testDistance = 61888.123
            it("should return >50km") {
                val convert = distanceConverter.convert(testDistance)
                assertTrue(convert.contentEquals(">50km"))
            }
        }
        on("distance 38777.23m") {
            val testDistance = 38777.23
            it("should return 38.8km") {
                val convert = distanceConverter.convert(testDistance)
                assertTrue(convert.contentEquals("38.8km"))
            }
        }
    }
})
```


Run this snippet and check results? We have tested two cases, however we are still repeating ourselves. It is as long and ugly as ```@Parametrized``` jUnit test or just test with many methods.

>Spek DSL is a huge lambda, so we can use any Kotlin language features.

Let's start with defining our test cases and expected outcome using ```Map<Double, String>```:

```kotlin
val testCases = mapOf(
                61888.123 to ">50 km",
                38777.23 to "38.8 km",
                16984.44 to "17.0 km",
                987.98 to "988 m"
        )
```

Now we can use ```forEach{}``` to generate desired test cases:

```kotlin
testCases.forEach { value, expectedValue ->
            on("$value"){
                it("should print $expectedValue"){}
            }
        }
```

Finally we should make some assertion. Full example will look like this:

```kotlin
class DistanceConverterSpecificiation : Spek({
    describe("distance converter") {
        val testCases = mapOf(
                61888.123 to ">50 km",
                38777.23 to "38.8 km",
                16984.44 to "17.0 km",
                987.98 to "988 m"
        )
        val converter = DistanceConverter()
        testCases.forEach { value, expectedValue ->
            on("$value"){
                it("should print $expectedValue"){
                    assertTrue(converter.convert(value).contentEquals(expectedValue))
                }
            }
        }
    }
})
```

Run this snippet and check results.

![Parametrized test outcome](https://raw.githubusercontent.com/rozkminiacz/rozkminiacz.github.io/master/_posts/spek-parametrized.png)

## Conclusion

When creating unit tests we have to be sure that all (reasonable) cases are covered. In provided example (distance converter) we used ```Map<Double, String>``` and ```forEach{k,v -> }``` 
to parametrize our test and create ```on()``` ActionBody and ```it()``` TestBody in a stream. 
We have performed checks and on each value, asserted expected outcome and displayed test results nicely in our IDE. 
We haven't lost any valuable info, and we can add another test cases in clear Kotlin syntax.

Using Spek gives you full power of Kotlin - you can write your tests just like you write your code and make use of language features.

I hope that you find this article useful, and it will help you looking at parametrized tests in a different way.