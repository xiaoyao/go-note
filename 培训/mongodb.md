# MongoDB

在mongodb中基本的概念是文档、集合、数据库

|SQL术语/概念|MongoDB术语/概念	|解释/说明|
|:------------:|:------------------:|---------|
|database	|database	|数据库|
|table	|collection	|数据库表/集合|
|row	|document	|数据记录行/文档|
|column	|field	|数据字段/域|
|index	|index	|索引|
|table |joins	 	|表连接,MongoDB不支持|
|primary key	|primary key	|主键,MongoDB自动将_id字段设置为主键

![](mongo1.png)


下表列出了 RDBMS 与 MongoDB 对应的术语：

RDBMS	|MongoDB|
---|---
数据库	|数据库
表格	|集合
行	|文档
列	|字段
表联合	|嵌入文档
主键	|主键 (MongoDB 提供了 key 为 _id )
**数据库服务和客户端**|
Mysqld/Oracle	|mongod
mysql/sqlplus	|mongo
## 特性
- 性能较好
- 无模式
- 支持二级索引
- 

## 缺点
- 不支持事务
- 不支持 join 操作
- 磁盘空间利用率不高
- 




分片



## 应用实践

### 索引


### 范式和反范式

***
# 操作：

***
## 短网址
###  映射函数
哈希

实现：
```go
type Instance struct {
    coll *mgo.Collection
    idx  int64
}
```

short 实现：
```go
func (p *Instance) Short(URL string) (string, error) {

    URL = strings.ToLower(URL)
    id := atomic.AddInt64(&p.idx, 1)
    idstr := strconv.FormatInt(id, 36)
    doc := &Entry{
        URL: URL,
    }

    if _, err := p.coll.UpsertId(idstr, doc); err != nil {
        return "", err
    }

    return idstr, nil
}
```

redirect 实现：

```go
func (p *Instance) Redirect(short string) (URL string, err error) {

    var entry Entry
    if err = p.coll.FindId(short).One(&entry); err != nil {
        return
    }
    return entry.URL, nil
}
```

### mongo 数据格式

## 问题
- 解决url重复问题
- 如何扩展到多个实例
- 




## code:
### main.go:

```go
package mongoDB
import (
	"github.com/qiniu/log"
)

func main() {

	conf := Config{
		Addr:    ":7090",
		MgoAddr: "127.0.0.1:27017",
		MgoDB:   "short",
		MgoColl: "short",
	}

	log.Fatal(Run(conf))
}

```
### server.go

