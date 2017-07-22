---
author : "杨承龙"
date : "2017-06-30T16:36:36+08:00"
draft : false
title : "Goroutine崩溃后"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
Go routine 崩溃后

验证后发现Go routine 崩溃后，不会影响其他运行过程.

<!--*more*-->

测试代码如下：

    package main
    
    import (
    	"bufio"
    	"bytes"
    	"fmt"
    	"os/exec"
    	"time"
    )
    
    func main() {
    	go demo1()
    
    	c := make(chan bool, 2)
    	go saferoutine(c)
    	for i := 0; i < 2; i++ {
    		fmt.Println(<-c)
    	}
    }
    
    func saferoutine(c chan bool) {
    	for i := 0; i < 4; i++ {
    		fmt.Println("Count:", i)
    		time.Sleep(1 * time.Second)
    	}
    	c <- true
    	close(c)
    }
    
    func demo1() {
    	fmt.Println("Run command `ps aux | grep apipe`: ")
    
    	cmd1 := exec.Command("ps", "aux")
    	cmd2 := exec.Command("grep", "apipe")
    
    	stdout1, err := cmd1.StdoutPipe()
    	if err != nil {
    		fmt.Printf("Error: Can not obtain the stdout pipe for command: %s", err)
    		return
    	}
    	
    	if err := cmd1.Start(); err != nil {
    		fmt.Printf("Error: The command can not running: %s\n", err)
    		return
    	}
    
    	outputBuf1 := bufio.NewReader(stdout1)
    	stdin2, err := cmd2.StdinPipe()
    	if err != nil {
    		fmt.Printf("Error: Can not obtain the stdin pipe for command: %s\n", err)
    		return
    	}
    	outputBuf1.WriteTo(stdin2)
    	
    	var outputBuf2 bytes.Buffer
    	cmd2.Stdout = &outputBuf2
    	
    	if err := cmd2.Start(); err != nil {
    		fmt.Printf("Error: The command can not be startup: %s\n", err)
    		return
    	}
    
    	err = stdin2.Close()
    
    	if err != nil {
    		fmt.Printf("Error: Can not close the stdio pipe: %s\n", err)
    		return
    	}
    	if err := cmd2.Wait(); err != nil {
    		fmt.Printf("Error: Can not wait for the command: %s\n", err)
    		return
    	}
    
    	fmt.Printf("%s\n", outputBuf2.Bytes())
    }

输出如下：

    yangs-MacBook-Air:tests yang$ go run test_gorun.go 
    Count: 0
    Run command `ps aux | grep apipe`: 
    Error: Can not wait for the command: exit status 1
    Count: 1
    Count: 2
    Count: 3
    true
    false
    yangs-MacBook-Air:tests yang$ 

在demo的goroutine退出后， saferoutine以及之后的for循环仍可以正常运行

 在上面的例子中只是验证了最简单的情况。下面这个例子中，demo在开始时加锁，后来就崩溃了。。。之后defer语句发挥作用，执行unlock。说明这种情况下崩溃的不彻底，相比panic还是比较友好的。在demo1的goroutine崩溃后，safegoroutine仍可以正常执行，即这种情况下，goroutine 崩溃后，不会影响其他运行过程.

    package main
    
    import (
    	"fmt"
    	"os/exec"
    	"time"
    	"sync"
    )
    
    var mutex = &sync.Mutex{}
    
    func main() {
    	go demo1()
    
    	c := make(chan bool, 2)
    	go saferoutine(c)
    	fmt.Println(<-c)
    }
    
    func saferoutine(c chan bool) {
    	time.Sleep(4*time.Second)
    	mutex.Lock()
    	defer mutex.Unlock()
    
    	fmt.Println("go already lock!")
    	c <- true
    	close(c)
    	fmt.Println("go exit")
    }
    
    func demo1() {
    	mutex.Lock()
    	defer mutex.Unlock()
    
    	fmt.Println("demo1 already lock!")
    	cmd2 := exec.Command("grep", "go")
    
    	if err := cmd2.Start(); err != nil {
    		fmt.Printf("Error: The command can not be startup: %s\n", err)
    		return
    	}
    
    	if err := cmd2.Wait(); err != nil {
    		fmt.Printf("Error: Can not wait for the command: %s\n", err)
    		return
    	}
    }

输出如下：

    bogon:tests yang$ go run test_gorun.go 
    demo1 already lock!
    Error: Can not wait for the command: exit status 1
    go already lock!
    go exit
    true
    bogon:tests yang$ 
