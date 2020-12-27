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