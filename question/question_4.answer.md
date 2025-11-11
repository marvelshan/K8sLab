1. The following questions concern MySQL. To create user andrew with password passwd123 on localhost one should write:

   建立使用者 andrew，密碼為 passwd123，且主機限制為 localhost，該使用者的建立語法應該是？

   A: CREATE USER 'andrew'@'localhost' IDENTIFIED BY 'passwd123';

   正確。這個會在 mysql.user 資料表中新增一個使用者帳號，並將密碼以雜湊方式儲存

   B: CREATE USER 'andrew'@'localhost' IDENTIFIED WITH 'passwd123';

   IDENTIFIED WITH 用來指定 認證外掛 (authentication plugin)，例如 `CREATE USER 'andrew'@'localhost' IDENTIFIED WITH mysql_native_password BY 'passwd123';`

   C: CREATE USER 'andrew'@'localhost' IDENTIFIED BY PASSWORD 'passwd123';

   這在 MySQL 5.7 以前 可以使用，當時 IDENTIFIED BY PASSWORD 代表你要直接指定一個「加密後的密碼字串（hash）」，而不是明文密碼 `CREATE USER 'andrew'@'localhost' IDENTIFIED BY PASSWORD '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29';` 才是正確用法

   D: CREATE USER 'andrew'@'localhost' SET PASSWORD 'passwd123';

   SET PASSWORD 是另一個獨立命令，用來修改密碼，不能直接接在 CREATE USER 後面使用

2. To allow a particular user to remove a particular table in the current database, one should grant which privilege?

   考 MySQL 權限 (Privileges)，特別是關於「讓某個使用者可以刪除特定資料表」時，應該授予哪一種權限。若想讓某個使用者能夠刪除目前資料庫中的某一張特定 table，應該授與哪一種權限？

   A: DELETE

   DELETE 權限只允許 刪除資料列（rows），而不是刪除整張表

   B: TRUNCATE

   TRUNCATE 是一個 DDL（資料定義語言）操作，不是 DML（資料操作語言）。它會清空表格中的資料，但不刪除表格本身。而且 MySQL 沒有 TRUNCATE 權限這種東西

   C: DROP

   正確。可用於刪除其他物件（如 view、database）。並且 MySQL 允許針對特定資料表授與權限 `GRANT DROP ON mydatabase.mytable TO 'andrew'@'localhost';`

   D: It is not possible to grant a specific privilege only to remove a table in MySQL.

   錯誤。因為 MySQL 確實提供 DROP 權限可以專門讓使用者刪除資料表

   E: It is not possible to specify privileges per table in MySQL.

   錯誤。MySQL 權限可以設定到不同層級：

   - 全伺服器層級（_._）
   - 資料庫層級（database_name.\*）
   - 資料表層級（database_name.table_name）
   - 欄位層級（column-level）

   因此可以針對「特定資料表」設定權限

3. To safely clear a table with foreign key constraints applied to it (together with all the records matched in other tables), one should:

   若一張資料表具有 foreign key constraints，要安全地清空這張表（以及被關聯的其他表中的對應紀錄），應該採取哪一種方式？

   A: Create a trigger ON CASCADE, also removing the records bound by constraints.

   錯誤。`ON CASCADE` 是外鍵的屬性 (foreign key option)，不是「觸發器 (trigger)」，正確做法是在建立外鍵時加上：

   ```sql
   FOREIGN KEY (child_col) REFERENCES parent_table(id) ON DELETE CASCADE
   ```

   B: Disable the foreign key check, TRUNCATE the table and then re-enable foreign key check.

   不安全。雖然這可以「強制清空」表格，但會破壞資料一致性，因為其他表格仍可能保留指向該表的外鍵資料，造成 orphan records，題目中說要「安全地清空（safely clear）」並「連同其他表的相關紀錄一起刪除」，這個方法只是繞過限制，不是安全做法

   ```sql
   SET FOREIGN_KEY_CHECKS=0;
   TRUNCATE TABLE mytable;
   SET FOREIGN_KEY_CHECKS=1;
   ```

   C: Perform a DELETE query on that table without a WHERE clause.

   正確。

   ```sql
   DELETE FROM mytable;
   ```

   這會刪除該表所有資料

   ```sql
   ON DELETE CASCADE
   ```

   則 MySQL 會自動刪除其他表中與這些紀錄相關的資料。這才是「安全地清空（safely）」並「連同其他表關聯紀錄」的方式，同時會維持外鍵約束，不破壞資料完整性

   D: None of the answers above is correct.

   | 指令       | 外鍵檢查 | 是否可觸發 ON DELETE CASCADE | 可回滾 (transaction) | 效能 |
   | ---------- | -------- | ---------------------------- | -------------------- | ---- |
   | `DELETE`   | 會檢查   | ✅ 會觸發                    | ✅ 可回滾            | 慢   |
   | `TRUNCATE` | 不會檢查 | ❌ 不會觸發                  | ❌ 不可回滾          | 快   |

