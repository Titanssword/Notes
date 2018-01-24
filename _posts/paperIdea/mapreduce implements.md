# This is the implementation of 6.824 Lab1: MapReduce

### Part 1: Map/Reduce and output

- What you have to do is to finish the two function doMap() and doReduce()
- Each call to doMap() reads the appropriate file, calls the map function on that file's contents, and writes the resulting key/value pairs to nReduce intermediate files. doMap() hashes each key to pick the intermediate file and thus the reduce task that will process the key. There will be nMap x nReduce files after all map tasks are done. Each file name contains a prefix, the map task number, and the reduce task number. If there are two map tasks and three reduce tasks, the map tasks will create these six intermediate files:
> mrtmp.xxx-0-0
> mrtmp.xxx-0-1
> mrtmp.xxx-0-2
> mrtmp.xxx-1-0
> mrtmp.xxx-1-1
> mrtmp.xxx-1-2

- The MapFunc ode and explanation are below

```
// first read the inFile and trasfer the results to the String
bytes, err := ioutil.ReadFile(inFile)
if err != nil {
    log.Fatalf("err: %s", err)
}
// in this part mapF() in the test_test.go. It Split the (inFile, bytes) in words, and get []KeyValue
// so we use a new Key structure to store the []KeyValue
kvs := mapF(inFile, string(bytes))
// This is standard use that can transfer the structure into JSON
encoders := make([]*json.Encoder, nReduce);
// for one input file, we need to generate *nReduce* output files for the reduce job
// so in this part we create *nReduce* Encoders, which are stored in the encoders, the
// file name is also generated by the reduceName provided by the MIT
for reduceTaskNumber := 0; reduceTaskNumber < nReduce; reduceTaskNumber++ {
    filename := reduceName(jobName, mapTaskNumber, reduceTaskNumber)
    // Create() 默认权限 0666
    file_ptr, err := os.Create(filename)
    if (err != nil) {
        log.Fatal("Unable to create file: ", filename)
    }
    defer file_ptr.Close()
    encoders[reduceTaskNumber] = json.NewEncoder(file_ptr);
}
// now what we are going to do is to encode the kv structure into JSON,
// Please Remember the use of it, here we use *&kv*  -- *the pointer* to get the structure
for _, kv := range kvs {
    key := kv.Key
    HashedKey := int(ihash(key) % nReduce)
    err := encoders[HashedKey].Encode(&kv)
    if err != nil {
        fmt.Println(err)
    }
}
```

- And another version firstly came to my mind, which is very slow
```
// In this version, we put the OpenFile method in the while of range kvs, in this way, we frequently
// openfile, which will cause a lot of time, so the results is not good due to the poor computer, we
// i also see someone's code is similar as this idea, maybe ther have a *great* computing environment.
for _, kv := range kvs {
    key := kv.Key
    HashedKey := int(ihash(key) % nReduce)
    filename := reduceName(jobName, mapTaskNumber, HashedKey)
    outputFile, _ := os.OpenFile(filename, os.O_WRONLY|os.O_APPEND|os.O_CREATE, 0666)
    enc := json.NewEncoder(outputFile)
    //err := encoders[HashedKey].Encode(&kv)
    err := enc.Encode(&kv)
    fmt.Println("writing ", key)
    if err != nil {
        fmt.Println(err)
    }
    outputFile.Close()
}
```

- the ReduceFunc code and explanation is below
```
kvMap := make(map[string]([]string))
// file all the intermediate files and transfer the json into kv structure
for mapNumber := 0; mapNumber < nMap; mapNumber++ {
    filename := reduceName(jobName, mapNumber, reduceTaskNumber)
    file, err := os.Open(filename)
    if err != nil {
        log.Fatal("err in open  file: %s", err)
    }
    defer file.Close()
    decoder := json.NewDecoder(file)
    for {
        var kv KeyValue
        if err := decoder.Decode(&kv); err == io.EOF {
            break
        } else if err != nil {
            log.Fatal(err)
        }
        //map append 方法
        kvMap[kv.Key] = append(kvMap[kv.Key], kv.Value)
    }
}
// use keys to store all the word in the file, the length is how many words
keys := make([]string, 0, len(kvMap))
for k, _ := range kvMap {
    keys = append(keys, k)
}
// just format, because the map structure is disordered
sort.Strings(keys)
// one reduce function generate one file which store the results of the reduceF
// It is going to be like this: word, 4, hello, 5
newFile , err2:= os.Create(mergeName(jobName, reduceTaskNumber))
if err2 != nil {
    fmt.Println("reduce merge files: %s cant open ", mergeName(jobName, reduceTaskNumber))
    return
}
enc := json.NewEncoder(newFile)
for _, k := range keys {
    enc.Encode(KeyValue{k, reduceF(k,  kvMap[k])})
    //fmt.Println("reduceF results is  %s")
    //fmt.Println(reduceF(k,  kvMap[k]))
}
// finally don't forget the Close() if you open a file
defer newFile.Close()


```
- finally you need run this code to check the correctness.

