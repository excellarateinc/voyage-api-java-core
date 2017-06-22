## Development Recipes
Instructional recipies for how to do something within the codebase. 

> __Keep Organized__ Keep the Table of Contents alphabetized and do your best to extend this document in a way that will be easy to read/scroll for all developers.

## Table of Contents
* [General](#general)
  - [Folder by Feature](#folder-by-feature)
  - [Run Tests](#run-tests)
* [Build & Deploy](#build-deploy)
  - [Clean workspace and rebuild](#clean-workspace-and-rebuild)

## General

### Folder by Feature

#### File and Folder Structure

Voyage uses a folder-by-feature approach, where code is organized into package folders by feature rather than by class type. Packaging classes by feature provides a clear view of the "components" required to support the feature. Utilizing this organization method also allows for easier modularization when a feature is desirable in other apps. 

#### Example
/com/company/app
   /search
      SearchController
      SearchService
      SearchRepository
      SearchCriteria
   /security
      UserController
      UserService
      UserRepository
      User

#### Notes
* If you are adding a new feature, create a new folder. 
* If you are modifying or adding to an existing feature, then work within that feature's folder. 
* Be mindful of creating dependencies on other features. Too many depedencies on other feature may be a 'bad code smell' for your feature. 
* Avoid putting features or utilities in a 'common' package if possible. ONLY put features into common if they are TRULY generic and non-feature specific. Otherwise, just make a dependency on the feature you need. 
* Keep your folder structure as flat as possible, only build sub folders for sub features if the file count grows larger than ~7.

:arrow_up: [Back to Top](#table-of-contents)


### Run Tests

#### Command Line
1. cd /path/to/my/workspace/voyage-api-java-core
2. gradle test

#### IntelliJ
1. Within the Project sidebar pane, right-click on the `/src/test` folder  
   ![Test in IntelliJ](./images/RECIPES_RunTests1.png)

2. Choose the 'Run All Tests' option to run all unit and integrations tests found
   ![Test in IntelliJ 2](./images/RECIPES_RunTests2.png)

3. Run a specific test by right-clicking on the Test class and selecting 'Run >ClassNameSpec<'
   ![Test in IntelliJ 3](./images/RECIPES_RunTests3.png)

4. Alternatively, open the Gradle tool menu in IntelliJ and double click the 'verification > test' option
   ![Test in IntelliJ 4](./images/RECIPES_RunTests4.png)


### Error Handling w/ i18n support
#### Overview
Error handling is a very important topic for a web services API because consumers of the API will need some uniform way to anticipate and process errors. Error handling can also be a very messy pattern within a code base because it's not always known where errors could come from or when they will occur. Fortunuately, Spring Framework has defined a nice error handling pattern in conjunction with Spring MVC where unchecked exceptions are intercepted and handed off to a error handler for futher processing. This API has extended the Spring MVC error handler within `/src/main/groovy/voyage/common/error/DefaultExceptionHandler` that intercepts Voyage `/src/main/groovy/voyage/common/error/AppException` unchecked exceptions and translates the contents into a well formed HTTP JSON response. 

#### AppException 
`/src/main/groovy/voyage/common/error/AppException` extends RuntimeException and defines an HTTP Status code and a message with the option to override the default generated error code. When AppException is thrown from anywhere in the app, SpringMVC will be watching for the exception and processing it using `/src/main/groovy/voyage/common/error/DefaultExceptionHandler`. Without going into much detail, the DefaultExceptionHandler will transform the AppException into a well formed JSON response and returned to the consumer. 

When creating your own error exceptions, extend AppException with your own properly named implemenation. 
```
class UnknownIdentifierException extends AppException {
    private static final HttpStatus HTTP_STATUS = HttpStatus.NOT_FOUND
    private static final String DEFAULT_MESSAGE = 'Unknown record identifier provided'

    UnknownIdentifierException() {
        this(DEFAULT_MESSAGE)
    }

    UnknownIdentifierException(String message) {
        super(HTTP_STATUS, message)
    }

    @Override
    String getErrorCode() {
        return HTTP_STATUS.value() + '_unknown_identifier'
    }
}
```
* Extending AppException provides an opportunitity to name your class with the specific error that it represents
* Embedding the default HTTP Status code and API response message keeps these details out of the code layers that shouldn't know about HTTP or the consumer. 
* getErrorCode() is overridden to provide a specific error code for consumers to use to translate the code into a language appropriate message to the end-user. 

Example of how an AppException can be used within the code:
```
@Service
class UserService {
    private final UserRepository userRepository

    @Autowired
    UserService(UserRepository userRepository) {
        this.userRepository = userRepository
    }
    
    User get(@NotNull Long id) {
        User user = userRepository.findOne(id)
        if (!user) {
            throw new UnknownIdentifierException()
        }
        return user
    }
}
```

#### Internationalization (i18n)
This API does not technically support internationalization and likely never will. The consumer of the API is responsible with interfacing with the end-user and therefore is in complete control of the language presentation. To support consumers with the ability to properly display error messages in a user appropriate language, the error handling framework in this API will always return a unique error code that can be used for langauge translation lookups. See [AngularJS i18n docs](https://docs.angularjs.org/guide/i18n) for more information on how a consumer would use a message code to look up a language translation.  

Error responses from web services of this API are described within the [Web Services Standards](./STANDARDS-WEB-SERVICES.md#response-errors) documentation. Please be sure to read through the Web Services Standards specification for a more thorough understanding of error handling for web services. 

When AppException is translated into a JSON response, the output conform to the following JSON pattern:
```
HTTP Status: 40x
[
   {error: "unique.code.here", errorDescription: "human readable description"},
   {error: "unique.code.here", errorDescription: "human readable description"},
]
```

A concrete example of the UnkownIdentifierException would be:
```
HTTP Status: 404 Not Found
[
   {error: "404_unknown_identifier", errorDescription: "Unknown record identifier provided"}
]
```
* Only one error is returned, but it is still contained within a list so that the consumer doesn't have to detect if there is a list response or an object response. Keeping this consistent makes for easier and consistent coding on the consumer side. 
* 'error' is a custom error code that can be used by the consumer to look up a specific message for display to the end user
* 'errorDescription' is an English message to the consumer that will likely not be used beyond testing. 

By default, the 'error' value in the JSON response will be a generated code based on the HTTP Status + HTTP Status Name. For example:
```
class SomeException extends AppException {
    private static final HttpStatus HTTP_STATUS = HttpStatus.BAD_REQUEST
    private static final String DEFAULT_MESSAGE = 'Some error just occurred'

    SomeException() {
        this(DEFAULT_MESSAGE)
    }

    SomeException(String message) {
        super(HTTP_STATUS, message)
    }
}
```
* The parent AppException.getErrorCode() is not overridden
* The code will be generated based on the HttpStatus.BAD_REQUEST value: 400_bad_request

Example JSON output of SomeException will be:
```
HTTP Status: 404 Not Found
[
   {error: "400_bad_request", errorDescription: "Some error just occurred"}
]
```

In many cases, the '400_bad_request' error code will be too generic for a consumer to translate into something meaningful for the end user. To provide a more meaningful error code, override the getErrorCode() method and create a unique error code.
```
class SomeException extends AppException {
    private static final HttpStatus HTTP_STATUS = HttpStatus.BAD_REQUEST
    private static final String DEFAULT_MESSAGE = 'Some error just occurred'

    SomeException() {
        this(DEFAULT_MESSAGE)
    }

    SomeException(String message) {
        super(HTTP_STATUS, message)
    }
    
    @Override
    String getErrorCode() {
        return HTTP_STATUS.value() + '_some_error'
    }
}
```

Example JSON output of SomeException will now include a unique error code for the consumer to use and translate appropriately. 
```
HTTP Status: 404 Not Found
[
   {error: "400_some_error", errorDescription: "Some error just occurred"}
]
```

#### Packages - Where Should Exceptions Live? 
This API follows a [package by feature](DEVELOPMENT-RECIPES.md#folder-by-feature) organization structure, which means that all files relating to a feature should be bundled together within the feature folder/package. For example, a UsernameAlreadyExistsException that is thrown by the voyage.user.UserService class, should be created within the voyage.user package or a sub-package like voyage.user.error if there are a number of exceptions that are cluttering the main package. 

> NOTE: Do not place Exception classes that are specific to a feature into a _common_ package or some other unrelated package. Follow the [package by feature](DEVELOPMENT-RECIPES.md#folder-by-feature) organization structure strictly and only have a class exist in a _common_ package if it is truly used by multiple features and is general enough to live outside of the feature. 

:arrow_up: [Back to Top](#table-of-contents)



## Build & Deploy

### Clean workspace and rebuild
#### Command line
`voyage-api-java-core > gradle clean build`

#### IntelliJ
1. Rebuild the IntelliJ compiled source files in /out folder by clicking the file menu 'Build' and option 'Rebuild Project'
![IntelliJ Build & Deploy 1](./images/RECIPES_CleanAndBuild1.png)

2. Rebuild by invoking the Gradle 'clean' and 'build' tasks within the Gradle tool window
![IntelliJ Build & Deploy 2](./images/RECIPES_CleanAndBuild2.png)

:arrow_up: [Back to Top](#table-of-contents)

