closure
slices 是引用
map
for 可以range；  for i, v := range slice1 { ... } 

没有类，有struct
interface
method

一个type，比如说某自定义struct，实现interface是通过直接实现函数
empty interface can hold any type (Println is designed as empty interface)

type assertion: 
t := i.(T) => will panic if type wrong
t, ok := i.(T) => ok == false if type wrong

One of the most ubiquitous interfaces is [`Stringer`](https://go.dev/pkg/fmt/#Stringer) defined by the [`fmt`](https://go.dev/pkg/fmt/) package, which represents itself as a astring
type Stringer interface {
    String() string
}

Go programs express error state with `error` values.
The `error` type is a built-in interface similar to `fmt.Stringer`:
type error interface {
    Error() string
}

## Readers

The `io` package specifies the `io.Reader` interface, which represents the read end of a stream of data.

The Go standard library contains [many implementations](https://cs.opensource.google/search?q=Read%5C(%5Cw%2B%5Cs%5C%5B%5C%5Dbyte%5C)&ss=go%2Fgo) of this interface, including files, network connections, compressors, ciphers, and others.

The `io.Reader` interface has a `Read` method:

func (T) Read(b []byte) (n int, err error)

`Read` populates the given byte slice with data and returns the number of bytes populated and an error value. It returns an `io.EOF` error when the stream ends.

The example code creates a [`strings.Reader`](https://go.dev/pkg/strings/#Reader) and consumes its output 8 bytes at a time.

## go coroutine也就是协程：
go foo()即可，轻量级，用户态，切换不需要操作系统参与，因此性能优异
### channel使用：
Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`.

ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.

(The data flows in the direction of the arrow.)

Like maps and slices, channels must be created before use:

ch := make(chan int)

By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.

The example code sums the numbers in a slice, distributing the work between two goroutines. Once both goroutines have completed their computation, it calculates the final result.

#### channel可以设置大小
注意死锁问题，当channel overfill的时候会block

#### Sender可以关闭channel（永远不让receiver关）
A sender can `close` a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression: after

``` go
v, ok := <-ch
The loop `for i := range c` receives values from the channel repeatedly until it is closed.
```
#### Select
The `select` statement lets a goroutine wait on multiple communication operations.
A `select` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

#### sync.Mutex
mu.Lock(), Unlock(), defer c.mu.Unlock()