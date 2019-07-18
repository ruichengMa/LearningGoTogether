# 如何优雅地关闭Channel
        package main
        
        import "fmt"
        
        type T int
        
        func IsClosed(ch <-chan T) bool {
            select {
            case <-ch:
                return true
            default:
            }
            
            return false
        }
        
        func main() {
            c := make(chan T)
            fmt.Println(IsClosed(c)) // false
            close(c)
            fmt.Println(IsClosed(c)) // true
        }
        如上所述，这并不是检查通道是否关闭的通用方法。

实际上，即使有一个简单的函数closed(chan T) bool可以检查通道是否关闭，它的用处也会非常有限，就像内置函数len一样。原因在于，调用此类函数并返回后，被检查通道的状态可能已经更改，因此返回的值已无法反应刚刚检查的通道的最新状态。即使在调用closed(ch)返回true后，可以停止向channel ch 中继续发送值，但是调用closed(ch)返回false后，关闭通道或继续向通道中发送值却并不安全。

## Channel 关闭的原则
使用Go Channel的一个通用原则是不要在接收者一侧关闭通道，并且，如果通道具有多个并发的发送者，也不要关闭通道。换句话说，如果发送者是该通道的唯一发送者，我们应该只在发送者一侧的goroutine中关闭通道。

当然，这不是关闭通道的普遍原则。通用原则是不关闭已关闭的通道(或向已关闭的通道发送值)。如果我们可以保证不再有goroutine关闭(或发送)未关闭的非零通道，那么goroutine可以安全地关闭通道。然而，由接收者或通道的许多发送者之一做出这种保证通常需要很多努力，并且容易使代码变得复杂。相反，保持上述通道关闭原则会很容易。

## 粗暴关闭Channel的解决方案
如果你无论如何都要在接收者一侧关闭通道，或者在通道众多的发送者中的某一个goroutine中关闭通道，那么你可以使用recover机制来防止可能的Panic导致的程序崩溃。这是一个例子：
        func SafeClose(ch chan T) (justClosed bool) {
            defer func() {
                if recover() != nil {
                    // The return result can be altered
                    // in a defer function call.
                    justClosed = false
                }
            }()
            
            // assume ch != nil here.
            close(ch)   // panic if ch is closed
            return true // <=> justClosed = true; return
        }
        这个解决方案显然打破了通道关闭原则。
        
可以使用相同的方法将值发送到一个潜在的已关闭通道中。
        func SafeSend(ch chan T, value T) (closed bool) {
            defer func() {
                if recover() != nil {
                    closed = true
                }
            }()
            
            ch <- value  // panic if ch is closed
            return false // <=> closed = false; return
        }
## 礼貌地关闭Channel的解决方案
大多数人倾向于使用sync.Once来关闭通道
        type MyChannel struct {
            C    chan T
            once sync.Once
        }
        
        func NewMyChannel() *MyChannel {
            return &MyChannel{C: make(chan T)}
        }
        
        func (mc *MyChannel) SafeClose() {
            mc.once.Do(func() {
                close(mc.C)
            })
        }
当然，我们也可以使用sync.Mutex来避免多次关闭通道
        
        type MyChannel struct {
            C      chan T
            closed bool
            mutex  sync.Mutex
        }
        
        func NewMyChannel() *MyChannel {
            return &MyChannel{C: make(chan T)}
        }
        
        func (mc *MyChannel) SafeClose() {
            mc.mutex.Lock()
            defer mc.mutex.Unlock()
            if !mc.closed {
                close(mc.C)
                mc.closed = true
            }
        }
        
        func (mc *MyChannel) IsClosed() bool {
            mc.mutex.Lock()
            defer mc.mutex.Unlock()
            return mc.closed
        }
        
我们应该理解为什么Go不支持内置的SafeClose函数的原因是，它被认为是Go中从接收者或多个并发发送者中的一个关闭通道的不良设计实践。Go甚至不允许关闭一个只接收通道。

## 优雅地关闭Channel的解决方案
上述SafeSend函数的一个缺点是它不能用在select块中case关键字之后，上述SafeSend和SafeClose函数的另一个缺点是很多人会认为使用panic/recover和sync包是不优雅的。因此下面将介绍一些不使用panic/recover和sync包的纯channel解决方案，适用于各种情况。

