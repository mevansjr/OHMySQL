# OHMySQL

[![License][license-image]][license-url]
[![License][platform-image]][platform-url]
[![Documentation][docs-image]][docs-url]

<p align="center" >★★ <b>Every star is appreciated!</b> ★★</p>

The library supports Objective-C and Swift, iOS and macOS. You can connect to your remote MySQL database using OHMySQL API. It allows you doing queries in easy and object-oriented way. Common queries such as SELECT, INSERT, DELETE, JOIN are wrapped by Objective-C code and you don't need to dive into MySQL C API.

- [Goal](https://github.com/oleghnidets/OHMySQL/wiki/Goal)
- [Features](#features)
- [Requirements](#requirements)
- [How To Get Started](#how-to-get-started)
- [Installation](#installation)
- [Usage](#usage)
    - [Query Context](#query-context)
    - [SELECT](#select)
    - [INSERT](#insert)
    - [UPDATE](#update)
    - [DELETE](#delete)
    - [JOINs](#joins)
    - [Object Mapping](#object-mapping)
    - [Set up SSL](https://github.com/oleghnidets/OHMySQL/wiki/Set-up-SSL)
- [Communication](#communication)
- [License](#license)

## Goal

If you are interested in and want to know [how it can be applied](https://github.com/oleghnidets/OHMySQL/wiki/Goal) in your project too. 

## Features

- [x] Easy to integrate and use
- [x] Many functionality features
- [x] Requires minimal knowledge in SQL
- [x] Supports iOS and macOS
- [x] Clean code with unit tests
- [x] [Complete documentation](http://oleghnidets.github.io/OHMySQL/) and [support](https://github.com/oleghnidets/OHMySQL/issues?q=is%3Aissue+is%3Aclosed)
## Requirements
- iOS 8.0+ / macOS 10.9+
- Xcode 8.1+

## How To Get Started
- To test locally you can install [MySQL](https://dev.mysql.com/downloads/mysql/) or [MAMP local server](https://www.mamp.info/en/).
- Try to use OHMySQL API ([set up demo project](https://github.com/oleghnidets/OHMySQL/wiki/Set-up-demo-project), [read documentation](http://oleghnidets.github.io/OHMySQL/)). 
- When it'll be ready then transfer your local Data Base(s) to remote MySQL server.

## Installation
You can use CocoaPods. Add the following line to your Podfile:
```ruby
pod 'OHMySQL'
```

If you are using Swift do not forget to add `use_frameworks!` at the top of Podfile. Add platform, example `platform :osx, '10.10'`.

## Usage

At the first you need to connect to the database.

*Objective-C version:*
```objective-c
OHMySQLUser *user = [[OHMySQLUser alloc] initWithUserName:@"root"
					 	 password:@"root"
					       serverName:@"localhost"
						   dbName:@"sample"
						     port:3306
						   socket:@"/Applications/MAMP/tmp/mysql/mysql.sock"];
OHMySQLStoreCoordinator *coordinator = [[OHMySQLStoreCoordinator alloc] initWithUser:user];
[coordinator connect];
```
*Swift version:*
```swift
let user = OHMySQLUser(userName: "root", password: "root", serverName: "localhost", dbName: "ohmysql", port: 3306, socket: "/Applications/MAMP/tmp/mysql/mysql.sock")
let coordinator = OHMySQLStoreCoordinator(user: user!)
coordinator.encoding = .UTF8MB4
coordinator.connect()
```
To end a connection:
```objective-c
[coordinator disconnect];
```
```swift
coordinator.disconnect()
```
## Query Context

To execute a query you have to create the context:
```objective-c
OHMySQLQueryContext *queryContext = [OHMySQLQueryContext new];
queryContext.storeCoordinator = coordinator;
```
```swift
let context = OHMySQLQueryContext()
context.storeCoordinator = coordinator
```

You will use this context to execute queries or manipulate the objects.

### SELECT 

The response contains array of dictionaries (like JSON).

```objective-c
OHMySQLQueryRequest *query = [OHMySQLQueryRequestFactory SELECT:@"tasks" condition:nil];
NSError *error = nil;
NSArray *tasks = [queryContext executeQueryRequestAndFetchResult:query error:&error];
```
```swift
let query = OHMySQLQueryRequestFactory.select("tasks", condition: nil)
let response = try? OHMySQLContainer.shared.mainQueryContext?.executeQueryRequestAndFetchResult(query)
```
You will get a response like this:
```objective-c
[{ @"id": @1, @"name": @"Task name", @"description": @"Task description", @"status": [NSNull null] }]
```

### INSERT

```objective-c
OHMySQLQueryRequest *query = [OHMySQLQueryRequestFactory INSERT:@"tasks" set:@{ @"name": @"Something", @"desctiption": @"new task" }];
NSError error;
[queryContext executeQueryRequest:query error:&error];
```
```swift
let query = OHMySQLQueryRequestFactory.insert("tasks", set: ["name": "Something", "desctiption": "new task"])
try? mainQueryContext?.execute(query)
```

### UPDATE

```objective-c
OHMySQLQueryRequest *query = [OHMySQLQueryRequestFactory UPDATE:@"tasks" set:@{ @"name": @"Something", @"description": @"new task update" } condition:@"id=5"];
NSError error;
[queryContext executeQueryRequest:query error:&error];
```
```swift
let query = OHMySQLQueryRequestFactory.update("tasks", set: ["name": "Something"], condition: "id=7")
try? mainQueryContext?.execute(query)
```

### DELETE

```objective-c
OHMySQLQueryRequest *query = [OHMySQLQueryRequestFactory DELETE:@"tasks" condition:@"id=10"];
NSError error;
[queryContext executeQueryRequest:query error:&error];
```
```swift
let query = OHMySQLQueryRequestFactory.delete("tasks", condition: "id=10")
try? mainQueryContext?.execute(query)
```

### JOINs

The response contains array of dictionaries (like JSON). You can do 4 types of joins (INNER, RIGHT, LEFT, FULL) using string constants.
```objective-c
OHMySQLQueryRequest *query = [OHMySQLQueryRequestFactory JOINType:OHJoinInner													fromTable:@"tasks"
						      columnNames:@[@"id", @"name", @"description"]
							   joinOn:@{ @"subtasks":@"tasks.id=subtasks.parentId" }];
NSArray *results = [queryContext executeQueryRequestAndFetchResult:query error:nil];
```
```swift
let query = OHMySQLQueryRequestFactory.joinType(OHJoinInner, fromTable: "tasks", columnNames: ["id", "name", "description"], joinOn: ["subtasks": "tasks.id=subtasks.parentId"])
let result = try? mainQueryContext?.executeQueryRequestAndFetchResult(query)
```

### Object Mapping

You have to implement the protocol OHMappingProtocol for your models. Insertion looks like the following (in this example the NSManagedObject instance). 
The library has only a primary logic for mapping, so I would recommend you writing a mapping logic by yourself. If you are using Swift you cannot use fundamental number types (Int, Double), only NSNumber (due to run-time). 
```objective-c
[queryContext insertObject:task];
BOOL result = [queryContext save:nil];
```
```swift
mainQueryContext?.insertObject(task)
try? mainQueryContext?.save()
```

You can update/delete the objects easily.
```objective-c
// You don't need to specify primary index here.  It'll be update for you.
OHTask *task = [OHTask new];
task.name = @"Code cleanup";
task.taskDescription = @"Delete unused classes and files";
task.status = 0;
[queryContext updateObject:task];
...
task.name = @"Something";
task.status = 1;
[task update];
...
[queryContext deleteObject:task];
BOOL result = [queryContext save:nil];
```
```swift
let task = Task()
task.name = "sample"
mainQueryContext?.updateObject(task)
mainQueryContext?.deleteObject(task)

try? mainQueryContext?.save()
```

## Communication
- If you found a bug, have suggestions or need help, please, [open an issue](https://github.com/oleghnidets/OHMySQL/issues/new).
- If you want to contribute, [submit a pull request](https://github.com/oleghnidets/OHMySQL/pulls).
- If you need help, write me oleg.oleksan@gmail.com.
- Make me feel happier ;]

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CVFAEEZJ9DJ3L)

## License 

OHMySQL is released under the MIT license. See [LICENSE](LICENSE) for details.

[platform-image]: https://img.shields.io/badge/platform-ios%20%7C%20macOS-lightgrey.svg
[platform-url]: https://github.com/oleghnidets/OHMySQL
[docs-image]: https://github.com/oleghnidets/OHMySQL/blob/master/docs/badge.svg
[docs-url]: http://oleghnidets.github.io/OHMySQL/
[license-image]: https://img.shields.io/badge/License-MIT-blue.svg
[license-url]: LICENSE
