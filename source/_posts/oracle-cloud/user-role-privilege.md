---
date: 2019-08-23 00:00:00
title: oracle cloud 用户角色权限表关系（废弃）
author: Joden_He
tags: 
  - Oracle Cloud
  - Role-Privileges
categories: 
  - Oracle Cloud
description: oracle cloud 中用户角色权限之间的表关系
---

> 根据 sr 回复，ase相关表只是给 oracle 内部使用，表的数据并不准确，后期如果有找到相关表再做更新吧，大家看到这个直接忽略吧，打扰了

## Cloud 查看用户角色权限

![Security Console](/images/oracle_cloud/20190823222822.png)

![user role privilege](/images/oracle_cloud/20190823223248.png)



## 表关系

主要涉及表：

> 前缀ASE开头的

`ASE_USER_VL`  用户表

`ASE_USER_ROLE_MBR`  用户角色分配表

`ASE_ROLE_VL`  角色表

`ASE_PRIVILEGE_VL`  权限表

`ASE_PRIV_ROLE_MBR`  权限角色表

`ASE_ROLE_ROLE_MBR`  角色父子关系表

## sql 查询

### 角色父子关系

```sql
SELECT t.parent_role_id,
         pr.code          AS parent_role_code,
         pr.role_name     AS parent_role_name,
         t.leaf,
         t.ase_role_id,
         r.code           AS role_code,
         r.role_name
    FROM (SELECT parent_role_id,
                 connect_by_isleaf leaf,
                 connect_by_root(child_role_id) ase_role_id
            FROM ase_role_role_mbr
          CONNECT BY PRIOR ase_role_role_mbr.parent_role_id = ase_role_role_mbr.child_role_id) t,
         ase_role_vl pr,
         ase_role_vl r
   WHERE t.parent_role_id = pr.role_id(+)
     AND t.ase_role_id = r.role_id

```

### 用户角色查询

```sql
WITH
-- 角色父子关系
role_role_mbr_ AS
 (SELECT t.parent_role_id,
         pr.code          AS parent_role_code,
         pr.role_name     AS parent_role_name,
         t.leaf,
         t.ase_role_id,
         r.code           AS role_code,
         r.role_name
    FROM (SELECT parent_role_id,
                 connect_by_isleaf leaf,
                 connect_by_root(child_role_id) ase_role_id
            FROM ase_role_role_mbr
          CONNECT BY PRIOR ase_role_role_mbr.parent_role_id = ase_role_role_mbr.child_role_id) t,
         ase_role_vl pr,
         ase_role_vl r
   WHERE t.parent_role_id = pr.role_id(+)
     AND t.ase_role_id = r.role_id),

-- 用户角色
user_role_ AS
 (SELECT u.user_login,
         u.user_display_name,
         -- 与当前日期比较判断是否有效
         decode((SELECT 1
                  FROM dual
                 WHERE (u.effective_start_date <= SYSDATE OR u.effective_start_date IS NULL)
                   AND (u.effective_end_date >= SYSDATE OR u.effective_end_date IS NULL)
                      -- 使用suspend 判断是否失效
                   AND EXISTS (SELECT 1
                          FROM per_users pu
                         WHERE pu.user_guid = u.user_guid
                           AND pu.suspended = 'N')),
                1,
                'Active',
                'Inactive') AS is_user_active,
         u.creation_date AS user_creation_date,
         u.last_update_date AS user_last_update_date,
         r.role_id,
         r.code AS role_code,
         r.role_name,
         -- 与当前日期比较判断是否有效
         decode((SELECT 1
                  FROM dual
                 WHERE (r.effective_start_date <= SYSDATE OR r.effective_start_date IS NULL)
                   AND (r.effective_end_date >= SYSDATE OR r.effective_end_date IS NULL)),
                1,
                'Active',
                'Inactive') AS is_role_active,
         r.creation_date AS role_creation_date,
         r.last_update_date AS role_last_update_date,
         -- 与当前日期比较判断是否有效
         decode((SELECT 1
                  FROM dual
                 WHERE (ur.effective_start_date <= SYSDATE OR ur.effective_start_date IS NULL)
                   AND (ur.effective_end_date >= SYSDATE OR ur.effective_end_date IS NULL)),
                1,
                'Active',
                'Inactive') AS is_user_role_active,
         ur.creation_date AS user_role_creation_date,
         ur.last_update_date AS user_role_last_update_date
    FROM fusion.ase_user_vl       u,
         fusion.ase_user_role_mbr ur,
         fusion.ase_role_vl       r
   WHERE u.user_id = ur.user_id
     AND r.role_id = ur.role_id)

-- 主查询
SELECT t.user_login,
       t.user_display_name,
       t.is_user_active,
       t.user_creation_date,
       t.user_last_update_date,
       t.role_code,
       t.role_name,
       rr.parent_role_code,
       rr.parent_role_name,
       t.is_role_active,
       t.role_creation_date,
       t.role_last_update_date,
       t.is_user_role_active,
       t.user_role_creation_date,
       t.user_role_last_update_date
  FROM user_role_     t,
       role_role_mbr_ rr
 WHERE t.role_id = rr.ase_role_id(+)
      /* 参数 */
      -- 用户登陆名
   AND (t.user_login = :p_user OR :p_user IS NULL)
      -- 角色编码，包含父角色
   AND (:p_role_code IS NULL OR t.role_code = :p_role_code OR rr.parent_role_code = :p_role_code)
      -- 用户是否有效
   AND (t.is_user_active = :p_user_active OR :p_user_active IS NULL)
      -- 角色是否有效
   AND (t.is_role_active = :p_role_active OR :p_role_active IS NULL)
      -- 用户角色分配是否有效
   AND (t.is_user_role_active = :p_user_role_active OR :p_user_role_active IS NULL)
      -- 用户创建时间
   AND (t.user_creation_date >= :p_user_creation_date_from OR :p_user_creation_date_from IS NULL)
   AND (t.user_creation_date <= :p_user_creation_date_to OR :p_user_creation_date_to IS NULL)
      -- 角色创建时间
   AND (t.role_creation_date >= :p_role_creation_date_from OR :p_role_creation_date_from IS NULL)
   AND (t.role_creation_date <= :p_role_creation_date_to OR :p_role_creation_date_to IS NULL)
      -- 用户角色创建时间
   AND (t.user_role_creation_date >= :p_ur_creation_date_from OR :p_ur_creation_date_from IS NULL)
   AND (t.user_role_creation_date <= :p_ur_creation_date_to OR :p_ur_creation_date_to IS NULL)


```



## 参考

[https://community.oracle.com/thread/3924763?parent=MOSC_EXTERNAL&sourceId=MOSC&id=3924763](https://community.oracle.com/thread/3924763?parent=MOSC_EXTERNAL&sourceId=MOSC&id=3924763)

![参考](/images/oracle_cloud/20190823223516.png) hen