4. If we have tables threads and posts (with foreign keys to the threads table), and we want to get a list of all threads containing more than five posts, we could:

   若有兩個資料表：
   threads：主表（例如討論串）
   posts：貼文表，內含外鍵 thread_id 連到 threads.id

   想查出「包含超過五篇貼文的所有討論串」，應該使用哪些 SQL 子句？

   ```sql
   SELECT t.id, t.title, COUNT(p.id) AS post_count
   FROM threads t
   JOIN posts p ON t.id = p.thread_id
   GROUP BY t.id, t.title
   HAVING COUNT(p.id) > 5;
   ```

   A: Use GROUP BY, JOIN, COUNT and WHERE.

   B: Use GROUP BY, JOIN, COUNT and HAVING.

   正確。

   C: Use JOIN, SUM and WHERE.

   D: Use JOIN, ORDER BY and LIMIT.

5. What does CREATE TABLE table2 SELECT \* FROM table1 do?

   `CREATE TABLE table2 SELECT * FROM table1` 這條語句會做什麼？

   A: Creates a new table called table2 and copies data from table1.

   正確。建立一個新的資料表 table2。根據 SELECT \* FROM table1 的結果，複製資料與欄位結構（schema）。table2 的欄位名稱、資料型別會根據 table1 的查詢結果自動推斷，這種做法也稱為 `CTAS - Create Table As Select`

   table2 不會繼承 table1 的索引 (indexes)、主鍵 (primary key)、外鍵 (foreign keys)、AUTO_INCREMENT 屬性等。它只會複製欄位名稱與資料內容

   B: Creates a new, empty table called table2 with the same structure as table1.

   這裏是錯的，因為這個指令會複製資料，若要只建立結構、不複製資料，應該使用

   ```sql
   CREATE TABLE table2 LIKE table1;
   ```

   C: Creates a view called table2 based on table1.

   建立 view（檢視表）要用

   ```sql
   CREATE VIEW table2 AS SELECT * FROM table1;
   ```

   D: Produces a syntax error.

6. If one creates ENUM('abc', 'xyz', '123', 'aaa', 'def') and selects records in descending order from this enum, then:

   ENUM 在 MySQL 中有一個「隱藏的索引值（internal index）」，排序不是依照字母順序，而是依照宣告時的順序，MySQL 會自動為 ENUM 成員分配內部整數值（index），這邊要注意，索引值是從 1 開始，0 是保留給空值或錯誤

   | ENUM 值 | 內部索引值 |
   | ------- | ---------- |
   | 'abc'   | 1          |
   | 'xyz'   | 2          |
   | '123'   | 3          |
   | 'aaa'   | 4          |
   | 'def'   | 5          |

   A: Records with abc will be first.

   B: Records with xyz will be first.

   C: Records with 123 will be first.

   D: Records with aaa will be first.

   E: Records with def will be first.

   索引值 5 → 降冪時最前面，如果希望按照字母順序排序 ENUM，必須用 CAST 轉為 CHAR

   ```sql
   SELECT * FROM table ORDER BY CAST(enum_column AS CHAR) DESC;
   ```

7. If we have a table that stores point coordinates in 2D with two columns, x and y, and we want to get a non-duplicated list of all points in the table, we should use:

   有一個資料表 points，有兩個欄位 x 和 y，要取得「所有不重複的點」，應該用哪個 SQL 語句？

   A: SELECT DISTINCT [x, y] FROM ...

   錯誤。MySQL 中不能直接在 DISTINCT 裡面寫 [x, y]，括號會被當作一個「單一欄位」處理，會報錯或得到不正確結果，正確寫法是

   ```sql
   SELECT DISTINCT x, y FROM points;
   ```

   B: SELECT DISTINCT x, DISTINCT y FROM ...

   錯誤。DISTINCT 只能寫一次，作用於整個列組合，而不能對每個欄位分別寫 DISTINCT

   C: SELECT x, y FROM ... GROUP BY x, y

   正確。GROUP BY x, y 會將相同的 (x, y) 組合分成一組，只保留一筆 → 等同於去重，功能上等價於

   ```sql
   SELECT DISTINCT x, y
   ```

   D: SELECT DISTINCT (x, y) FROM ...

   正確。雖然選項 D 寫為 SELECT DISTINCT (x, y)，括號在某些情境下可能不必要或被忽略，但其實質意圖（使用 DISTINCT 於多欄位）是正確的

   E: More than one answer is correct.

