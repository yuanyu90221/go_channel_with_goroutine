# go_channel_with_goroutine
## introduction
當使用多個 goroutine來共同存取變數時 有兩種作法

1 communicate by sharedMemory
意思就是寫一個global variable
然後透過對這個變數的存取來分享訊息
但這樣做需要注意到
對寫入變數的會有複寫問題

因此對於同個變數的存取
為了避免同時寫入
所以需要使用lock方式在race conditon的地方
舉例來說：
```golang===
func addByShareMemory(n int) []int {
	var ints []int
	var wg sync.WaitGroup
	var mux sync.Mutex

	wg.Add(n)
	for i := 0; i < n; i++ {
		go func(i int) {
			defer wg.Done()
			mux.Lock()
			ints = append(ints, i)
			mux.Unlock()
		}(i)
	}

	wg.Wait()
	return ints
}
```

2 shareCommunicate
利用channel的特性
假如說 需要n個寫入 就開 buffer為n的channel
因為各自寫入 所以不會有複寫的問題

然後在read的地方 因為是寫入 才會讀取 
因此不用怕讀取不到

舉例來說:
```golang===
func addByShareCommunicate(n int) []int {
	var ints []int
	channel := make(chan int, n)

	for i := 0; i < n; i++ {
		go func(channel chan<- int, order int) {
			channel <- order
		}(channel, i)
	}

	for i := range channel {
		ints = append(ints, i)

		if len(ints) == n {
			break
		}
	}
	close(channel)
	return ints
}
```