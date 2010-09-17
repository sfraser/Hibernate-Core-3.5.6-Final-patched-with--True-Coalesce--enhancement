# Introduction

This is a patched version of Hibernate 3.5.6-Final. The patch is to enhance the
flushing behavior of Hibernate and cause it to "coalesce" the UPDATE into the 
INSERT even when their were dirty changes made to the entity post INSERT.

What this boils down to is if you merge/persist an entity into a Hibernate session,
and then make changes to that managed object, Hibernate ends up executing two 
DML operations instead of one at flush time - an INSERT and then an UPDATE.

This patch will cause Hibernate to merge the UPDATE into the INSERT, and results
in a single INSERT DML operation at flush time.

For more links on this issue, and why people care, see [these links.](http://delicious.com/sfraser/coalesce)
