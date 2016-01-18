Go Commons Pool
=====

[![Build Status](https://travis-ci.org/jolestar/go-commons-pool.svg?branch=master)](https://travis-ci.org/jolestar/go-commons-pool)
[![Circle CI](https://circleci.com/gh/jolestar/go-commons-pool.svg?style=svg)](https://circleci.com/gh/jolestar/go-commons-pool)
[![Coverage Status](https://coveralls.io/repos/jolestar/go-commons-pool/badge.svg?branch=master&service=github&_day=201606)](https://coveralls.io/github/jolestar/go-commons-pool?branch=master)
[![GoDoc](http://godoc.org/github.com/jolestar/go-commons-pool?status.svg)](http://godoc.org/github.com/jolestar/go-commons-pool)

The Go Commons Pool is a generic object pool for [Golang](http://golang.org/), direct rewrite from [Apache Commons Pool](https://commons.apache.org/proper/commons-pool/).


Features
-------
1. Support custom [PooledObjectFactory](https://godoc.org/github.com/jolestar/go-commons-pool#PooledObjectFactory).
1. Rich pool configuration option, can precise control pooled object lifecycle. see [ObjectPoolConfig](https://godoc.org/github.com/jolestar/go-commons-pool#ObjectPoolConfig).
	* Pool LIFO (last in, first out) or FIFO (first in, first out) 
	* Pool cap config
	* Pool object validate config
	* Pool object borrow block and max waiting time config
	* Pool object eviction config
	* Pool object abandon config

Dependency
-------
* [testify](https://github.com/stretchr/testify) for test

Pool Configuration Option
-------

Configuration option table, more detail description see [ObjectPoolConfig](https://godoc.org/github.com/jolestar/go-commons-pool#ObjectPoolConfig)

| Option                        | Default        | Description  |
| ------------------------------|:--------------:| :------------|
| Lifo                          | true           |If pool is LIFO (last in, first out)|
| MaxTotal                      | 8              |The cap of pool|
| MaxIdle                       | 8              |Max "idle" instances in the pool |
| MinIdle                       | 0              |Min "idle" instances in the pool |
| TestOnCreate                  | false          |Validate when object is created|
| TestOnBorrow                  | false          |Validate when object is borrowed|
| TestOnReturn                  | false          |Validate when object is returned|
| TestWhileIdle                 | false          |Validate when object is idle, see TimeBetweenEvictionRunsMillis |
| BlockWhenExhausted            | true           |Whether to block when the pool is exhausted  |
| MaxWaitMillis                 | -1             |Max block time, less than 0 mean indefinitely|
| MinEvictableIdleTimeMillis    | 1000 * 60 * 30 |Eviction configuration,see DefaultEvictionPolicy |
| SoftMinEvictableIdleTimeMillis| math.MaxInt64  |Eviction configuration,see DefaultEvictionPolicy  |
| NumTestsPerEvictionRun        | 3              |The maximum number of objects to examine during each run evictor goroutine |
| TimeBetweenEvictionRunsMillis | -1             |The number of milliseconds to sleep between runs of the evictor goroutine, less than 0 mean not run |


Usage
-------

    //use create func
    pool := NewObjectPoolWithDefaultConfig(NewPooledObjectFactorySimple(
    		func() (interface{}, error) {
    			return &MyPoolObject{}, nil
    		}))
    obj, _ := pool.BorrowObject()
    pool.ReturnObject(obj)
    	
    //use custom Object factory
    
    type MyObjectFactory struct {
    	
    }
    
    func (this *MyObjectFactory) MakeObject() (*PooledObject, error) {
    	return NewPooledObject(&MyPoolObject{}), nil
    }
    
    func (this *MyObjectFactory) DestroyObject(object *PooledObject) error {
    	//do destroy
    	return nil
    }
    
    func (this *MyObjectFactory) ValidateObject(object *PooledObject) bool {
    	//do validate
    	return true
    }
    
    func (this *MyObjectFactory) ActivateObject(object *PooledObject) error {
    	//do activate
    	return nil
    }
    
    func (this *MyObjectFactory) PassivateObject(object *PooledObject) error {
    	//do passivate
    	return nil
    }
    
    pool := NewObjectPoolWithDefaultConfig(new(MyObjectFactory))
    pool.Config.MaxTotal = 100
    obj, _ := pool.BorrowObject()
    pool.ReturnObject(obj)

more example please see pool_test.go and example_test.go

Note
-------
PooledObjectFactory.MakeObject must return a pointer, not value.
The following code will complain error. 

	pool := NewObjectPoolWithDefaultConfig(NewPooledObjectFactorySimple(
		func() (interface{}, error) {
			return "hello", nil
		}))
	obj, _ := pool.BorrowObject()
	pool.ReturnObject(obj)

The right way is:

	pool := NewObjectPoolWithDefaultConfig(NewPooledObjectFactorySimple(
		func() (interface{}, error) {
			var stringPointer = new(string)
			*stringPointer = "hello"
			return stringPointer, nil
		}))

more example please see example_test.go


PerformanceTest
-------
The results of running the pool_perf_test is almost equal to the java version [PerformanceTest](https://github.com/apache/commons-pool/blob/trunk/src/test/java/org/apache/commons/pool2/performance/PerformanceTest.java)
    
    go test --perf=true

For Apache commons pool user
-------
* Direct use pool.Config.xxx to change pool config
* Default config value is same as java version
* If TimeBetweenEvictionRunsMillis changed after ObjectPool created, should call  **ObjectPool.StartEvictor** to take effect. Java version do this on set method.
* No KeyedObjectPool
* No ProxiedObjectPool
* No pool stats (TODO)

FAQ
-------
[FAQ](https://github.com/jolestar/go-commons-pool/wiki/FAQ)

How to contribute
-------
* Choose one open issue you want to solve, if not create one and describe what you want to change.
* Fork the repository on GitHub.
* Write code to solve the issue.
* Create PR and link to the issue.
* Make sure test and coverage pass.
* Wait maintainers to merge.

License
-------

Go Commons Pool is available under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).
