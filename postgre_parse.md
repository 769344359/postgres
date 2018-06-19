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

编译表名

```
(gdb) bt
#0  transformTableEntry (pstate=0x1f7dc80, r=0x1f7d8a8) at parse_clause.c:435
#1  0x00000000005c0432 in transformFromClauseItem (pstate=0x1f7dc80, n=0x1f7d8a8, top_rte=0x7ffefe615b00, top_rti=0x7ffefe615afc, namespace=0x7ffefe615b08) at parse_clause.c:1121
#2  0x00000000005be59a in transformFromClause (pstate=0x1f7dc80, frmList=0x1f7d918) at parse_clause.c:139
#3  0x0000000000585c1e in transformSelectStmt (pstate=0x1f7dc80, stmt=0x1f7daf0) at analyze.c:1212
#4  0x000000000058430b in transformStmt (pstate=0x1f7dc80, parseTree=0x1f7daf0) at analyze.c:301
#5  0x00000000005841e3 in transformOptionalSelectInto (pstate=0x1f7dc80, parseTree=0x1f7daf0) at analyze.c:246
#6  0x00000000005840de in transformTopLevelStmt (pstate=0x1f7dc80, parseTree=0x1f7dc00) at analyze.c:196
#7  0x0000000000583f56 in parse_analyze (parseTree=0x1f7dc00, sourceText=0x1f7cd70 "select * from nothing where key=1;", paramTypes=0x0, numParams=0, queryEnv=0x0) at analyze.c:116
#8  0x000000000086babf in pg_analyze_and_rewrite (parsetree=0x1f7dc00, query_string=0x1f7cd70 "select * from nothing where key=1;", paramTypes=0x0, numParams=0, queryEnv=0x0) at postgres.c:666
#9  0x000000000086c106 in exec_simple_query (query_string=0x1f7cd70 "select * from nothing where key=1;") at postgres.c:1047
#10 0x000000000087062f in PostgresMain (argc=1, argv=0x1fa65a8, dbname=0x1fa6490 "test", username=0x1fa6478 "vagrant") at postgres.c:4153
#11 0x00000000007db551 in BackendRun (port=0x1f9e470) at postmaster.c:4361
#12 0x00000000007dac72 in BackendStartup (port=0x1f9e470) at postmaster.c:4033
#13 0x00000000007d737e in ServerLoop () at postmaster.c:1706
#14 0x00000000007d6be6 in PostmasterMain (argc=3, argv=0x1f779e0) at postmaster.c:1379
#15 0x000000000070938e in main (argc=3, argv=0x1f779e0) at main.c:228
```
