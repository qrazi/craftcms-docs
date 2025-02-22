# Testing

## Types of tests

There are a variety of tests each providing pros and cons to specific use cases and functionalities within your application/module/plugin. Craft currently supports the following four test types.

## Manual testing

Everyone has and probably does manual testing when developing with Craft. Manual testing can be defined in the following steps

- Write some code
- Press `f5` or trigger a controller action
- Verify the result by seeing the result in the browser, IDE or database. 

Manual testing is the most time consuming but often the most effective way to catch bugs in the *primary* implementation of code - it also takes a lot of time and fails in certain key areas. Most importantly - if you make a change to a codebase in one place - it can very likely fail in other places, which you are not manually testing.

It would be woefully inefficient to test your **entire** application after each `git push`. This is where **automated** tests can help.

::: tip Testing is all about strategy and approaches. Manual testing and automated testing work best together. You can use your judgement to detect/prevent issues that computers cannot see whilst computers can execute many tests in short time.  
:::

### Unit testing

Many definitions exist regarding unit testing. Fundamentally a unit test is focused on testing an individual 'unit' of your code. In practice, this will often mean testing the results of a function or in some cases, class.

Consider the following class.

```php
class SalaryChecker {
    public function multiply(int $a, int $b) : int
    {
        return $a * $b;
    }

    public function add(int $a, int $b) : int
    {
        return $a + $b;
    }
}
```

If you were to unit test this class you would write something like follows:

```php
use Codeception\Test\Unit;
class MyTest extends Unit {
    public $salaryChecker;

    public function _before()
    {
      parent::_before(); 

      $this->salaryChecker = new SalaryChecker();
    }
    public function testMultiply()
    {
        $this->assertSame(
            4, 
            $this->salaryChecker->multiply(2, 2)
        );
    }
    public function testAdd()
    {
        $this->assertSame(
            3, 
            $this->salaryChecker->add(2, 1)
        );
    }
} 
```

The anatomy of this test can be defined somewhat as follows:

1. In the `_before` hook we create a `new SalaryChecker()` which can be used during the test. The `_before` method is run before *every* test.

2. A test is executed and a method of the `SalaryChecker` class is called. An **assertion** (checking that when passing 2 twice, 4 is returned - because the 2's are multiplied) is then made regarding its return result.

Fundamentally - that is a unit test. Now imagine a developer was to change the `add` method of the `SalaryChecker` class to the following:

```php
public function multiply(int $a, int $b) : int
{
    $result = $a * $b;

    // Dont return salaries lower than 25000
    if ($result < 25000) {
        return 25000
    }

    return $result;
}
```

The test would fail. Obviously, this is a basic example but as your codebase expands/changes, new devs join or old dev's leave and/or project requirements differ from the original spec more and more code will become dependant on each other.

Your unit tests will primarily cover your [service](../extend/services.md) classes. It is not recommended to test *every* method that your service class has. Use your best judgement and try to test methods as high up in the [call stack/backtrace](https://www.php.net/manual/en/function.debug-backtrace.php) as possible (excluding your controllers - they are covered by functional and acceptance testing).

Having good unit tests ensures that your individual functions work correctly, and if they don't, quickly catch and fix bugs relating this hereto.

It is recommended to read the Codeception documentation on [unit tests](https://codeception.com/docs/05-UnitTests) as well to see more practical examples of unit tests.

### Functional & acceptance testing

Your application isn't just a collection of PHP classes on a server. These classes work together to create an end product. These methods are often linked via controllers. The end product will then be shipped to a user via these controllers. The controller actions are the place where your application functionality is encompassed into a usable package - seems a good place to write tests for?

Typically a controller will:

- 1. Process a request (authentication, authorization, request types e.t.c.)
- 2. Invoke craft services
- 3. Return a response

Point 2 is covered by unit tests - 1 and 3 are covered by functional and acceptance tests.

What separates functional and acceptance tests from unit tests are that they are conducted from the **user** perspective. Consider the following twig template located at route `/pages/bob`:

```twig
Hi {{ currentUser.firstName }}

Welcome to this app
```

For this page/template you would create the following test class (Assuming you are creating a functional test):

```php
<?php
use FunctionalTester;
class FunctionalCest {
    public function testWelcomeMessage(FunctionalTester $I)
    {
        $I->amLoggedInAs($userWithFirstNameBob);
        $I->amOnPage('/pages/bob');
        $I->see('Hi Bob');
        $I->see('Welcome to this app');
    }
}
```

Dont worry about `$userWithFirstNameBob`. Just pretend that this variable is an instance of `craft\elements\User` where `$firstName = "bob"`.

Notice how the test reads very much like instructions that you *could* give to a human to perform on a production version of your application.

Underneath the functional test actually triggers the controller associated with this route. If you have a module/plugin you can also pass in for example `?p=actions/my-plugin/my-controller/my-action` which will test your controller actions.

You can even test the CP functionality by passing in a url that starts with the [cpTrigger](../config/config-settings.md#cptrigger) config (I.E with a `cpTrigger` of `admin` you could do `$I->amOnRoute('/admin/my-plugin/my-route/my-action)`). ::: tip Acceptance and functional tests are quite similar with some subtle differences in their *implementation*. See the [codeception docs](https://codeception.com/docs/01-Introduction) for an explanation hereto. :::