```go
package mongoDB

import (
	"fmt"
	"net/http"
	"strconv"
	"strings"
	"sync/atomic"

	"github.com/qiniu/log"
	"gopkg.in/mgo.v2"
)

const startIdx = 1452652841

type Config struct {
	Addr    string `json:"addr"`
	MgoAddr string `json:"mongo_addr"`
	MgoDB   string `json:"mongo_db"`
	MgoColl string `json:"mongo_coll"`
}

func Run(conf Config) error {
	session, err := mgo.Dial(conf.MgoAddr)
	if err != nil {
		return err
	}
	coll := session.DB(conf.MgoDB).C(conf.MgoColl)

	ist, err := NewInstance(coll)
	if err != nil {
		return err
	}
	return ist.Run(conf.Addr)
}

//-----------------------------------------------------------------------------

func NewInstance(coll *mgo.Collection) (*Instance, error) {
	ist := &Instance{coll, 0}
	if err := ist.pickIdx(); err != nil {
		return nil, err
	}
	return ist, nil
}

type Entry struct {
	URL string `bson:"url"`
}

type M map[string]interface{}

type Instance struct {
	coll *mgo.Collection
	idx  int64
}

func (p *Instance) Run(addr string) error {
	mux := http.NewServeMux()
	mux.HandleFunc("/short", p.HandleShort)
	mux.HandleFunc("/", p.HandleRedirect)
	log.Info("running at:", addr)
	return http.ListenAndServe(addr, mux)
}

func (p *Instance) HandleShort(resp http.ResponseWriter, req *http.Request) {

	URL := req.FormValue("url")

	if URL == "" {
		replyErr(resp, "need url arg")
		return
	}

	if !strings.HasPrefix(URL, "http://") && !strings.HasPrefix(URL, "https://") {
		URL = "http://" + URL
	}

	short, err := p.Short(URL)
	if err != nil {
		replyErr(resp, err.Error())
		return
	}

	resp.Write([]byte(short))
	return
}

func (p *Instance) HandleRedirect(resp http.ResponseWriter, req *http.Request) {

	short := strings.TrimPrefix(req.URL.Path, "/")
	URL, err := p.Redirect(short)
	if err != nil {
		replyErr(resp, err.Error())
		return
	}
	resp.Header().Add("Location", URL)
	resp.WriteHeader(301)
	return
}

func replyErr(resp http.ResponseWriter, err string) {
	msg := fmt.Sprintf("err: %s\n", err)
	resp.WriteHeader(500)
	resp.Write([]byte(msg))
}

func (p *Instance) pickIdx() (err error) {

	p.idx = startIdx

	var doc map[string]interface{}
	if err = p.coll.Find(nil).Sort("-_id").Limit(1).One(&doc); err != nil {
		if err == mgo.ErrNotFound {
			return nil
		}
		return err
	}

	n, err := strconv.ParseInt(doc["_id"].(string), 36, 64)
	if err != nil {
		return
	}
	p.idx = n
	return
}

func (p *Instance) Short(URL string) (string, error) {

	URL = strings.ToLower(URL)
	id := atomic.AddInt64(&p.idx, 1)
	idstr := strconv.FormatInt(id, 36)
	doc := &Entry{
		URL: URL,
	}

	if _, err := p.coll.UpsertId(idstr, doc); err != nil {
		return "", err
	}

	return idstr, nil
}

func (p *Instance) Redirect(short string) (URL string, err error) {

	var entry Entry
	if err = p.coll.FindId(short).One(&entry); err != nil {
		return
	}
	return entry.URL, nil
}

```
### server_test.go
```go
package mongoDB

import (
	"testing"

	"gopkg.in/mgo.v2"
)

func initMgo(t *testing.T, dbname string) *mgo.Database {
	session, err := mgo.Dial("127.0.0.1")
	if err != nil {
		t.Fatal(err)
	}

	db := session.DB(dbname)
	db.DropDatabase()

	return db
}

func TestServer(t *testing.T) {
	db := initMgo(t, "short_test")

	coll := db.C("short")
	ist, err := NewInstance(coll)
	if err != nil {
		t.Fatal("new instance error -", err)
	}

	short, err := ist.Short("www.baidu.com")
	if err != nil {
		t.Fatal("short error -", err)
	}

	if short != "o0ve3u" {
		t.Fatal("short unpexted:", short)
	}

	short, err = ist.Short("www.google.com")
	if err != nil {
		t.Fatal("short error -", err)
	}

	if short != "o0ve3v" {
		t.Fatal("short unpexted:", short)
	}

	// test pickIdx
	ist2, err := NewInstance(coll)
	if err != nil {
		t.Fatal("new instance error -", err)
	}

	short, err = ist2.Short("www.qiniu.com")
	if err != nil {
		t.Fatal("short error -", err)
	}
	if short != "o0ve3w" {
		t.Fatal("short unpexted:", short)
	}

	// test redirect
	URL, err := ist.Redirect("o0ve3u")
	if err != nil {
		t.Fatal("redirect err -", err)
	}

	if URL != "www.baidu.com" {
		t.Fatal("redirect unexpected:", URL)
	}

	URL, err = ist.Redirect("o0ve3w")
	if err != nil {
		t.Fatal("redirect err -", err)
	}

	if URL != "www.qiniu.com" {
		t.Fatal("redirect unexpected:", URL)
	}
}

```