---
title: "How I structure Cloud Functions in Go"
date: 2020-02-16T11:46:58+01:00
draft: false
toc: true
images:
tags: 
  - go
  - gcloud
  - cloud functions
  - guideline
---

I started deploying cloud functions in Golang when they were first released in beta last year. The Go runtime support
 for cloud functions currently supports Go version 1.11.6 and Go version 1.13.1 (Beta). This is the most stable it
  has been and in the last 6 months of running functions in production, I have faced a few 'aha' moments that I could
   have avoided if I had a guide like the one am writing now. This post, hence, is more about the journey and the
    structure I use now after iterating with a few different techniques.

### Context
I use cloud functions to handle small routine tasks, sometimes from the client, sometimes as Subscribers to
 Google PubSub topics. So, am deploying both [Background functions](https://cloud.google.com/functions/docs/writing/background)
 and [HTTP functions](https://cloud.google.com/functions/docs/writing/http). All the invocations require a service
  account or a logged in firebase user and are secure. These limitations are required to operate in a safe
   environment, considering you are charged for the number of times your function is invoked.

There are also caveats around cold function invocations vs hot starts. Most of the time, your function
 will already be running in an instance and would be ready to respond to triggers, however,  there are times when
  your function might be in something called a "cold state" because of any of the following reasons:
  
* Your function has been deployed but not triggered yet.
* Your function has been idle long enough for it to be recycled to free up resources.
* Your function is auto-scaling to create capacity. 

All of these, require careful consideration on what dependencies you need to include in your functions and what you
 need to place in the `init()` method to make sure you do all the heavy computation and initialisations there.
 
>With this in mind, lets start approaching the problem at hand. We need to structure cloud functions in a way that the
> cold boot times are small, we could run the function locally & we could define private dependencies if need be

### Process
So, we have a set criteria and boundaries that we must operate in to milk the most out of our cloud functions. 

My initial approach was to club all my functions together in one repository and have deploy scripts to deploy them if
 some common dependencies changed. This, as you would imagine got to a mess by the time I had reached the fifth
  function. Apart from the usual nightmares you get in such a shared code base, my honest intentions of not
   wanting to repeat a lot of code had now ended up in me trying to duplicate files cause you need to deploy all the
    dependencies along with every function. If you have common code for initialisation you would end up copying this
     into all your functions.

Like any sane Go developer I decided to create a package of my own in the same repository and import that package
 into my functions, but this does not work either cause you would then need to upload the whole package directory and
  all the other cloud functions along with it, because you are uploading the whole root folder.
  
```shell

// crappy idea

|-- pkg
    |
    |-- common code
|-- function1
    |-- function code
|-- function2
    |-- function code
```

So, I did the next sane thing, create a module and import this module from an external repository that had to be
 private. For those working on Go, you already know how modules work and what the vendoring mode is, I will not go
  into that and if you need to know more you could read through https://blog.golang.org/using-go-modules. This led to
   a new branch of problems because, if you have private repositories, cloud deploy will not be able to pull these
    modules and you need to use vendoring mode. This was easy to solve and is well documented [here](https://cloud
    .google.com/functions/docs/writing/specifying-dependencies-go)
       
With this I came to the structure that worked for me for some time.

```shell
.
├── README.md
├── cmd
│   ├── func1
│   │   └── main.go
│   ├── func2
│   │   └── main.go
├── deploy.sh
├── go.mod
├── go.sum
├── pkg
│   ├── func1
│   │   ├── func1.go
│   │   ├── someInit.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   └── vendor
│   ├── func2
│   │   ├── func2.go
│   │   ├── someInit.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   └── vendor
```

This structure had its own problems. First, multiple go.mod files in the same repository made it almost impossible to
 test the functions locally. My process to work with this structure on local was to:
 * checkout to a different branch
 * push the branch with function changes
 * run `go get -d func1@branchName` inside `cmd/` package
 * proceed with running a local server to test
 
 This along with the `deploy.sh` was troublesome for me. The multiple go.mod's wreaked havoc and I was at a point in
  time I wanted different environment variables for different functions. However, all these functions shared common
   initialisation script I did not want to rewrite.
   
### The structure
What I have finally ended up with now, helps me code and deploy faster, is a breeze to test and is easy to run
 locally. The first thing I did was to split each function to its own repository. To do this, all initialisation code
  was packaged together into an external package. 

This is the structure of my function now:

```shell
.
├── README.md
├── .env.yaml
├── .gcloudignore
├── cmd
│   └── main.go
├── deploy.sh
├── functions.go
├── FunctionOne.go
├── FunctionOne_test.go
├── go.mod
├── go.sum
└── vendor
``` 

I have a functions.go that initializes the dependencies in the event of a cold restart. These are reused every time a
 function is called.
   
// functions.go
```go

// Package function is the package that houses my cloud function
package function

import (
        "context"
        "encoding/json"
        "fmt"
        "io/ioutil"
        "log"
        "net/http"
        "os"

        "github.com/automaticalldramatic/mypackage"
)

// projectID is set from the GCP_PROJECT environment variable, which is
// automatically set by the Cloud Functions runtime.
var envVar = os.Getenv("SOME_VAR")

var (
	client *mypackage.Client
	logger *log.Logger
)

func init() {
        // err is pre-declared to avoid shadowing client.
        var err error
        
        logger = log.New(os.Stdout, "[Timefox::Function::"+funcName+"]", log.LstdFlags)


        // client is initialized with context.Background() because it should
        // persist between function invocations.
        client, err = mypakcage.NewClient(context.Background(), envVar)
        if err != nil {
                logger.Fatalf("something: %v", err)
        }
}
```

The external package also has error codes for retry and no retry specific scenarios. These are important when you are
 writing background functions and don't want certain scenarios to be retried even though the execution errored out.

```go
&package.ErrorResponse{
    Error:   errors.New("400 - Bad Request"),
    Message: "You are not welcome here, my dear",
    Code:    http.StatusNoContent, // we don't retry the message with a 204
}
```

To run the function locally, I have created a command module with a main file. This is how it usually looks:

```go

package main

import (
	"log"
	"net/http"
	"net/http/httputil"

	"github.com/gorilla/mux"

	"github.com/automaticalldramatic/server"

	function "github.com/automaticalldramatic/function-module"
)

func main() {

	options := server.DefaultOptions
	options.Address = ":8080"
	s := server.NewServer(options)

	r := mux.NewRouter()

	log.Printf("server started on: %s", options.Address)
	
	r.Use(VerifyHTTPRequest)

	r.HandleFunc("/function/method", function.FunctionName)
	s.Handler = r
	log.Fatal(s.ListenAndServe())
}
```

This helps me test locally and apply middlewares to check my auth tokens in case I want to. 

### Deploying
I use `.gcloudignore` to ignore the go.mod files, since am using the vendoring mode due to private packages. You
 could skip this if you like, but its generally considered good practice to have one of these.

```shell
# For more information, run:
#   $ gcloud topic gcloudignore
#
.gcloudignore
deploy.sh

.env.yaml

go.mod
go.sum
```

The `deploy.sh` could be replaced by any CI. The `.env.yaml` is added to `.gitignore` and `.gcloudignore` and is only
 for local deployments. When you use a CI setup, you would ideally add env variables and secrets to your CI.

However, this is what I run essentially to deploy the function from my local deploy file using the environment file:

```shell script
gcloud functions deploy FunctionName --entry-point FunctionName --trigger-http  --region europe-west1 --runtime go113
 --env-vars-file ./.env.yaml  --clear-labels --update-labels version=someVersion
```

I hope this helps you deploy your cloud function and maintain it. If you need further help, [the documentation](https
://cloud.google.com/functions/docs/how-to) is excellent and covers almost all the use cases.
