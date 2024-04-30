# Mysql Procedure


### 基本语法
#### 创建存储过程
```
Create Procedure name
BEGIN
  执行语句
END
```
#### 更新存储过程
```
Alter Procedure name
BEGIN
  执行语句
END
```
#### 删除存储过程
```
Drop Procedure name
```
### 参数类型
| 参数类型  | 是否返回 |
|-------|------|
| In    | 否    |
| Out   | 是    |
| InOut | 是    |

### 流控制语句
1. BEGIN ... END 语句
2. DECLARE 声明变量
3. SET 赋值语句
4. SELECT ... INTO 为变量赋值 
5. IF...THEN...ENDIF 条件判断语句
6. CASE多条件判断语句
7. LOOP、 LEAVE和 ITERATE：LOOP循环语句，使用LEAVE可以跳出循环，使用ITERATE则可以进入下一次循环
8. REPEAT...UNTIL...END REPEAT：循环语句
9. WHILE...DO...END WHILE：循环语句
### 优缺点
#### 优点
* 一次编译多次使用
* 提升 SQL的执行效率，减少开发工作量
* 安全性强
* 减少网络传输量
### 缺点
* 可移植性差
* 调试困难
* 版本管理困难
* 不适合高并发的场景
