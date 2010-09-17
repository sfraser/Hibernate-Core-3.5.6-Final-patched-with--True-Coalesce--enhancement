# Introduction

This is a patched version of Hibernate 3.5.6-Final. The patch is to enhance the
flushing behavior of Hibernate and cause it to "coalesce" the UPDATE into the 
INSERT even when there were dirty changes made to the entity post INSERT.

What this boils down to is if you merge/persist an entity into a Hibernate session,
and then make changes to that managed object, Hibernate ends up executing two 
DML operations instead of one at flush time - an INSERT and then an UPDATE.

This patch will cause Hibernate to merge the UPDATE into the INSERT, and results
in a single INSERT DML operation at flush time.

For more links on this issue, and why people care, see [these links.](http://delicious.com/sfraser/coalesce)

The files of interest modified in this patch are:

[EntityInsertAction](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-3)
[ActionQueue](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-4)
[DefaultFlushEntityEventListener](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-5)