---
layout: post
title: Log/Audit Tables In Oracle
tags:
- oracle
- sql
---

One of the useful things that i've been asked to set-up lately is automatic logging of changes to several
of our database tables.

My first thought was to do this in Perl (as the rest of the system is in Perl), but this would mean
adding extra methods and calls in the Perl code to update the database (both the original tables and the
new log tables). That seemed like a solution - a pain in the arse to implement, but a solution.

Thankfully one of the helpful chaps in my department suggested doing it all in the database with triggers
as this is quite common in banks and the like. What a damn fine idea! Only a little SQL to write and no
extra Perl.

Here's an example...

First, here's the table that we want to create an audit log for:

{% highlight sql %}
CREATE TABLE "COMMENT" (
  "COMMENT_ID" NUMBER(10,0) NOT NULL,
  "COMMENT_BODY" VARCHAR2(1000),
  "USER" VARCHAR2(100),
  "CREATED_DATE" DATE,
  CONSTRAINT "PK_COMMENT" PRIMARY KEY ("COMMENT_ID")
);
{% endhighlight %}

Now, we create the audit table - the audit table is exactly the same as the table that we wish to keep a
log of, except for one extra column, "AUDIT_DATE", this will keep record the date/time of when an change
occurred on the original table.

{% highlight sql %}
CREATE TABLE "COMMENT_AUDIT" (
  "COMMENT_ID" NUMBER(10,0) NOT NULL,
  "COMMENT_BODY" VARCHAR2(1000),
  "USER" VARCHAR2(100),
  "CREATED_DATE" DATE,
  "AUDIT_DATE" DATE DEFAULT SYSTIMESTAMP NOT NULL
);
{% endhighlight %}

Finally, we create the trigger to keep a log of changes to our original table.

{% highlight sql %}
CREATE OR REPLACE TRIGGER "TA_COMMENT"
before update or delete on COMMENT for each row
begin
if deleting or updating then
  insert into comment_audit(
    COMMENT_ID,
    COMMENT_BODY,
    USER,
    CREATED_DATE
  )
  values (
    :old.COMMENT_ID,
    :old.COMMENT_BODY,
    :old.USER,
    :old.CREATED_DATE
  );
end if;
end;
/
ALTER TRIGGER "TA_COMMENT" ENABLE;
{% endhighlight %}

Now whenever a delete or update action is performed on the "COMMENT" table, a record of the old values
and the date/time at which they were changed is recorded in the audit table.
