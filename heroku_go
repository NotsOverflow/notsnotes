
# Heroku Go

### The setup
download and install [GIT](https://git-scm.com/) , [GoLang](https://golang.org/) ,  and [heroku](https://devcenter.heroku.com/articles/heroku-cli) .
##### Configure git:
```bash
git config --global user.name "John Doe"
git config --global user.email "johndoe@example.com"
```
##### Configure heroku:
```bash
$> heroku login
##Enter your Heroku credentials.
##Email: user@example.com
##Password:
```
##### Install govendor:
```bash
go get -u github.com/kardianos/govendor
```
### Build your first App
```bash
mkdir -p $GOPATH/src/helloworld
```
Create a ```main.go``` file with the following content:
```go
package main

import (  
  "log"  
  "fmt"  
  "net/http"  
  "os"  
)

func determineListenAddress() (string, error) {  
  port := os.Getenv("PORT")  
  if port == "" {  
    return "", fmt.Errorf("$PORT not set")  
  }  
  return ":" + port, nil  
}

func hello(w http.ResponseWriter, r *http.Request) {  
  fmt.Fprintln(w, "Hello World")  
}

func main() {  
  addr, err := determineListenAddress()  
  if err != nil {  
    log.Fatal(err)  
  }

  http.HandleFunc("/", hello)  
  log.Printf("Listening on %s...\n", addr)  
  if err := http.ListenAndServe(addr, nil); err != nil {  
    panic(err)  
  }  
}
```

##### Build and run:
```bash
$ go install ./...
$ PORT=5000 $GOPATH/bin/helloworld  
Listening on 5000...
```
You sould see a nice "Hello world" when accessing ```127.0.0.1:5000``` with your web browser :)

### Deploying to Heroku

```bash
$ git init
$ govendor init
$ printf "web: helloworld" > Procfile
$ git add vendor/vendor.json Procfile main.go
$ git commit -m "first go app"
$ heroku create -b https://github.com/heroku/heroku-buildpack-go.git -a unique-name-you-whant-123
$ git push heroku master
```
If everything went well you should be able to access ```https://unique-name-you-whant-123.herokuapp.com/``` with your web browser and see a nice "hello world" :)
##### Shud down app
```bash
$ heroku ps:scale web=0
```
##### Run Locally
```bash
$ heroku local web
```
