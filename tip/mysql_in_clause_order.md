# MySql in절에 sort ordering 
간혹 쿼리를 할 때, 한 번에 select하기 위해 in절을 이용하게 된다.
다만 이 in절이라는 게 순서를 보장하지 않는다.
optimizer에 plan에 따라서 움직이겠지만, 결국은 해당 값이 있는지만 체크하기 때문에
실제로 in절 내의 순서는 내가 List로 전달했다 하더라도 쿼리 결과에 별 영향을 미치지 않는다.

그런데, 간혹 내가 in을 했던 순서가 보장되어 결과가 나왔으면 하는 경우가 있다.
mysql 5.7버전 이후에 제공하는 Field()를 사용하면 된다.

http://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_field

> FIELD(str,str1,str2,str3,...)

> Returns the index (position) of str in the str1, str2, str3, ... list. Returns 0 if str is not found.

> If all arguments to FIELD() are strings, all arguments are compared as strings. If all arguments are numbers, they are compared as numbers. Otherwise, the arguments are compared as double.

> If str is NULL, the return value is 0 because NULL fails equality comparison with any value. FIELD() is the complement of ELT().

 
> mysql> SELECT FIELD('ej', 'Hej', 'ej', 'Heja', 'hej', 'foo');
        -> 2

> mysql> SELECT FIELD('fo', 'Hej', 'ej', 'Heja', 'hej', 'foo');
        -> 0


`field_idx IN (3, 2, 5, 7, 8, 1) ODER BY FIELD(field_idx, 3, 2, 5, 7, 8, 1)` 라고 하면 field_idx는 1, 2, 3, 5, 7, 8을 가져오되, 순서까지 보장해서 3, 2, 5, 7, 8, 1로 select하게 된다.

ref. http://stackoverflow.com/questions/1332434/mysql-order-by-in
