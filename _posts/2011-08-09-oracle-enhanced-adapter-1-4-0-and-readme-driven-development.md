---
layout: post
title: Oracle enhanced adapter 1.4.0 and Readme Driven Development
tags: oracle_enhanced ruby rails oracle
---

I just released [Oracle enhanced adapter](http://github.com/rsim/oracle-enhanced) version 1.4.0 and here is the summary of main changes.

### Rails 3.1 support

Oracle enhanced adapter GitHub version was working with Rails 3.1 betas and release candidate versions already but it was not explicitly stated anywhere that you should use git version with Rails 3.1. Therefore I am releasing new version 1.4.0 which is passing all tests with latest Rails 3.1 release candidate. As I [wrote before](http://blog.rayapps.com/2011/01/05/oracle-enhanced-adapter-1-3-2-is-released/) main changes in ActiveRecord 3.1 are that it using prepared statement cache and using bind variables in many statements (currently in select by primary key, insert and delete statements) which result in better performance and better database resources usage.

To follow up how Oracle enhanced adapter is working with different Rails versions and different Ruby implementations I have set up [Continuous Integration server](http://ci.rayapps.com/job/oracle_enhanced/) to run tests on different version combinations. At the moment of writing everything was green :)

### Other bug fixes and improvements

Main fixes and improvements in this version are the following:

* On JRuby I switched from using old `ojdbc14.jar` JDBC driver to latest `ojdbc6.jar` (on Java 6) or `ojdbc5.jar` (on Java 5). And JDBC driver can be located in Rails application `./lib` directory as well.

* `RAW` data type is now supported (which is quite often used in legacy databases instead of nowadays recommended `CLOB` and `BLOB` types).

* `rake db:create` and `rake db:drop` can be used to create development or test database schemas.

* Support for virtual columns in improved (only working on Oracle 11g database).

* Default table, index, CLOB and BLOB tablespaces can be specified (if your DBA is insisting on putting everything in separate tablespaces :)).

* Several improvements for context index additional options and definition dump.

See list of [all enhancements and bug fixes](https://github.com/rsim/oracle-enhanced/blob/master/History.txt)

If you want to have a new feature in Oracle enhanced adapter then the best way is to implement it by yourself and write some tests for that feature and send me pull request. In this release I have included commits from five new contributors and two existing contributors - so it is not so hard to start contributing to open source!

### Readme Driven Development

One of the worst parts of Oracle enhanced adapter so far was that for new users it was quite hard to understand how to start to use it and what are all additional features that Oracle enhanced adapter provides. There were many blog posts in this blog, there were wiki pages, there were posts in discussion forums. But all this information was in different places and some posts were already outdated and therefore for new users it was hard to understand how to start.

After [reading about Readme Driven Development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html) and [watching presentation about Readme Driven Development](http://igniterailsconf.com/speakers/677-matt-parker) I knew that README of Oracle enhanced adapter was quite bad and should be improved (in my other projects I am already trying to be better but this was my first one :)).

Therefore I have written [new README of Oracle enhanced adapter](https://github.com/rsim/oracle-enhanced#readme) which includes main installation, configuration, usage and troubleshooting tasks which previously was scattered across different other posts. If you find that some important information is missing or outdated then please submit patches to README as well so that it stays up to date and with relevant information.

If you have any questions please use [discussion group](http://groups.google.com/group/oracle-enhanced) or [report issues at GitHub](http://github.com/rsim/oracle-enhanced/issues) or post comments here.
