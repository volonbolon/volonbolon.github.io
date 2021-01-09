---
layout: post
title: "Testing with mocks"
date: 2018-01-16 13:43:31 GMT
tags: swift ui test testing
---

I love Objective-C. Swift is also wonderful, but Objective-C is spectacular. 

One thing I miss from Objective-C is dynamic patching. It was wonderful for mocking behaviors in testing. [OCMock](http://ocmock.org/) was awesome. And when I’ve found [OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs) I couldn’t stop myself to add it to all my projects. 

Swift is far more protective. Ok, it’s true since Swift app runs inside the Objective-C runtime, is possible to manipulate the dispatching of messages at runtime. And I think that’s for the better. Let’s remove OCMock and OHHTTPStubs from the cart file and let’s think how we can mock using Swift. Because mock is absolutely necessary. 

## Mocking the network
Perhaps, the canonical example for mocking is networking. To test against the Net is hard, expensive, and it could slow tests a lot. Let’s revisited, for instance, `func fetchCurrentWeatherData(input:completionHandler:)` from the [weather](https://github.com/volonbolon/refactored-journey/blob/f71336e33336d5e18bbb5819107906a64865b5da/weather/Helpers/Networking/OpenWeatherMapNetworkController.swift#L18) app. 

Sure, we could call it from the test suite, and we can inject an URL to make it fail, and then, a point the task to an URL that makes it pass. 

```
let task = session.dataTask(with: request) { (data: Data?, _: URLResponse?, error: Error?) in

            guard error == nil else {

                let networkError = NetworkControllerError.forwarded(error!)

                let payload = Either<NetworkControllerError, WeatherData>.left(networkError)

                completionHandler(payload)

                return

            }




            guard let jsonData = data else {

                let payloadError = NetworkControllerError.invalidPayload(url)

                let payload = Either<NetworkControllerError, WeatherData>.left(payloadError)

                completionHandler(payload)

                return

            }




            self.decode(jsonData: jsonData,

                        endpointURL: url,

                        temperatureUnit: input.unit,

                        completionHandler: { (result: Either<NetworkControllerError, WeatherData>) in

                            completionHandler(result)

            })

        }

        task.resume()

``` 

Obviously, it will take a lot of time to make the trips to the server, we would be loading its load, and besides, there is a principle at stake. Aren’t we supposed to be doing Unit Test? Well, if that’s the case, by doing something like 

```
    func testOpenWeatherMap() {
        let exp = expectation(description: "Get weather data")

        let controller = OpenWeatherMapNetworkController()
        let input = Input(location: "Campana", unit: TemperatureUnit.metric)
        controller.fetchCurrentWeatherData(input: input) { (result: Either<NetworkControllerError, WeatherData>) in
            switch result {
            case .left:
                XCTFail("no data returned by fetchWeatherData()")
            case .right(let data):
                let city = input.location
                print("Weather in \(city): \(data.condition), \(data.temperature)\(data.unit)")
                exp.fulfill()
            }
        }
        waitForExpectations(timeout: 10, handler: nil)
    }
```

We would be testing two things, the network, and the function itself. Not one, but two. The word “Unit” is no longer applicable. 

We need to abstract the actual network interactions. At the very basic, we need to inject our own closure for `func dataTask(with request:completionHandler:) -> URLSessionDataTask`. Obviously, we can subclass `URLSession` to introduce our own version of the method. But, we also need to mock `URLSessionDataTask`, to hijack the whole interaction. Let’s mock `URLSessionDataTask` first. 

```
class MockURLSessionDataTask: URLSessionDataTask {
    // I know, against Apple naming convention,
    // but I rather have the `Mock` part right on front
    // to prevent autocompletion to introduce errors.
    private let closure: () -> Void

    init(closure: @escaping () -> Void) {
        self.closure = closure
    }

    override func resume() {
        self.closure()
    }
}
```

Pretty self-explanatory. We save the closure we want to pass when the task is done, and then, we just call it from our overrode `resume`

Now, let’s mock the session. 

```
class MockURLSession: URLSession {
    typealias CompletionHandler = (Data?, URLResponse?, Error?) -> Void

    var data: Data?
    var error: Error?
    var response: URLResponse?

    override func dataTask(with request: URLRequest,
                           completionHandler: @escaping CompletionHandler) -> URLSessionDataTask {
        let data = self.data
        let response = self.response
        let error = self.error

        let dataTask = MockURLSessionDataTask(closure: {
            completionHandler(data, response, error)
        })

        return dataTask
    }
}
```

Again, we have ivars for `data`, `error` and `response`. We can set them as we wish to exercise different portions of our code. Do we want to make sure the `guard` is kicked on when an error is there? Just create a session, set the error, and test. 

```
func testOpenWeatherShouldFailWithError() {
        let exp = expectation(description: "Get weather data")

        let session = MockURLSession()
        let error = NSError(domain: "asd", code: 123, userInfo: nil)
        session.error = error as Error

        self.prepareRetreiveWeatherSessionDataSucess(session)

        let controller = OpenWeatherMapNetworkController(session: session)
        let input = Input(location: "Campana", unit: TemperatureUnit.metric)
        controller.fetchCurrentWeatherData(input: input) { (result: Either<NetworkControllerError, WeatherData>) in
            switch result {
            case .left(let error):
                XCTAssertNotNil(error)
                exp.fulfill()
            case .right:
                XCTFail("It should fail")
            }
        }
        waitForExpectations(timeout: 10, handler: nil)
    }
``` 

And obviously, you can also test for data to be correctly parsed, and because you are injecting the data yourself, you can check for specific values as well. 

```
func testOpenWeatherMap() {
        let exp = expectation(description: "Get weather data")

        let session = MockURLSession()

        self.prepareRetreiveWeatherSessionDataSucess(session)

        let controller = OpenWeatherMapNetworkController(session: session)
        let input = Input(location: "Campana", unit: TemperatureUnit.metric)
        controller.fetchCurrentWeatherData(input: input) { (result: Either<NetworkControllerError, WeatherData>) in
            switch result {
            case .left:
                XCTFail("no data returned by fetchWeatherData()")
            case .right(let data):
                let city = input.location
                let condition = data.condition
                let temperature = data.temperature

                XCTAssertEqual(city, "Campana")
                XCTAssertEqual(condition, "Clear")
                XCTAssertEqual(temperature, 42.8)

                exp.fulfill()
            }
        }
        waitForExpectations(timeout: 10, handler: nil)
    }
```

A complete example can be found in [Gist](https://gist.github.com/volonbolon/45683f472e45001f83e6b4b9fad01176)