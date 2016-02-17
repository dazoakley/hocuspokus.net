---
layout: post
title: Auto-Increment ID's In Oracle
tags:
- mysql
- oracle
- sql
---

This is old news to most people who have been using Oracle for a while, but to me this is something new that I learnt today.  How to have MySQL like "auto-increment" id's for your tables in Oracle. :)  Here's an example...

First we need to create a sequence for the ID's:

{% highlight sql %}
CREATE SEQUENCE "S_COMMENT_ID"
START WITH 1
INCREMENT BY 1
CACHE 10;
{% endhighlight %}

Then we create the table and the required trigger:

{% highlight sql %}
CREATE TABLE "COMMENT" (
  "COMMENT_ID" NUMBER(10,0) NOT NULL,
  "COMMENT_BODY" VARCHAR2(1000),
  "USER" VARCHAR2(100),
  "CREATED_DATE" DATE,
  CONSTRAINT "PK_COMMENT" PRIMARY KEY ("COMMENT_ID")
);

CREATE OR REPLACE TRIGGER "TR_COMMENT_ID"
BEFORE INSERT
ON COMMENT
REFERENCING NEW AS NEW OLD AS OLD
FOR EACH ROW
BEGIN
  if(:new.comment_id is null) then
  SELECT S_COMMENT_ID.nextval
  INTO :new.COMMENT_ID
  FROM dual;
  end if;
END;
/
ALTER TRIGGER "TR_COMMENT_ID" ENABLE;
{% endhighlight %}

And there you go - an auto-incrementing primary key... Damn that's a lot more code to write than a simple
'auto-increment'.

*Edit:* An alternative when using [SQL Developer](http://www.oracle.com/technology/software/products/sql/index.html)
is to create your table, select it, then go to the 'Triggers' tab. In there you will find a useful
'Actions...' button where you can create a new trigger linking your tables primary key to a sequence.
