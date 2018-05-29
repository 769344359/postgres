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
