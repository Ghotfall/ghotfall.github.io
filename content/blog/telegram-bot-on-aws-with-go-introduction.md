+++
author = "Ghotfall"
categories = ["AWS", "Golang", "DevOps", "Programming"]
date = ""
description = "First article about writing Telegram Bot using Go (Golang) and AWS. You will learn a few basic tips to help you start creating your own bot."
draft = true
images = []
tags = ["AWS", "Bot", "Telegram", "Golang", "Go"]
title = "Telegram Bot on AWS with Go: introduction"

+++
Telegram bots make our life simpler every day. They can be used to solve a wide variety of tasks, from entertainments in your spare time to automation of your work routine.

You can easily make your own bot using Golang and then host it on AWS Lambda and API Gateway, which would be quite cheap (or even free, if your resources usage stay under AWS Free Tier limits). Serverless computing is very usefull for this kind of tasks!

So, let's write some code!

I will use [go-telegram-bot-api](https://github.com/go-telegram-bot-api/telegram-bot-api) as binding for the Telegram Bot API. Let's install it:

```bash
go get -u github.com/go-telegram-bot-api/telegram-bot-api@master
```

**Note: it will be better to use latest changes from the repository via @master because @latest version is older and has less features.**

In addition, we will need [aws-lambda-go](https://github.com/aws/aws-lambda-go) for working with AWS Lambda.

```bash
go get -u github.com/aws/aws-lambda-go
```

And the last preparation will be registration of our bot inside Telegram. Make sure you get an API token from [@Botfather](https://t.me/Botfather) before moving forward.

**Note: you can read** [**official docs**](https://core.telegram.org/bots) **in order to get some more info about  Telegram Bot API.**

Ok, now we can write some code. Firstly, some boilerplate code is needed for working with AWS Lambda:

```go
package main

import (
	"context"
	"github.com/aws/aws-lambda-go/lambda"
)

func main() {
	lambda.Start(HandleRequest)
}

func HandleRequest(ctx context.Context, req events.APIGatewayV2HTTPRequest) (events.APIGatewayV2HTTPResponse, error) {
	// Your code...
    
	return events.APIGatewayV2HTTPResponse{StatusCode: 200}, nil
}
```

Our main function only starts HandleRequest function. HandleRequest will include the code which will be executed. Not every handler will be valid for execution inside AWS Lambda. There is a list of acceptable signatures:

* ```func ()```
* ```func () error```
* ```func (TIn), error```
* ```func () (TOut, error)```
* ```func (context.Context) error```
* ```func (context.Context, TIn) error```
* ```func (context.Context) (TOut, error)```
* ```func (context.Context, TIn) (TOut, error)```

asfd