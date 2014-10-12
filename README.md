# Convergence

Convergence is a Database Schema management tools.
Currently, This tools is support only MySQL.

It defines DB Schema using Convergence DSL(like Rails DSL).
For more information about Convergence DSL, See below 'Detail About Convergence DSL'.


[![Build Status](https://travis-ci.org/nishio-dens/convergence.svg?branch=master)](https://travis-ci.org/nishio-dens/convergence)

## Installation

Add this line to your application's Gemfile:

```
gem 'convergence'
```

and then execute

```
bundle
```

Or install it yourself as:

```
gem install convergence
```

## What's this?

```
$ mysql -u root -e 'create database example_database;'
$ cat database.yml

adapter: mysql
database: example_database
host: 127.0.0.1
username: root
password:

$ cat example.schema

create_table 'test_tables' do |t|
  t.int :id, primary_key: true, extra: 'auto_increment'
  t.varchar :name, limit: 100, null: true
  t.datetime :created_at
  t.datetime :updated_at

  t.index :name
end

$ convergence -c database.yml -i example.schema --dryrun

#
#
# CREATE TABLE `test_tables` (
#   `id` int(11) NOT NULL AUTO_INCREMENT,
#   `name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
#   `created_at` datetime NOT NULL,
#   `updated_at` datetime NOT NULL,
#   PRIMARY KEY (`id`),
#   KEY `index_test_tables_on_name` (`name`)
# ) ENGINE=InnoDB ROW_FORMAT=Compact DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci;

$ convergence -c database.yml -i example.schema --apply

SET FOREIGN_KEY_CHECKS=0;
  --> 0.000794s
CREATE TABLE `test_tables` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `index_test_tables_on_name` (`name`)
) ENGINE=InnoDB ROW_FORMAT=Compact DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci;
  --> 0.01485s
SET FOREIGN_KEY_CHECKS=1;
  --> 0.000115s

$ cat changed_example.schema

create_table 'test_tables', comment: 'Table Comment Test', engine: 'MyISAM' do |t|
  t.int :id, primary_key: true, extra: 'auto_increment'
  t.varchar :name, limit: 100, null: true
  t.datetime :created_at
  t.datetime :posted_at
end

$ convergence -c database.yml -i changed_example.schema  --dryrun

# ALTER TABLE `test_tables` DROP COLUMN `updated_at`;
# ALTER TABLE `test_tables` ADD COLUMN `posted_at` datetime NOT NULL AFTER `created_at`;
# DROP INDEX `index_test_tables_on_name` ON `test_tables`;
# ALTER TABLE `test_tables` ENGINE=MyISAM COMMENT='Table Comment Test';

$ convergence -c database.yml -i changed_example.schema  --apply
SET FOREIGN_KEY_CHECKS=0;
  --> 0.000847s
ALTER TABLE `test_tables` DROP COLUMN `updated_at`;
  --> 0.034026s
ALTER TABLE `test_tables` ADD COLUMN `posted_at` datetime NOT NULL AFTER `created_at`;
  --> 0.025328s
DROP INDEX `index_test_tables_on_name` ON `test_tables`;
  --> 0.011465s
ALTER TABLE `test_tables` ENGINE=MyISAM COMMENT='Table Comment Test';
  --> 0.01115s
SET FOREIGN_KEY_CHECKS=1;
  --> 0.000147s

$ mysql -u root example_database -e 'show create table test_tables\G'

*************************** 1. row ***************************
       Table: test_tables
Create Table: CREATE TABLE `test_tables` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `created_at` datetime NOT NULL,
  `posted_at` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='Table Comment Test'
```

## Usage

```
Usage: convergence [options]
    -v, --version
    -c, --config       Database Yaml Setting
    -d, --diff         DSL1,DSL2
    -e, --export       export db schema to dsl
    -i, --input        Input DSL
        --dryrun
        --apply        execute sql to your database
    -h, --help         Display this help message.
```

### DB Config

You need to make database.yml to access your database.

```
$ cat database.yml
dapter: mysql
database: convergence_test
host: 127.0.0.1
username: root
password:
```

### Export Your DB Schema

First, you need to create database.yml.
And then, execute command like below.

```
$ convergence -c database.yml --export
```

Export DSL like this.

```
create_table "authors", collate: "utf8_general_ci" do |t|
  t.int "id", primary_key: true, extra: "auto_increment"
  t.varchar "name", limit: 110
  t.datetime "created_at", null: true
  t.datetime "updated_at", null: true

  t.index "created_at", name: "index_authors_on_created_at"
end

create_table "papers", collate: "utf8_general_ci", comment: "Paper" do |t|
  t.int "id", primary_key: true, extra: "auto_increment"
  t.varchar "title1", limit: 300, comment: "Title 1"
  t.varchar "title2", limit: 300, comment: "Title 2"
  t.text "description", null: true, comment: "Description"
end

create_table "paper_authors", collate: "utf8_general_ci", comment: "Paper Author Relation" do |t|
  t.int "id", primary_key: true, extra: "auto_increment"
  t.int "paper_id", comment: "Paper id"
  t.int "author_id", comment: "Paper author id"

  t.foreign_key "author_id", reference: "authors", reference_column: "id", name: "paper_authors_author_id_fk"
  t.foreign_key "paper_id", reference: "papers", reference_column: "id", name: "paper_authors_paper_id_fk"
end
```

## Detail About Convergence DSL