>cd 6.824
>export "GOPATH=$PWD"  # go needs $GOPATH to be set to the project's working directory
>cd "$GOPATH/src/mapreduce"
>go test -run Sequential

### Part 2 Single-worker word count

- Now you will implement word count — a simple Map/Reduce example. Look in main/wc.go; you'll find      empty mapF() and reduceF() functions. Your job is to insert code so that wc.go reports the number of occurrences of each word in its input. A word is any contiguous sequence of letters, as determined by unicode.IsLetter.

- this mapF function is similar in the test_test.go that we use to split the strings into words

```

// the point of this function is the strings.FieldsFunc which can divides the
// string into []string with resect to the unicode.IsLetter
func mapF(filename string, contents string) []mapreduce.KeyValue {
	words := strings.FieldsFunc(contents, func(r rune) bool {
		return !unicode.IsLetter(r)
	})
	var kv []mapreduce.KeyValue
	// one word is representing 1 time
	for _, word := range words {
		kv = append(kv, mapreduce.KeyValue{Key: word, Value: "1"})
	}
	return kv
}

// simply just count the values length
func reduceF(key string, values []string) string {
	return strconv.Itoa(len(values))
}

```

-  You can test your solution using:

>cd "$GOPATH/src/main"
` go run wc.go master sequential pg-*.txt `


- The output will be in the file "mrtmp.wcseq". Your implementation is correct if the following command produces the output shown here:

>sort -n -k2 mrtmp.wcseq | tail -10

>that: 7871
>it: 7987
>in: 8415
>was: 8578
>a: 13382
>of: 13536
>I: 14296
>to: 16079
>and: 23612
>the: 29748

### Part 3: Distributing MapReduce tasks

-  Your job is to implement schedule() in mapreduce/schedule.go. The master calls schedule() twice during a MapReduce job, once for the Map phase, and once for the Reduce phase. schedule()'s job is to hand out tasks to the available workers. There will usually be more tasks than worker threads, so schedule() must give each worker a sequence of tasks, one at a time. schedule() should wait until all tasks have completed, and then return.

- schedule() learns about the set of workers by reading its registerChan argument. That channel yields a string for each worker, containing the worker's RPC address. Some workers may exist before schedule() is called, and some may start while schedule() is running; all will appear on registerChan. schedule() should use all the workers, including ones that appear after it starts.

- schedule() tells a worker to execute a task by sending a Worker.DoTask RPC to the worker. This RPC's arguments are defined by DoTaskArgs in mapreduce/common_rpc.go. The File element is only used by Map tasks, and is the name of the file to read; schedule() can find these file names in mapFiles.

- Use the call() function in mapreduce/common_rpc.go to send an RPC to a worker. The first argument is the the worker's address, as read from registerChan. The second argument should be "Worker.DoTask". The third argument should be the DoTaskArgs structure, and the last argument should be nil.

- In this part you need to know some concepts
1. RPC package documents the Go RPC package.
2. schedule() should send RPCs to the workers in parallel so that the workers can work on tasks concurrently. You will find the go statement useful for this purpose; see Concurrency in Go.
schedule() must wait for a worker to finish before it can give it another task. *Go routines*
3. You may find *Go's channels* useful.
You may find *sync.WaitGroup* useful.
4. The easiest way to track down bugs is to insert print state statements (perhaps calling debug() in common.go), collect the output in a file with go test -run TestBasic > out, and then think about whether the output matches your understanding of how your code should behave. The last step (thinking) is the most important.

### Part 4: Handling worker failures
- MapReduce makes this relatively easy because workers don't have persistent state. If a worker fails, any RPCs that the master issued to that worker will fail (e.g., due to a timeout). Thus, if the master's RPC to the worker fails, the master should re-assign the task given to the failed worker to another worker.

- An RPC failure doesn't necessarily mean that the worker didn't execute the task; the worker may have executed it but the reply was lost, or the worker may still be executing but the master's RPC timed out. Thus, it may happen that two workers receive the same task, compute it, and generate output. Two invocations of a map or reduce function are required to generate the same output for a given input (i.e. the map and reduce functions are "functional"), so there won't be inconsistencies if subsequent processing sometimes reads one output and sometimes the other. In addition, the MapReduce framework ensures that map and reduce function output appears atomically: the output file will either not exist, or will contain the entire output of a single execution of the map or reduce function (the lab code doesn't actually implement this, but instead only fails workers at the end of a task, so there aren't concurrent executions of a task).

