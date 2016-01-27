# database/sql接口


对数据库无内置驱动支持，用户基于驱动接口开发相应数据库的驱动

## database/sql 接口：

database/sql在database/sql/driver提供的接口基础上定义了一些更高阶的方法,用以简化数据库操作,同时内部还 建议性地实现一个conn pool。

```
// DB is a database handle representing a pool of zero or more
// underlying connections. It's safe for concurrent use by multiple
// goroutines.
//
// The sql package creates and frees connections automatically; it
// also maintains a free pool of idle connections. If the database has
// a concept of per-connection state, such state can only be reliably
// observed within a transaction. Once DB.Begin is called, the
// returned Tx is bound to a single connection. Once Commit or
// Rollback is called on the transaction, that transaction's
// connection is returned to DB's idle connection pool. The pool size
// can be controlled with SetMaxIdleConns.
type DB struct {
   driver driver.Driver
   dsn    string
   // numClosed is an atomic counter which represents a total number of
   // closed connections. Stmt.openStmt checks it before cleaning closed
   // connections in Stmt.css.
   numClosed uint64

   mu           sync.Mutex // protects following fields
   freeConn     []*driverConn
   connRequests []chan connRequest
   numOpen      int
   pendingOpens int
   // Used to signal the need for new connections
   // a goroutine running connectionOpener() reads on this chan and
   // maybeOpenNewConnections sends on the chan (one send per needed connection)
   // It is closed during db.Close(). The close tells the connectionOpener
   // goroutine to exit.
   openerCh chan struct{}
   closed   bool
   dep      map[finalCloser]depSet
   lastPut  map[*driverConn]string // stacktrace of last conn's put; debug only
   maxIdle  int                    // zero means defaultMaxIdleConns; negative means 0
   maxOpen  int                    // <= 0 means unlimited
}
```


	1、sql.Register：
函数；注册数据库驱动；
init 函数里调用Register(name string, driver driver.Driver)注册驱动

	2、driver.Driver
数据库驱动的接口
定义了一个method：Open(name string)，返回一个数据库的 Conn 接口
只能用在一个 goroutine 里面

	3、driver.Conn
数据库连接的接口定义
只能用在一个 goroutine 里面

	4、driver.Stmt
一种准备好的状态，和Conn相关联
只能用在一个 goroutine 里面

```
type Stmt interface {
   // Close closes the statement.
   //
   //As of Go 1.1, a Stmt will not be closed if in use by any queries.
   Close() error

   NumInput() int

   // Exec executes a query that doesn't return rows, such
   // as an INSERT or UPDATE.
   Exec(args []Value) (Result, error)

   // Query executes a query that may return rows, such as a
   // SELECT.
   Query(args []Value) (Rows, error)
}
```

Exec函数执行Prepare准备好的sql,传入参数执行update/insert等操作,返回Result数据 

Query函数执行Prepare准备好的sql,传入需要的参数执行select操作,返回Rows结果集 