8. If more than one column is returned by a subquery when using the IN operator in the main query, then what is used as the result of the subquery?

   當 subquery 返回多於一個欄位，並且主查詢使用 IN 運算子時，子查詢的結果會怎麼被處理？

   A: There is a syntax error in such a statement; only one column should be returned by the subquery in this case.

   不完全正確。會報語法錯誤，因為單純 IN 只能接受一欄位的子查詢，但是，SQL 標準允許多欄位 tuple 用法，只要左邊也寫 tuple

   ```sql
   SELECT * FROM table1
   WHERE (col1, col2) IN (SELECT col1, col2 FROM table2);
   ```

   B: Values of the primary key(s).

   錯誤。SQL 並不自動選擇 primary key 值作為比較，IN 的行為只依據「左側欄位與子查詢列值匹配」

   C: Values of the first returned column.

   錯誤。SQL 並不是自動只取第一欄，這樣會破壞多欄位匹配的語意

   D: Values of the last returned column.

   錯誤。也不是自動只取最後一欄，行為同上。

   E: All columns are used as tuples, and one can provide a column names tuple as the left operand of IN, as long as it has the same arity as the returned tuples.

   正確。SQL 標準允許多欄位比較（row constructor / tuple comparison）要求是子查詢返回 tuple、左邊也必須提供相同數量的欄位 tuple，這稱為 row constructor

   ```sql
   SELECT *
   FROM threads t
   WHERE (t.id, t.author_id) IN (SELECT p.thread_id, p.author_id FROM posts p);
   ```

9. If one creates a multicolumn index (col1, col2, col3), then what types of queries will it use?

   如果建立了多欄位索引 (col1, col2, col3)，下列哪種查詢會使用該索引？ 這是關於 multicolumn index / composite index 的使用規則，特別是 leftmost prefix rule

   ### MySQL 索引原則

   - 最左前綴原則 (Leftmost Prefix Rule)：

     多欄位索引 (col1, col2, col3) 可以用於查詢中從最左邊欄位開始的任意前綴。

     可用索引的查詢：

     WHERE col1 = ...

     WHERE col1 = ... AND col2 = ...

     WHERE col1 = ... AND col2 = ... AND col3 = ...

   - 注意：

     不能只使用中間或最後的欄位，如 WHERE col2 = ... 或 WHERE col3 = ...
     → 這種查詢 不會使用索引（除非另有單欄索引）。

   A: SELECT ... WHERE col2 = ... AND col1 = ...

   雖然寫成 col2 = ... AND col1 = ...，但資料庫的最佳化器會識別到 $col1$ 和 $col2$ 都在條件中，它們是索引 $(col1, col2, col3)$ 的最左前綴 $(col1, col2)$，因此可以有效利用索引

   B: SELECT ... WHERE col2 = ...

   這個條件不包含最左欄位 $col1$ 作為查找條件。索引不會被用作高效查找 (Index Seek)

   C: None.

   D: Both.

10. If one wants to select all records with the value NULL from column col, one could use:

- 如果要選取某欄位 col 的所有 NULL 值，哪個查詢是正確的？

  A: WHERE col = NULL

  正確。在 SQL 中，NULL = NULL 不會返回 TRUE，而是 UNKNOWN（NULL），因此這個條件永遠不會匹配任何行，正確的用法為用 `IS NULL` 或 `<=> NULL`

  B: WHERE col <=> NULL

  正確。`<=>` 是 MySQL 的 NULL-safe 等於運算子，對 NULL 值 <=> 會返回 TRUE，所以可以正確匹配 NULL

  ```sql
  SELECT * FROM table WHERE col <=> NULL;
  ```

  C: WHERE col IFNULL(col)

  錯誤。IFNULL(col, value) 是函數，不是條件運算子

  D: WHERE (NOT col) IS NOT NULL

  錯誤。(NOT col) 會在布林上下文轉換，NULL 會被當作 UNKNOWN → (NOT col) 還是 NULL

  E: More than one answer is correct.

  正確的 SQL 查 NULL

  ```sql
   SELECT * FROM table WHERE col IS NULL;
  ```

  ```sql
  SELECT * FROM table WHERE col <=> NULL;
  ```
