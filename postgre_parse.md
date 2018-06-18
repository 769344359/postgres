```c
typedef struct List
{
	NodeTag		type;			/* T_List, T_IntList, or T_OidList */
	int			length;
	ListCell   *head;
	ListCell   *tail;
} List;
```
> 首先分析 postgres 的 select  相关的sql

```c
static void
exec_simple_query(const char *query_string)
{
  CommandDest dest = whereToSendOutput;
	MemoryContext oldcontext;
	List	   *parsetree_list;
	ListCell   *parsetree_item;
	bool		save_log_statement_stats = log_statement_stats;
	bool		was_logged = false;
	bool		isTopLevel;
	char		msec_str[32];
  
  ...
  parsetree_list = pg_parse_query(query_string);
  ...
}
```
> 堆栈
```c
(gdb) bt
#0  raw_parser (str=str@entry=0x13bc450 "select id  from test  where id=1;") at parser.c:36
#1  0x00000000007045c5 in pg_parse_query (query_string=query_string@entry=0x13bc450 "select id  from test  where id=1;") at postgres.c:603
#2  0x0000000000704918 in exec_simple_query (query_string=query_string@entry=0x13bc450 "select id  from test  where id=1;") at postgres.c:919
#3  0x0000000000706726 in PostgresMain (argc=<optimized out>, argv=argv@entry=0x1366d00, dbname=0x133cd40 "exampledb", username=<optimized out>)
    at postgres.c:4072
#4  0x000000000069da04 in BackendRun (port=port@entry=0x1362570) at postmaster.c:4342
#5  0x000000000069f9e0 in BackendStartup (port=port@entry=0x1362570) at postmaster.c:4016
#6  0x000000000069fc7e in ServerLoop () at postmaster.c:1721
#7  0x00000000006a0ebb in PostmasterMain (argc=argc@entry=3, argv=argv@entry=0x133abc0) at postmaster.c:1329
#8  0x0000000000620a44 in main (argc=3, argv=0x133abc0) at main.c:228


```
主要的语法解析在函数`raw_parser`中