#### 1. M 个接收者，一个发送者，发送者通过关闭通道说“不再发送”
这是最简单的情况，只需让发送者在不想发送更多数据时关闭通道
        package main
        
        import (
            "time"
            "math/rand"
            "sync"
            "log"
        )
        
        func main() {
            rand.Seed(time.Now().UnixNano())
            log.SetFlags(0)
            
            // ...
            const MaxRandomNumber = 100000
            const NumReceivers = 100
            
            wgReceivers := sync.WaitGroup{}
            wgReceivers.Add(NumReceivers)
            
            // ...
            dataCh := make(chan int, 100)
            
            // the sender
            go func() {
                for {
                    if value := rand.Intn(MaxRandomNumber); value == 0 {
                        // The only sender can close the channel safely.
                        close(dataCh)
                        return
                    } else {            
                        dataCh <- value
                    }
                }
            }()
            
            // receivers
            for i := 0; i < NumReceivers; i++ {
                go func() {
                    defer wgReceivers.Done()
                    
                    // Receive values until dataCh is closed and
                    // the value buffer queue of dataCh is empty.
                    for value := range dataCh {
                        log.Println(value)
                    }
                }()
            }
            
            wgReceivers.Wait()
        }
#### 2. 一个接收者，N个发送者，接收者通过关闭一个额外的信号通道说“请停止发送更多”
这种情况比上述情况稍微复杂一些。我们不能让接收者关闭通道来停止数据传输，这样做会打破通道关闭原则。但我们可以让接收者关闭一个额外的信号通道，以通知发送者停止发送。
        package main
        
        import (
        	"time"
        	"math/rand"
        	"sync"
        	"log"
        )
        
        func main() {
        	rand.Seed(time.Now().UnixNano())
        	log.SetFlags(0)
        
        	// ...
        	const MaxRandomNumber = 100000
        	const NumSenders = 1000
        
        	wgReceivers := sync.WaitGroup{}
        	wgReceivers.Add(1)
        
        	// ...
        	dataCh := make(chan int, 100)
        	stopCh := make(chan struct{})
        	// stopCh is an additional signal channel.
        	// Its sender is the receiver of channel dataCh.
        	// Its reveivers are the senders of channel dataCh.
        
        	// senders
        	for i := 0; i < NumSenders; i++ {
        		go func() {
        			for {
        				// The try-receive operation is to try to exit
        				// the goroutine as early as possible. For this
        				// specified example, it is not essential.
        				select {
        				case <- stopCh:
        					return
        				default:
        				}
        
        				// Even if stopCh is closed, the first branch in the
        				// second select may be still not selected for some
        				// loops if the send to dataCh is also unblocked.
        				// But this is acceptable for this example, so the
        				// first select block above can be omitted.
        				select {
        				case <- stopCh:
        					return
        				case dataCh <- rand.Intn(MaxRandomNumber):
        				}
        			}
        		}()
        	}
        
        	// the receiver
        	go func() {
        		defer wgReceivers.Done()
        
        		for value := range dataCh {
        			if value == MaxRandomNumber-1 {
        				// The receiver of the dataCh channel is
        				// also the sender of the stopCh channel.
        				// It is safe to close the stop channel here.
        				close(stopCh)
        				return
        			}
        
        			log.Println(value)
        		}
        	}()
        
        	// ...
        	wgReceivers.Wait()
        }
        
如注释中所述，对于额外的信号通道，其发送者是数据通道的接收者。额外的信号通道由其唯一的发送者关闭，该发送者保证了通道关闭原则。

在这个例子中，通道dataCh永远不会关闭。是的，通道不必关闭。如果没有goroutine再次引用它，无论是否关闭，通道最终都会被垃圾回收。因此，这里所谓的优雅关闭并不是关闭这个通道。

