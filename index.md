---
layout: default
---

[![Gem Version](https://badge.fury.io/rb/ridgepole.svg)](http://badge.fury.io/rb/ridgepole)
[![Build Status](https://travis-ci.org/winebarrel/ridgepole.svg?branch=master)](https://travis-ci.org/winebarrel/ridgepole)
[![Coverage Status](https://img.shields.io/coveralls/winebarrel/ridgepole.svg)](https://coveralls.io/r/winebarrel/ridgepole?branch=master)

**Notice**

* `>= 0.4.8`
  * `activerecord-mysql-unsigned` is now optional. Please pass `--enable-mysql-unsigned` after you install [activerecord-mysql-unsigned](https://github.com/waka/activerecord-mysql-unsigned) if you want to use.
  * Please pass `--enable-foreigner` after you install [foreigner](https://github.com/matthuhiggins/foreigner) if you want to use the foreign key.
* `>= 0.4.11`
  * Add `--enable-mysql-pkdump` option.
* `>= 0.4.12`
  * Fix `activerecord-mysql-unsigned` version: `~> 0.2.0`
* `>= 0.5.0`
  * Fix `activerecord-mysql-unsigned` version: `~> 0.3.1`
* `>= 0.5.1`
  * Add `--enable-migration-comments` option ([migration_comments](https://github.com/pinnymz/migration_comments) is required)
  * Fix rails version `< 4.2.0`
* `>= 0.5.2`
  * Add `--enable-mysql-awesome` option ([activerecord-mysql-awesome](https://github.com/kamipo/activerecord-mysql-awesome) is required `>= 0.0.3`)
  * It is not possible to enable both `--enable-mysql-awesome` and --enable-migration-comments`, `--enable-mysql-awesome` and --enable-mysql-unsigned`, `--enable-mysql-awesome` and --enable-mysql-pkdump`
  * Fix foreigner version `<= 1.7.1`

## Installation

Add this line to your application's Gemfile:

    gem 'ridgepole'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install ridgepole

## Help
{% highlight sh %}
Usage: ridgepole [options]
    -c, --config CONF_OR_FILE
    -E, --env ENVIRONMENT
    -a, --apply
    -m, --merge
    -f, --file FILE
        --dry-run
        --table-options OPTIONS
        --bulk-change
        --default-int-limit LIMIT
        --pre-query QUERY
        --post-query QUERY
    -e, --export
        --split
        --split-with-dir
    -d, --diff DSL1 DSL2
        --reverse
        --with-apply
    -o, --output FILE
    -t, --tables TABLES
        --ignore-tables TABLES
        --enable-mysql-unsigned
        --enable-mysql-pkdump
        --enable-foreigner
        --enable-migration-comments
        --enable-mysql-awesome
        --mysql-awesome-unsigned-pk
        --normalize-mysql-float
        --dump-without-table-options
    -r, --require LIBS
        --log-file LOG_FILE
        --verbose
        --debug
    -v, --version
{% endhighlight %}

## Usage

{% highlight sh %}
$ git init
Initialized empty Git repository in ...

$ cat config.yml
adapter: mysql2
encoding: utf8
database: blog
username: root

$ ridgepole -c config.yml --export -o Schemafile # or `ridgepole -c '{adapter: mysql2, database: blog}' ...`
Export Schema to `Schemafile`

$ cat Schemafile
create_table "articles", force: true do |t|
  t.string   "title"
  t.text     "text"
  t.datetime "created_at"
  t.datetime "updated_at"
end

$ git add .
$ git commit -m 'first commit'  -a
[master (root-commit) a6c2d31] first commit
 2 files changed, 10 insertions(+)
 create mode 100644 Schemafile
 create mode 100644 config.yml

$ vi Schemafile
$ git diff
diff --git a/Schemafile b/Schemafile
index f5848b9..c266fed 100644
--- a/Schemafile
+++ b/Schemafile
@@ -1,6 +1,7 @@
 create_table "articles", force: true do |t|
   t.string   "title"
   t.text     "text"
+  t.text     "author"
   t.datetime "created_at"
   t.datetime "updated_at"
 end

$ ridgepole -c config.yml --apply --dry-run
Apply `Schemafile` (dry-run)
add_column("articles", "author", :text, {:after=>"text"})

# ALTER TABLE `articles` ADD `author` text AFTER `text`

$ ridgepole -c config.yml --apply
Apply `Schemafile`
-- add_column("articles", "author", :text, {:after=>"text"})
   -> 0.0202s
{% endhighlight %}

## Rename
{% highlight ruby %}
create_table "articles", force: true do |t|
  t.string   "title"
  t.text     "desc", renamed_from: "text"
  t.text     "author"
  t.datetime "created_at"
  t.datetime "updated_at"
end

create_table "user_comments", force: true, renamed_from: "comments" do |t|
  t.string   "commenter"
  t.text     "body"
  t.integer  "article_id"
  t.datetime "created_at"
  t.datetime "updated_at"
end
{% endhighlight %}

## Foreign Key
You can use the foreign key by passing `--enable-foreigner` ([foreigner](https://github.com/matthuhiggins/foreigner) is required)

{% highlight ruby %}
create_table "parent", force: true do |t|
end

create_table "child", id: false, force: true do |t|
  t.integer "id"
  t.integer "parent_id"
end

add_index "child", ["parent_id"], name: "par_ind", using: :btree

add_foreign_key "child", "parent", name: "child_ibfk_1", dependent: :delete
{% endhighlight %}

## Comment
You can use the table/column comment by passing `--enable-migration-comments` ([migration_comments](https://github.com/pinnymz/migration_comments) is required)

{% highlight ruby %}
create_table "articles", force: true, comment: "table comment" do |t|
  t.string   "title", comment: "column comment"
  t.text     "text"
  t.datetime "created_at"
  t.datetime "updated_at"
end
{% endhighlight %}

## Collation
You can use the column collation by passing `--enable-mysql-awesome` ([activerecord-mysql-awesome](https://github.com/kamipo/activerecord-mysql-awesome) is required)

{% highlight ruby %}
create_table "articles", force: true, options: "ENGINE=InnoDB DEFAULT CHARSET=utf8" do |t|
  t.string   "title",                    collation: "ascii_bin"
  t.text     "text",       null: false,  collation: "utf8mb4_bin"
  t.datetime "created_at"
  t.datetime "updated_at"
end
{% endhighlight %}

## bigint support
Export of `bigint` PK is enabled by passing `--enable-mysql-pkdump` ([activerecord-mysql-pkdump](https://github.com/winebarrel/activerecord-mysql-pkdump) is required)

{% highlight ruby %}
create_table "books", id: "bigint(20) PRIMARY KEY auto_increment", force: true do |t|
  t.string   "title",      null: false
  t.integer  "author_id",  null: false
  t.datetime "created_at"
  t.datetime "updated_at"
end
{% endhighlight %}

If you use `--enable-mysql-awesome`:

{% highlight ruby %}
create_table "books", id: :bigint, force: true do |t|
  ...
end
{% endhighlight %}

## Execute
{% highlight ruby %}
create_table "authors", force: true do |t|
  t.string "name", null: false
end

create_table "books", force: true do |t|
  t.string  "title",                     null: false
  t.integer "author_id", unsigned: true, null: false
end

add_index "books", ["author_id"], name: "idx_author_id", using: :btree

execute("ALTER TABLE books ADD CONSTRAINT fk_author FOREIGN KEY (author_id) REFERENCES authors (id)") do |c|
  # Execute SQL only if there is no foreign key
  c.raw_connection.query("SELECT 1 FROM information_schema.key_column_usage WHERE TABLE_SCHEMA = 'bookshelf' AND CONSTRAINT_NAME = 'fk_author' LIMIT 1").each.length.zero?
end
{% endhighlight %}

## Diff
{% highlight sh %}
$ ridgepole --diff file1.schema file2.schema
add_column("articles", "author", :text, {:after=>"title"})
rename_column("articles", "text", "desc")

# You can apply to the database the difference:
# $ ridgepole -c config.yml --diff file1.schema file2.schema --with-apply
{% endhighlight %}

You can also compare databases and files.

{% highlight sh %}
$ ridgepole --diff config.yml file1.schema
remove_column("articles", "author")
{% endhighlight %}

### Reverse diff
{% highlight sh %}
$ cat file1.schema
create_table "articles", force: true do |t|
  t.string   "title"
  t.text     "text"
  t.datetime "created_at"
  t.datetime "updated_at"
end

$ cat file2.schema
create_table "articles", force: true do |t|
  t.string   "title"
  t.text     "desc", renamed_from: "text"
  t.text     "author"
  t.datetime "created_at"
  t.datetime "updated_at"
end

$ ridgepole --diff file1.schema file2.schema
add_column("articles", "author", :text, {:after=>"title"})
rename_column("articles", "text", "desc")

$ ridgepole --diff file1.schema file2.schema --reverse
rename_column("articles", "desc", "text")
remove_column("articles", "author")
{% endhighlight %}

## Demo

* [asciinema.org/a/9349](https://asciinema.org/a/9349)
* [asciinema.org/a/11788](https://asciinema.org/a/11788)

## Example project

* [github.com/winebarrel/ridgepole-example](https://github.com/winebarrel/ridgepole-example)
  * [github.com/winebarrel/ridgepole-example/pull/1](https://github.com/winebarrel/ridgepole-example/pull/1)
  * [github.com/winebarrel/ridgepole-example/pull/2](https://github.com/winebarrel/ridgepole-example/pull/2)

## Similar tools
* [Codenize.tools](http://codenize.tools/)