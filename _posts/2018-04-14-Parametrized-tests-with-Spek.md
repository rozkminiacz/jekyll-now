![Header](https://raw.githubusercontent.com/rozkminiacz/rozkminiacz.github.io/master/_posts/spek-parametrized-header.png)



Well, if you want have different input on your tests while using jUnit, you have to use parametrized runner and add some methods with dedicated annotations.
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
|--- |--- |
|753|753m|
|1337|1.34km|
|15888|15.9km|
|31270|31km|
|51000|>50km|


It's simple case - implementation is just one ```switch``` or ```when``` with function to round number with desired precision.

However, we can write specification and test this simple case using Spek. 

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


What happened? We are still repeating ourselves. It is as long as ```@Parametrized``` jUnit test or just test with many methods.

Spek DSL is huge lambda - we can use any Kotlin language features.

Let's start with defining our test cases and expected outcome in ```map<Double, String>```:

```kotlin
val testCases = mapOf(
                61888.123 to ">50km",
                38777.23 to "38.8km",
                16984.44 to "17.0km",
                987.98 to "988m"
        )
```

Now we can use ```forEach{}``` to generate test cases:

```kotlin
testCases.forEach { value, expectedValue ->
            on("$value"){
                it("should print $expectedValue"){}
            }
        }
```

And finally we can make some assertion. Full example will look like this:

```kotlin
class DistanceConverterSpecificiation : Spek({
    describe("distance converter") {
        val testCases = mapOf(
                61888.123 to ">50km",
                38777.23 to "38.8km",
                16984.44 to "17.0km",
                987.98 to "988m"
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

![Parametrized test outcome](https://raw.githubusercontent.com/rozkminiacz/rozkminiacz.github.io/master/_posts/spek-parametrized.png)


