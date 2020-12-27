# 一、pg_namespace属性表
## 1.nspname
  数据类型NameData char[64];存放命名空间名称
## 2.nspowner
  Oid,命名空间所有者
## 3.nspacl
  aclitem类型的变长数组，存放命名空间的访问权限
# 二、pg_namespace.h
```
 #ifndef PG_NAMESPACE_H
 #define PG_NAMESPACE_H
 
 #include "catalog/genbki.h"
 
 /* ----------------------------------------------------------------
  *		pg_namespace definition.
  *
  *		cpp turns this into typedef struct FormData_pg_namespace
  *
  *	nspname				name of the namespace
  *	nspowner			owner (creator) of the namespace
  *	nspacl				access privilege list
  * ----------------------------------------------------------------
  */
 #define NamespaceRelationId  2615
 
 CATALOG(pg_namespace,2615)
 {
 	NameData	nspname;
 	Oid			nspowner;
 
 #ifdef CATALOG_VARLEN			/* variable-length fields start here */
 	aclitem		nspacl[1];
 #endif
 } FormData_pg_namespace;
 
 /* ----------------
  *		Form_pg_namespace corresponds to a pointer to a tuple with
  *		the format of pg_namespace relation.
  * ----------------
  */
 typedef FormData_pg_namespace *Form_pg_namespace;
 
 /* ----------------
  *		compiler constants for pg_namespace
  * ----------------
  */
 
 #define Natts_pg_namespace				3
 #define Anum_pg_namespace_nspname		1
 #define Anum_pg_namespace_nspowner		2
 #define Anum_pg_namespace_nspacl		3
 
 
 /* ----------------
  * initial contents of pg_namespace
  * ---------------
  */
 
 DATA(insert OID = 11 ( "pg_catalog" PGUID _null_ ));
 DESCR("system catalog schema");
 #define PG_CATALOG_NAMESPACE 11
 DATA(insert OID = 99 ( "pg_toast" PGUID _null_ ));
 DESCR("reserved schema for TOAST tables");
 #define PG_TOAST_NAMESPACE 99
 DATA(insert OID = 2200 ( "public" PGUID _null_ ));
 DESCR("standard public schema");
 #define PG_PUBLIC_NAMESPACE 2200
 
 
 /*
  * prototypes for functions in pg_namespace.c
  */
 extern Oid	NamespaceCreate(const char *nspName, Oid ownerId, bool isTemp);
 
 #endif							/* PG_NAMESPACE_H */
```
# 三、pg_namespce.c
```
#include "postgres.h"

#include "access/heapam.h"
#include "access/htup_details.h"
#include "catalog/dependency.h"
#include "catalog/indexing.h"
#include "catalog/objectaccess.h"
#include "catalog/pg_namespace.h"
#include "utils/builtins.h"
#include "utils/rel.h"
#include "utils/syscache.h"


/*
* 创建命名空间*
*/
Oid
NamespaceCreate(const char *nspName, Oid ownerId, bool isTemp)
{
	Relation	nspdesc;
	HeapTuple	tup;
	Oid			nspoid;
	bool		nulls[Natts_pg_namespace]; //为空对应缺省值
	Datum		values[Natts_pg_namespace];//元组字段对应的值
	NameData	nname;
	TupleDesc	tupDesc;
	ObjectAddress myself;
	int			i;
	Acl		   *nspacl;

	/* sanity checks */
	if (!nspName)
		elog(ERROR, "no namespace name supplied");

	/* make sure there is no existing namespace of same name  检查名称不能重复*/
	if (SearchSysCacheExists1(NAMESPACENAME, PointerGetDatum(nspName)))
		ereport(ERROR,
				(errcode(ERRCODE_DUPLICATE_SCHEMA),
				 errmsg("schema \"%s\" already exists", nspName)));

	
    if (!isTemp)//其他表空间需要权限信息
		nspacl = get_user_default_acl(ACL_OBJECT_NAMESPACE, ownerId,
									  InvalidOid);
	else//临时表空间后台操作不需要权限信息
		nspacl = NULL;

	/* initialize nulls and values  初始化*/
	for (i = 0; i < Natts_pg_namespace; i++)
	{
		nulls[i] = false;
		values[i] = (Datum) NULL;
	}
	namestrcpy(&nname, nspName);
	values[Anum_pg_namespace_nspname - 1] = NameGetDatum(&nname);
	values[Anum_pg_namespace_nspowner - 1] = ObjectIdGetDatum(ownerId);
	if (nspacl != NULL)
		values[Anum_pg_namespace_nspacl - 1] = PointerGetDatum(nspacl);
	else
		nulls[Anum_pg_namespace_nspacl - 1] = true;

    //打开关系表pg_namespace
	nspdesc = heap_open(NamespaceRelationId, RowExclusiveLock);
	tupDesc = nspdesc->rd_att;

    //构造元组
	tup = heap_form_tuple(tupDesc, values, nulls);
    //增加元组
	nspoid = CatalogTupleInsert(nspdesc, tup);
	Assert(OidIsValid(nspoid));
    //关闭关系表
	heap_close(nspdesc, RowExclusiveLock);

	/* Record dependencies */
	myself.classId = NamespaceRelationId;
	myself.objectId = nspoid;
	myself.objectSubId = 0;

	/* dependency on owner 记录pg_shdepend表*/
	recordDependencyOnOwner(NamespaceRelationId, nspoid, ownerId);

	/* dependences on roles mentioned in default ACL */
	recordDependencyOnNewAcl(NamespaceRelationId, nspoid, 0, ownerId, nspacl);

	/* dependency on extension ... but not for magic temp schemas */
	if (!isTemp)
		recordDependencyOnCurrentExtension(&myself, false);

	/* Post creation hook for new schema */
	InvokeObjectPostCreateHook(NamespaceRelationId, nspoid, 0);

	return nspoid;
}

```