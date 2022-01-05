---
title: Go语言JSON库的使用
date: 2018-03-05 11:52:37
tags: 
  - golang
  - json
categories: 
  - golang
---

```json
{
  "name": "jack",
  "age": 21
}
```

```go
type User struct {
  Name string `json:"name"`
  Age int `json:"age"`
}

user := User{}
json.Unmarshal([]byte(jsonStr), &user)

```