```

var wait_group sync.WaitGroup
// firstly, we have nTasks, and our job is dividing these tasks into the worker by the call function
for i:=0; i < ntasks; i++ {
    wait_group.Add(1)
    //struct of DoTaskArgs: the information of the job
    var args DoTaskArgs
    args.JobName = jobName
    if phase == mapPhase {
        args.File = mapFiles[i]
    }
    args.Phase = phase
    args.TaskNumber = i
    args.NumOtherPhase = n_other
    reply := ShutdownReply{0}
    // use go routines
    go func ()  {
        defer wait_group.Done()
        // keep runing until success
        for {
            // all the worker is stored in the registerChan channel
            worker := <-registerChan
            //func call(srv string, rpcname string, args interface{}, reply interface{})
            if (call(worker, "Worker.DoTask", &args, &reply)){
                go func(){registerChan <- worker} ()
                break
            }
            /*
            // firstly i want to use this form, but it will go wrong when i want to add the
            // else statement. i still don't know why
            succeeded := call(worker, "Worker.DoTask", &args, &reply)
            if !succeeded {
                fmt.Println("RPC call ERROR *****************")
            }
            else {
                fmt.Println("finished TaskNumber->>>",args.TaskNumber)
            }
            */
        }
    }()
}
// the finish
wait_group.Wait()
fmt.Printf("Schedule: %v phase done\n", phase)
```
- use below statement to test your code for the part 3

> go test -run TestBasic

- use below statement to test your code for the part 4
> go test -run Failure

### Part 5: Inverted index generation

- Inverted indices are widely used in computer science, and are particularly useful in document searching. Broadly speaking, an inverted index is a map from interesting facts about the underlying data, to the original location of that data. For example, in the context of search, it might be a map from keywords to documents that contain those words.

- We have created a second binary in main/ii.go that is very similar to the wc.go you built earlier. You should modify mapF and reduceF in main/ii.go so that they together produce an inverted index. Running ii.go should output a list of tuples, one per line, in the following format:

```
$ go run ii.go master sequential pg-*.txt
$ head -n5 mrtmp.iiseq

A: 8 pg-being_ernest.txt,pg-dorian_gray.txt,pg-frankenstein.txt,pg-grimm.txt,pg-huckleberry_finn.txt,pg-metamorphosis.txt,pg-sherlock_holmes.txt,pg-tom_sawyer.txt
ABOUT: 1 pg-tom_sawyer.txt
ACT: 1 pg-being_ernest.txt
ACTRESS: 1 pg-dorian_gray.txt
ACTUAL: 8 pg-being_ernest.txt,pg-dorian_gray.txt,pg-frankenstein.txt,pg-grimm.txt,pg-huckleberry_finn.txt,pg-metamorphosis.txt,pg-sherlock_holmes.txt,pg-tom_sawyer.txt
```
If it is not clear from the listing above, the format is:
```
word: #documents documents,sorted,and,separated,by,commas
```

- the mapF code and explanation are as BELOW:
```
// The mapping function is called once for each piece of the input.
// In this framework, the key is the name of the file that is being processed,
// and the value is the file's contents. The return value should be a slice of
// key/value pairs, each represented by a mapreduce.KeyValue.

func mapF(document string, value string) (res []mapreduce.KeyValue) {
	// TODO: you should complete this to do the inverted index challenge
	words := strings.FieldsFunc(value, func(r rune) bool {
		return !unicode.IsLetter(r)
	})
	// use DocMaps structure to store the word, inFile information
	// word, doc1,
	DocMaps := make(map[string]string)
	var kv []mapreduce.KeyValue
	// get the word from the words list and meanwhile we can get rid of the repeat word
	// for this structure repeat word is not the times, it has no other information.
	for _, word := range words {
		DocMaps[word] = document
	}
	for k, docMap := range DocMaps {
		kv = append(kv, mapreduce.KeyValue{k, docMap})
	}
	return kv
}

```

- the reduceF code and explanation are as BELOW:
```
// The reduce function is called once for each key generated by Map, with a
// list of that key's string value (merged across all inputs). The return value
// should be a single output value for that key.

func reduceF(key string, values []string) string {
	// TODO: you should complete this to do the inverted index challenge
	//return strconv.Itoa(len(values))
	// example doc1
	//example2 doc2
	nDoc := len(values)
	// fisrt, sort, because the map is disordered
	sort.Strings(values)
	// and then put the []string into string
	// be careful with the commas
	resString := strconv.Itoa(nDoc) + " "
	for _, v := range values {
		resString = resString + v + ","
	}
	lenString := len(resString)-1
	return resString[:lenString]
}

```
### conclusion

- Firstly, i have to admit that this lab is quite difficult for me. It took me nearly one
week to finish it. I spent a lot of time to know and how to use GO, till now, I still have
some confusion about the go routines and the go channel. The structure of the code is awesome.  
It also took me a long time in understanding the structure of the code, different parts have
different functions. But If you let me to create the whole MapReduce structure, I still don't
the confidence to finish it. I start with the window platform, but later, I am stuck in the PRC part. I got a lot of problems in the use of PRC. So I turn to the Ubuntu. finally, I still have some problems in part 2, actually, I think the result is same as the given result, but when I run the test bash, it told me different. And the part 5, the answer is also failed, but I have seen a lot of code from the github, they are the same idea. I have a look of the `pg-*.txt`, they are different from previous years. So I am not going to figure out what is going wrong. By implementing the MapReduce key part, I have a new look of this useful processing structure. With respect to my own research, I am considering that I can put the Sketches algorithm into the MapReduce computing structre, every node get its own results of the data stream Sketches, and in the end, master node put them together and get a whole sketch of the data stream.