#### 3. M 个接收者，N 个发送者，其中任何一个通过通知一个哨兵关闭信号通道说“让我们结束游戏”
这是最复杂的情况。我们不能让任何接收者和发送者关闭数据通道，也不能让任何接收者关闭额外的信号通道以通知所有发送者和接收者退出游戏。这么做都会打破通道关闭原则。但是，我们可以引入一个哨兵的角色来关闭额外的信号通道。以下示例的一个技巧是如何使用try-send操作来通知哨兵关闭额外的信号通道。
        
        package main
        
        import (
        	"time"
        	"math/rand"
        	"sync"
        	"log"
        	"strconv"
        )
        
        func main() {
        	rand.Seed(time.Now().UnixNano())
        	log.SetFlags(0)
        
        	// ...
        	const MaxRandomNumber = 100000
        	const NumReceivers = 10
        	const NumSenders = 1000
        
        	wgReceivers := sync.WaitGroup{}
        	wgReceivers.Add(NumReceivers)
        
        	// ...
        	dataCh := make(chan int, 100)
        	stopCh := make(chan struct{})
        	// stopCh is an additional signal channel.
        	// Its sender is the moderator goroutine shown below.
        	// Its reveivers are all senders and receivers of dataCh.
        	toStop := make(chan string, 1)
        	// The channel toStop is used to notify the moderator
        	// to close the additional signal channel (stopCh).
        	// Its senders are any senders and receivers of dataCh.
        	// Its reveiver is the moderator goroutine shown below.
        	// It must be a buffered channel.
        
        	var stoppedBy string
        
        	// moderator
        	go func() {
        		stoppedBy = <-toStop
        		close(stopCh)
        	}()
        
        	// senders
        	for i := 0; i < NumSenders; i++ {
        		go func(id string) {
        			for {
        				value := rand.Intn(MaxRandomNumber)
        				if value == 0 {
        					// Here, the try-send operation is to notify the
        					// moderator to close the additional signal channel.
        					select {
        					case toStop <- "sender#" + id:
        					default:
        					}
        					return
        				}
        
        				// The try-receive operation here is to try to exit the
        				// sender goroutine as early as possible. Try-receive
        				// try-send select blocks are specially optimized by the
        				// standard Go compiler, so they are very efficient.
        				select {
        				case <- stopCh:
        					return
        				default:
        				}
        
        				// Even if stopCh is closed, the first branch in this
        				// select block may be still not selected for some
        				// loops (and for ever in theory) if the send to dataCh
        				// is also non-blocking. If this is not acceptable,
        				// then the above try-receive operation is essential.
        				select {
        				case <- stopCh:
        					return
        				case dataCh <- value:
        				}
        			}
        		}(strconv.Itoa(i))
        	}
        
        	// receivers
        	for i := 0; i < NumReceivers; i++ {
        		go func(id string) {
        			defer wgReceivers.Done()
        
        			for {
        				// Same as the sender goroutine, the try-receive
        				// operation here is to try to exit the receiver
        				// goroutine as early as possible.
        				select {
        				case <- stopCh:
        					return
        				default:
        				}
        
        				// Even if stopCh is closed, the first branch in this
        				// select block may be still not selected for some
        				// loops (and for ever in theory) if the receive from
        				// dataCh is also non-blocking. If this is not acceptable,
        				// then the above try-receive operation is essential.
        				select {
        				case <- stopCh:
        					return
        				case value := <-dataCh:
        					if value == MaxRandomNumber-1 {
        						// The same trick is used to notify
        						// the moderator to close the
        						// additional signal channel.
        						select {
        						case toStop <- "receiver#" + id:
        						default:
        						}
        						return
        					}
        
        					log.Println(value)
        				}
        			}
        		}(strconv.Itoa(i))
        	}
        
        	// ...
        	wgReceivers.Wait()
        	log.Println("stopped by", stoppedBy)
        }
        
在这个例子中，通道关闭原则仍然可以保证。
请注意，通道toStop的缓冲区大小为1。这是为了避免在哨兵goroutine准备好接收来自toStop的通知之前发送第一个通知。
我们还可以将toStop通道的容量设置为发送者和接收者数量之和，然后我们不需要try-send select块来发送通知。
        
        ...
        toStop := make(chan string, NumReceivers + NumSenders)
        ...
                    value := rand.Intn(MaxRandomNumber)
                    if value == 0 {
                        toStop <- "sender#" + id
                        return
                    }
        ...
                        if value == MaxRandomNumber-1 {
                            toStop <- "receiver#" + id
                            return
                        }
        ...
        
        作者：绝望的祖父
        链接：https://www.jianshu.com/p/c7b25ed78b89
        来源：简书
        简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
        
转自: https://www.jianshu.com/p/c7b25ed78b89