# Testing

The purpose of this document is to provide a set of guidelines and best practices for the development of tests at [DearDoc](https://getdeardoc.com/).

This document is a living document and will be updated as the team grows and the company evolves.

## Table of Contents

- [Testing](#testing)
- [Naming](#naming)
- [Test file names and structure](#Test-file-names-and-structure)
- [Mocking](#mocking)
- [External API Calls](#external-api-calls)
- [Clean up](#clean-up)
- [External Resources](#external-resources)

## Naming

### Test file names and structure

Test file names should ideally identify what type of test it is:

 - Unit tests: *.unit.test.ts
 - Integration tests: *.integration.test.ts

Being able to separate and quickly identify the test type allows us to specify different configuration settings
if necessary. For example, we can configure separate test coverage on unit tests and integration tests.

```
src/jest.config.js
src/jest.integration.config.js
```

Each configuration may include the type of tests it should match against:

```
testMatch: ["**/integration.test.ts"]
````

The location of these test files is quite important as well. Having a unit test file along the file its testing against
is helpful for quickly finding it instead of searching for it somewhere else where multiple tests could be located.
For example:

```
src/user.ts
src/user.unit.test.ts
```

For integration tests, these usually test multiple services or functions so it makes sense to group them together
inside the nearest __tests__ directory.

```
src/user.service.ts
src/db.service.ts
src/__tests__/user.service.integration.test.ts
```

## Mocking

Mocking functions or classes is very useful when we want to focus our tests on the bahavior we are interested in and not on the dependencies of it. Mocking allows us to:

- Erase or override the actual implementation of a function
- Capture calls to the mocked function
- Capture instances of constructor functions

With this, we are able to control those dependencies and set the desired outcome or expectations in our tests.

### External API calls

API mocking is a very useful technique that allows us to test an external API without actually making real HTTP calls.
This reduces the duration of our tests, removes an external dependency and we can modify it's behavior to fit our required tests.

The API mock responses and errors should match actual responses and errors from the API. The idea is to be able to test as if real HTTP calls were being made.

### Clean up

It is very important to do some clean up every time a function is mocked. If the mocks aren't cleared then it might lead to some unexpected behavior in other tests.

Jest has different methods that accomplish this but we can focus on two main ones: reset and restore.

#### Reset

`jest.resetAllMocks` is used to reset the state of all currently mocked functions. This means that mock data like `mock.calls` and others will reset back to 0, for example. The implementation of the mocked function will also be reset back to a simple `jest.fn()`.

Running this between each test case is a good way to clean up mocks and it's data:

```
const getObjectSpy = jest.spyOn(objectClient, "getById");

expect(getObjectSpy).toHaveBeenCalled();

afterEach(() => {
  jest.resetAllMocks()
})
```

#### Restore

`jest.restoreAllMocks` is used to restore the original implementation of a mocked function or property. This only applies to mocks that were created using `jest.spyOn()` and `jest.replaceProperty()`, other mocks need to be restored manually by calling .restore() on them.

Running this after a test suite has concluded is a good way to restore the original functionality:

```
const axiosPostSpy = jest.spyOn(axios, "post");

axiosPostSpy.mockResolvedValueOnce({
  status: 200,
  data: {
  ...
  },
});

afterAll(() => {
  jest.restoreAllMocks()
})
```

## External Resources
- [The Testing Trophy and Testing Classifications](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications)
- [An Expert’s Guide to Understanding the Testing Pyramid](https://thectoclub.com/topics/testing-pyramid/)
- [On the Diverse And Fantastical Shapes of Testing](https://martinfowler.com/articles/2021-test-shapes.html)
- [Jest Mocking Best Practices](https://devblogs.microsoft.com/ise/jest-mocking-best-practices/)
- [Telling a Story with Test Code](https://markus.oberlehner.net/blog/telling-a-story-with-test-code)