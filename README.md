<h2>swift lambda
  <img src="http://zeezide.com/img/lambda/swift-lambda-256x256.png"
       align="right" width="128" height="128" />
</h2>

Build and deploy Swift Package Manager projects on AWS Lambda.

`swift lambda` builds Swift projects using the
[Swift AWS Lambda Runtime](https://github.com/swift-server/swift-aws-lambda-runtime)
for deployment into
[AWS Lambda](https://aws.amazon.com/lambda/)
functions. 
It uses a Swift
[cross compilation toolchain](https://github.com/SPMDestinations/homebrew-tap)
for
[Amazon Linux](https://aws.amazon.com/amazon-linux-2/) 
to build the project right on macOS, no Docker required.
Simply call `swift lambda build`.

A built Swift lambda can then be deployed using `swift lambda deploy`
(using either `aws lambda publish` or `sam deploy`).

Note: Due to a [bug](https://bugs.swift.org/browse/SR-13312) 
      in Xcode 11's Swift 5.2.4, a Swift 5.3 install (e.g. Xcode beta 12) is 
      currently required.


## Installation

First make sure `swift --version` shows a 5.3 release, it is currently required.

`swift lambda` is easiest to install using [Homebrew](https://brew.sh),
get it [over here](https://brew.sh).

This single call installs `swift lambda` and all its dependencies:
```bash
$ brew install SPMDestinations/tap/swift-lambda
```

> It's a pretty big download at over 1GB (binary host & target Swift toolchains
> from [Swift.org](https://swift.org/download/#releases) and the
> [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)).


## Usage

A simple Swift Lambda using
[Macro](https://github.com/Macro-swift/MacroLambda).

Setup the Swift package:
```swift
mkdir HelloWorld && cd HelloWorld
swift package init --type executable
open Package.swift # opens Xcode
```

Configure the `Package.swift` to look like this:
```swift
// swift-tools-version:5.2
import PackageDescription

let package = Package(
  name         : "HelloWorld",
  platforms    : [ .macOS(.v10_13) ], // <== add this
  dependencies : [ // and add this dependency ↓
    .package(url: "https://github.com/Macro-swift/MacroLambda.git",
             from: "0.1.3"),
    .package(url: "https://github.com/AlwaysRightInstitute/cows.git",
             from: "1.0.0") // optional but fun
  ],
  targets: [
    .target(name: "HelloWorld", 
            dependencies: [ "MacroLambda", "cows" ])
  ]
)
```

Fill in the Lambda's code in the `main.swift`:

```swift
import MacroLambda

let app = Express()
app.use(bodyParser.text())

app.post("/hello") { req, res, next in
  res.send("Client sent: \(req.body.text ?? "~nothing~")")
}

app.get { req, res, next in
  res.send("Welcome to Macro!\n")
}

if process.isRunningInLambda {
  Lambda.run(app)
}
else {
  app.listen(1337) {
    console.log("server running on http://localhost:1337/")
  }
}
```

Build the package using `swift lambda build -d 5.2` and deploy it to AWS by
calling `swift lambda deploy -d 5.2`. 
It expects a Lambda configuration called `HelloWorld` (select a different
function using the `-f` argument).

> The `-d 5.2` is necessary until AmazonLinux 5.3 development toolchains are
> available.

A more complex example:
[express-simple-lambda](https://github.com/Macro-swift/Examples/tree/feature/lambda-express-1/Sources/express-simple-lambda),
it looks like this:
<img src="https://zeezide.de/img/macro/MacroExpressSimple.png">


## Usage: `swift lambda build`

```
$ swift lambda build -h
Unknown argument: -h

usage: swift lambda build [mode] [options]

Modes:
  -c, --configuration debug|release [default: debug]
  --clean build|dist                [default: build]

Product:
  -p, --product &lt;product&gt;           [default: directory name]
  -d, --destination &lt;dest&gt;

Options:
  -v, --verbose
  -s, --silent
  --static
  --static-libs &lt;libs&gt;

```

## Usage: `swift lambda deploy`

```
$ swift lambda deploy -h
usage: swift lambda deploy [options]

  -f, --function <name>
  -p, --product  <product>
  -t, --template <SAM template>     (optional)
  --stack-name   <SAM stackname>    (optional)

If no function/product name is provided, the current directory
will be used.

Build Options:
  --skip-build                      (do not invoke swift lambda build)
  -c, --configuration debug|release [default: debug]
  -d, --destination &lt;dest&gt;
  --static
  --static-libs &lt;libs&gt;

Options:
  -v, --verbose
  -s, --silent

```

### Links

- [SPMDestinations](https://github.com/SPMDestinations/homebrew-tap)
- [MacroLambda](https://github.com/Macro-swift/MacroLambda)
- Amazon:
  - [AWS Lambda](https://aws.amazon.com/lambda/) homepage
  - Amazon Web Services [API Gateway](https://aws.amazon.com/api-gateway/)
- Apple:
  - WWDC 2020: [Use Swift on AWS Lambda with Xcode](https://developer.apple.com/videos/play/wwdc2020/10644/)
  - Swift Blog: [Introducing Swift AWS Lambda Runtime](https://swift.org/blog/aws-lambda-runtime/)
  - [Swift AWS Lambda Runtime](https://github.com/swift-server/swift-aws-lambda-runtime)
  - [SwiftNIO](https://github.com/apple/swift-nio)
- Tutorials by Fabian Fett:
  - [Getting started with Swift on AWS Lambda](https://fabianfett.de/getting-started-with-swift-aws-lambda-runtime)
  - [Create your first HTTP endpoint with Swift on AWS Lambda](https://fabianfett.de/swift-on-aws-lambda-creating-your-first-http-endpoint)
- Optical Aberration: Messing with Swift AWS Lambda Runtime
  - [Part 1](https://opticalaberration.com/2020/06/investigating-swift-aws-lambda-runtime.html)
  - [Part 2](https://opticalaberration.com/2020/06/investigating-swift-aws-lambda-runtime-part2.html)

### Who

**swift-lambda** is brought to you by
[ZeeZide](http://zeezide.de).
We like feedback, GitHub stars, cool contract work,
presumably any form of praise you can think of.
