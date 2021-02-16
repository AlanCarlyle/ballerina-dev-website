---
layout: ballerina-blank-page
title: Release Note
---
### Overview of Ballerina Swan Lake Alpha2

This Alpha2 release includes the language features planned for the Ballerina Swan Lake release. Moreover, this release includes improvements and bug fixes to the compiler, runtime, standard library, and developer tooling. This release note lists only the features and updates added after the Alpha1 release of Ballerina Swan Lake.

- [Updating Ballerina](#updating-ballerina)
    - [For Existing Users](#for-existing-users)
    - [For New Users](#for-new-users)
- [Highlights](#highlights)
- [What is new in Ballerina Swan Lake Alpha2](#what-is-new-in-ballerina-swan-lake-alpha2)
    - [Language](#language)
        -  [Support for Mapping and Error Binding Patterns in the Match Statement](#support-for-mapping-and-error-binding-patterns-in-the-match-statement)
    - [Runtime](#runtime)
        - [Support for Configurable Variables of Record and Table Types](#support-for-configurable-variables-of-record-and-table-types)
        - [Support for Decrypting String Values Using the New Config Lang Library](#support-for-decrypting-string-values-using-the-new-config-lang-library)
    - [Standard Library](#standard-library)
        - [HTTP Module Improvements](#http-module-improvements)
        - [MIME Module Improvements](#mime-module-improvements)
        - [WebSocket Module Improvements](#websocket-module-improvements)
        - [GraphQL Module Improvements](#graphql-module-improvements)
        - [WebSub Module Improvements](#websub-module-improvements)
        - [WebSubHub Module Improvements](#websubhub-module-improvements)
        - [IO Module Improvements](#io-module-improvements)
        - [Email Module Improvements](#email-module-improvements)
        - [TCP Module Improvements](#tcp-module-improvements)
        - [UDP Module Improvements](#udp-module-improvements)
        - [Crypto Module Improvements](#crypto-module-improvements)
        - [JWT Module Improvements](#jwt-module-improvements)
    - [Code to Cloud](#code-to-cloud)
    - [Developer Tools](#developer-tools)
        - [Language Server](#language-server)
        - [Debugger](#debugger)
        - [Ballerina Shell REPL - EXPERIMENTAL](#ballerina-shell-repl-experimental)
    - [Breaking Changes](#breaking-changes)

### Updating Ballerina

You can use the [update tool](/learn/keeping-ballerina-up-to-date/) to update to Ballerina Swan Lake Alpha2 as follows.

#### For Existing Users

If you are already using Ballerina, you can directly update your distribution to the Swan Lake channel using the [Ballerina Update Tool](/swan-lake/learn/keeping-ballerina-up-to-date/). To do this, first, execute the command below to get the update tool updated to its latest version. 

> `ballerina update`

From Swan Lake Alpha1 onwards, the `ballerina` command has to be issued as `bal`. Next, execute the command below to update to Swan Lake Alpha2.

> `bal dist pull slalpha2`

However, if you are using a Ballerina version below 1.1.0, install via the [installers](/downloads/#swanlake).

#### For New Users

If you have not installed Ballerina, then download the [installers](/downloads/#swanlake) to install.

### Highlights

- Support for mapping and error binding patterns in the `match` statement
- Support for configurable variables of `record` and `table` types
- Support for decrypting string values using the new `config` lang library
- Improvements to the HTTP, mime, WebSocket, GraphQL, WebSub, WebSubHub, IO, email, TCP, UDP, crypto, and JWT standard library modules
- The extension of the Ballerina package distribution file has been changed from `.balo` to `.bala`
- Improvements to developer tools such as the language server and debugger.

### What is new in Ballerina Swan Lake Alpha2

#### Language

##### Support for Mapping and Error Binding Patterns in the Match Statement

**Mapping Binding Pattern**

The `match` statement now supports mapping and error binding patterns with `var`.

```ballerina
match v {
    var {a, b} => {
        // Matches mappings that contain at least fields `a` and `b`.
        // The values of this fields can be accessed via the variables 
        // `a` and `b` within this block.
        io:println(a);
    }
    var {c: {x: a1, y: a2}, ...d} => {
        // Matches mappings that have a field `c` where its value is 
        // another mapping that contains at least the fields `x` and `y`.
        // All of the remaining fields (if any) can be accessed via
        // the new variable `d`.
        int length = d.length();
    }
}
```

**Error Binding Pattern**

```ballerina
match v {
    var error(message, error(causeMessage)) => {
        // Matches errors that have a cause. 
        // The messages of the matched error and the cause error
        // can be accessed via the variables `message` and 
        // `causeMessage` within this block.
        io:println(causeMessage);
    }
    var error(a, code = matchedCode) => {
        // Matches errors that have a detail entry with the key `code`.
        // The `code` can be accessed using the `matchedCode` variable 
        // within this block.
        io:println(matchedCode);
    }
}
```

#### Runtime

##### Support for Configurable Variables of Record and Table Types

**Record Type**

Record fields with simple types like `int`, `string`, `boolean`, `float`, `decimal`, and arrays of the respective types are now supported. 

For example, if the `Config.toml` file contains the following TOML table,

```toml
[Pkg.testUserOne]
username = "john"
password = "abcd"
scopes = ["write"]

[Pkg.testUserTwo]
Username = “mary”
Password = “test123”
```

it can be loaded as a configurable variable of a record type as follows.

```ballerina
type AuthInfo record {
  readonly string username;
  string password;
  string[] scopes?;
};

configurable AuthInfo & readonly testUserOne = ?;
configurable AuthInfo & readonly testUserTwo = ?;
```

**Table Type**

Ballerina now supports configurable variables of type `table` through TOML arrays of tables.

For example, if the `Config.toml` file contains the following TOML table array,

```toml
[[Pkg.users]]
username = "alice"
password="password1"
scopes=["scope1"]

[[Pkg.users]]
username = "bob"
password="password2"
scopes=["scope1", "scope2"]

[[Pkg.users]]
username = "jack"
password="password3"
```

it can be loaded as a configurable variable of a `table` type as follows.

```ballerina
configurable table<AuthInfo> key(username) & readonly users = ?;
```

##### Support for Decrypting String Values Using the New Config Lang Library

The `bal encrypt` command can be used to encrypt the plain text values and specify those in the `Config.toml` file. Then, the `config:decryptString(<string variable>)` can be used to decrypt the configurable value. 

For example, if the `Config.toml` file contains the following encrypted string value,

```toml
password = "@encrypted:{ODYUoKSw0xW31eoxa/s2ESdBNgk1gX77txBIgpLC6NQ=}"
```

it will be decrypted in the Ballerina code as follows.

```ballerina
import ballerina/lang.config;

configurable string password = ?;
public function main() {
    string decryptedPassword = config:decryptString(password);
}
```

#### Standard Library

##### HTTP Module Improvements

###### Introduced Byte Stream Manipulation Functions to the Request and Response 

This introduction enables manipulating the payload as a stream of `byte[]`. The `setByteStream()` and `getByteStream()` methods use the Ballerina stream feature.

```ballerina
http:Request request = new;
io:ReadableByteChannel byteChannel = check io:openReadableFile
                                ("path/to/file.tmp");
stream<io:Block, io:Error> byteStream = check byteChannel.blockStream(8196);
request.setByteStream(byteStream);

http:Response response = new;
stream<byte[], io:Error>|error str = response.getByteStream();
```

###### Introduced a Header Parameter to the Resource Signature

With the introduction of the `@http:Header` annotation, inbound request headers can be retrieved by binding them to a resource signature parameter. Individual headers can be accessed as `string` or `string[]` types while a parameter of type `http:Headers` can be used to access all headers. 

```ballerina
service on helloEP {
    resource function get hello(@http:Header {name:”Accept”} string? acceptHeader, 
            http:Headers allHeaders) {
        //...
    }
}
```

##### MIME Module Improvements

###### Introduced Byte Stream Manipulation Functions to the `mime:Entity` Class

This introduction enables manipulating the entity body as a stream of `byte[]`.

```ballerina
function setByteStream(stream<byte[], io:Error> byteStream,
        string contentType = "application/octet-stream") {
}

function getByteStream(int arraySize = 8196) returns 
        stream<byte[],  io:Error>|mime:ParserError {
}

function getBodyPartsAsStream(int arraySize = 8196) returns 
        stream<byte[], io:Error>|mime:ParserError {

// Sample
byte[][] content = ["File Content".toBytes()];
stream<byte[], io:Error> byteStream = content.toStream();
mime:Entity entity = new;
entity.setByteStream(byteStream);
stream<byte[], io:Error>|mime:ParserError str = entity.getByteStream();
```

#### WebSocket Module Improvements

Introduced the Sync client. This is the primary client of the WebSocket module. This client is capable of reading and writing messages synchronously.

**Reading and Writing Text Messages**

```ballerina
websocket:Client wsClient = check new("ws://echo.websocket.org");
var err = wsClient->writeTextMessage("Text message");
string textResp = check wsClient->readTextMessage();
```

**Reading and Writing Binary Messages**

```ballerina
websocket:Client wsClient = check new("ws://echo.websocket.org");
var err = wsClient->writeBinaryMessage("Binary message".toBytes());
byte[] byteResp = check wsClient->readBinaryMessage();
```

###### GraphQL Module Improvements

The Ballerina GraphQL listeners can now be secured using auth configurations. This configuration is the same as the listener configurations in the `http:Listener`. Additionally, a GraphQL service can be secured by defining a `maxQueryDepth` as an annotation to restrict the depth of a query before execution.

```ballerina
import ballerina/graphql;

graphql:ListenerConfiguration configs = {
	// http listener configurations
};
listener graphql:Listener graphqlListener = new(9090, configs);

@graphql:ServiceConfigurration {
    maxQueryDepth: 3
}
Service /graphql on graphqlListener {
    // Service definition
}
```

##### WebSub Module Improvements

Included functionality to the `websub:SubscriberService` to respond with user-defined custom payloads/header parameters in error scenarios.

```ballerina

import ballerina/websub;

listener websub:Listener subscriberListener = new (9001);

service /subscriber on subscriberListener {
    remote function onSubscriptionValidationDenied(websub:SubscriptionDeniedError msg) 
                      returns websub:Acknowledgement? {
        websub:Acknowledgement ack = {
                  headers = { "Content-Encoding" :  "gzip" },
                  body = { "message" :  "Successfully processed request" }
        };
        return ack;
    }

    remote function onSubscriptionVerification(websub:SubscriptionVerification msg)
                        returns websub:SubscriptionVerificationSuccess|websub:SubscriptionVerificationError {
        if (msg.hubTopic == "https://www.sample.topic") {
            return error websub:SubscriptionVerificationError(
                                 "Hub topic not supported", 
                                  headers = { "Content-Encoding" :  "gzip" }, 
                                  body = { "message" :  "Hub topic not supported" });
        } else {
            return {};
        }
      }

    remote function onEventNotification(websub:ContentDistributionMessage event) 
                        returns websub:Acknowledgement|websub:SubscriptionDeletedError? {
        return error websub:SubscriptionDeletedError(
                             "Subscriber wants to unsubscribe",
                             headers = { "Content-Encoding" :  "gzip" }, 
                             body = {"message": "Unsubscribing from the topic"});
    }
}

```

##### WebSubHub Module Improvements

Included functionality to the `websubhub:Service` to respond with user-defined custom payloads/header parameters in error scenarios.

```ballerina

Import ballerina/websubhub;

service /websubhub on websubhub:Listener(9091) {

    remote function onRegisterTopic(websubhub:TopicRegistration message)
                                returns websubhub:TopicRegistrationSuccess|websubhub:TopicRegistrationError {
        if (message.topic == "https://sub.topic.com") {
            websubhub:TopicRegistrationSuccess successResult = {
                body: <map<string>>{
                       isSuccess: "true"
                    }
            };
            return successResult;
        } else {
           return error websubhub:TopicRegistrationError("Topic registration failed!",
                        headers = { "Content-Encoding" :  "gzip" },
                        body = { "hub.additional.details": "Feature is not supported in the hub"});
        }
    }

}
```

##### IO Module Improvements

Introduce a parameter of type `XmlWriteOptions` to specify the entity type and the document type declaration.

```ballerina
public function fileWriteXml(@untainted string path, xml content, *XmlWriteOptions xmlOptions, FileWriteOption fileWriteOption = OVERWRITE) returns Error? {}
```

##### Email Module Improvements

- Make the `body` field of the `email:Message` record optional. This enables sending an email only with the `htmlBody` without a text-typed body field.

- Add `mime:Entity` type to the union type of the `attachments` field in the `email:Message` record. That enables attaching a single MIME Entity as an attachment. This `email:Message` record change would appear as follows.

```ballerina
public type Message record {|
    // … other fields
    mime:Entity|Attachment|(mime:Entity|Attachment)[] attachments?;
|};
```

- The above-mentioned `body` and `attachments` fields related changes are updated in the `email:Options` record as given below in order to facilitate the `sendEmail` method in the `email:SmtpClient`.

```ballerina
public type Options record {|
    // … other fields
   string body?;
   mime:Entity|Attachment|(mime:Entity|Attachment)[] attachments?;
|};
```

- Remove the `properties` field representing custom properties from all Email module related configuration records: `email:SmtpConfig`, `email:PopConfig`, `email:ImapConfig`, `email:PopListenerConfig`, and `email:ImapListenerConfig`.

- The `mail.smtp.ssl.checkserveridentity` custom property was passed as an entry in `properties` to enable/disable server certificate’s hostname verification. As the `properties`field is removed from the API, from this release onwards, the `verifyHostName` boolean field is introduced to the `secureSocket` record for all the configurations related to the Email module. 

    This `email:SecureSocket` record change would appear as follows.

    ```ballerina
    public type SecureSocket record {|
        // … other fields
    boolean verifyHostname = true;
    |};
    ```

##### TCP Module Improvements

Introduced SSL/TLS-based communication to the TCP module using the `secureSocket` record configuration in both the client and the listener.

###### TCP Client with `secureSocket`

A sample code is as follows.

```ballerina

import ballerina/tcp;

configurable string keyPath = ?;
configurable string certPath = ?;

public function main() returns error? {
    tcp:Client socketClient = check new ("localhost", 9002, secureSocket = {
        certificate: {path: certPath},
        protocol: {
            name: "TLS",
            versions: ["TLSv1.2", "TLSv1.1"]
        },
        ciphers: ["TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"]
    });

    string msg = "Hello Ballerina Echo from secure client";
    byte[] msgByteArray = msg.toBytes();
    check socketClient->writeBytes(msgByteArray);

    readonly & byte[] receivedData = check socketClient->readBytes();
    test:assertEquals('string:fromBytes(receivedData), msg, "Found unexpected output");

    check socketClient->close();
}
```

###### TCP Listener with `secureSocket`

A sample code is as follows.

```ballerina

configurable string keyPath = ?;
configurable string certPath = ?;

service on new tcp:Listener(9002, secureSocket = {
    certificate: {path: certPath},
    privateKey: {path: keyPath},
    protocol: {
        name: "TLS",
        versions: ["TLSv1.2", "TLSv1.1"]
    },
    ciphers: ["TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"]
}) {

    isolated remote function onConnect(tcp:Caller caller) returns tcp:ConnectionService {
        io:println("Client connected to secureEchoServer: ", caller.remotePort);
        return new EchoService(caller);
    }
}

service class EchoService {
    tcp:Caller caller;

    public isolated function init(Caller c) {
        self.caller = c;
    }

    remote function onBytes(readonly & byte[] data) returns (readonly & byte[])|tcp:Error? {
        io:println("Echo: ", 'string:fromBytes(data));
        return data;
    }
}
```

##### UDP Module Improvements

- The `sendDatagram` function will now send multiple datagrams if the size of the `byte[]` value provided as the `data` field of the datagram exceeds 8Kb.

- Returning `Datagram` from the `onDatagram` or `onBytes` remote methods also sends multiple datagrams if the size of the `byte[]` value provided as the `data` field of the datagram  exceeds 8Kb.

###### Crypto Module Improvements

Added support to decode private keys from `.key` files and public keys from `.cert` files and update the APIs for decoding private/public keys. This enables reading private/public keys from PEM files.

###### JWT Module Improvements

Extended the private key support for JWT signature generation and public cert support for JWT signature validation.

#### Code to Cloud

- The `Kubernetes.toml` file is renamed to `Cloud.toml`. 
- The `--cloud=docker` build option is implemented. This will build the Dockerfile and Docker image without generating the Kubernetes artifacts.


#### Developer Tools

##### Language Server

Implemented renaming any symbol in the Language Server.

##### Debugger

Added variable paging support. With this feature, the Ballerina variables, which contain a large number of child variables will be shown in a paged view in the debug variable presentation.

##### Ballerina Shell REPL [EXPERIMENTAL]

1. Fixed the REPL expression output to output the `toBalString()` result.
2. Improved the REPL parser to support some partial snippets such as the example cases below.

- Template strings will allow continuing on new lines.
- The CLI will wait for more input if the last character was an operator.
- The CLI will not wait for unclosed double quotes.

3. Enable REPL to exit on `Ctrl+D`.

#### Breaking Changes

1. Member access on a value of type `table`now returns `()` if the `table` does not contain a member with the specified key. Otherwise, the result is the member of the `table` with the given key.

```ballerina
type Employee record {
    readonly string name;
    int id;
};

public function main() {
    table<Employee> key(name) employeeTable = table [
        {name: "Mike", id: 1234},
        {name: "John", id: 4567}
    ];

    Employee? emp1 = employeeTable["John"]; 
    io:println(emp1); //{name: "John", id: 4567}
    Employee? emp2 = employeeTable["Kate"]; 
    boolean value2 = emp2 is (); //true
}
```

2. Iterating over `xml` in a `from` clause in query expressions now returns `xml` and iterating over `xml<T>` returns `T` .

```ballerina
xml authorList = xml `<authorList>
                                      <author>
                                          <name>Sir Arthur Conan Doyle</name>
                                          <country>UK</country>
                                      </author>
                                     <author>
                                        <name>Dan Brown</name>
                                        <country>US</country>
                                     </author>
                                 </authorList>`;

xml authors = from xml y in authorList/<author>/<name>
select y;
io:println(authors); //<name>Sir Arthur Conan Doyle</name><name>Dan Brown</name>
```

3. The `readonly` and `value:Cloneable` values cannot be assigned to `any` since they contain an `error`.