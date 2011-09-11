---
layout: post
title: Easy Business Intelligence with eazyBI
tags: eazyBI business-intelligence mondrian-olap
---

I have been interested in business intelligence and data warehouse solutions for quite a while. And I have seen that traditional data warehouse and business intelligence tool implementations take quite a long time and cost a lot to set up infrastructure, develop or implement business intelligence software and train users. And many business users are not using business intelligence tools because they are too hard to learn and use.

<a href="https://eazybi.com"><img src="/images/eazybi.gif" style="float: left; margin: 20px;"/></a>Therefore some while ago a had an idea of developing easy-to-use business intelligence web application that you could implement and start to use right away and which would focus on ease-of-use and not on too much unnecessary features. And result of this idea is [eazyBI](https://eazybi.com) which after couple months of beta testing now is launched in production.

### Many sources of data

One of the first issues in data warehousing is that you need to prepare / import / aggregate data that you would like to analyze. It's hard to create universal solution for this issue therefore I am building several predefined ways how to upload or prepare your data for analysis.

In the simplest case if you have source data in CSV (comma-separated values) format or you can export your data in CSV format then you can [upload these files to eazyBI](https://eazybi.com/help/demo-example), map columns to [dimensions and measures](https://eazybi.com/help/overview) and start to analyze and create reports from uploaded data.

If you are already using some software-as-a-service web applications and would like to have better analysis tools to analyze your data in these applications then eazyBI will provide standard integrations for data import and will create initial sample reports. Currently the first integrations are with [37signals Basecamp project collaboration application](https://eazybi.com/help/basecamp) as well as you can [import and analyze your Twitter timeline](https://eazybi.com/help/demo-twitter). More standard integrations with other applications will follow (please let me know if you would be interested in some particular integration).

One of the main limiting factors for business intelligence as a service solutions is that many businesses are not ready to upload their data to service providers - both because of security concerns as well as uploading of big data volumes takes quite a long time. [Remote eazyBI solution](https://eazybi.com/help/remote-setup) is unique solution which allows you to use eazyBI business intelligence as a service application with your existing data in your existing databases. You just need to download and set up remote eazyBI application and start to analyze your data. As a result you get the benefits of fast implementation of business intelligence as a service but without the risks of uploading your data to service provider.

### Easy-to-use

Many existing business intelligence tools suffer from too many features which make them complicated to use and you need special business intelligence tool consultants which will create reports and dashboards for business users.

Therefore my goal for eazyBI is to make it with less features but any feature that is added should be easy-to-use. With simple drag-and-drop and couple clicks you can select dimensions by which you want to analyze data and start with summaries and then drill into details. Results can be viewed as traditional tables or as different bar, line, pie, timeline or map charts. eazyBI is easy and fun to use as you play with your data.

### Share reports with others

When you have created some report which you would like to share with other your colleagues, customers or partners then you can either send link to report or you can also embed report into other HTML pages. Here is example of embedded report from demo eazyBI account:

<div style="text-align:center;"><iframe width="650" height="475" src="https://eazybi.com/accounts/1/embed/report/111?dashboard_id=6" frameborder="0"></iframe></div>

Embedded reports are fully dynamic and will show latest data when page will be refreshed. They can be embedded in company intranets, wikis or blogs or any other web pages.

### Open web technologies

eazyBI user interface is built with open HTML5/CSS3/JavaScript technologies which allows to use it both on desktop browsers as well as on mobile devices like iPad (you will be able to open reports even on mobile phones like iPhone or Android but of course because of screen size it will be harder to use). And if you will use modern browser like Chrome, Firefox, Safari or Internet Explorer 9 then eazyBI will be also very fast and responsive. I believe that speed should be one of the main features of any application therefore I am optimizing eazyBI to be as fast as possible.

In the backend eazyBI uses open source [Mondrian OLAP engine](http://mondrian.pentaho.com/) and [mondrian-olap](https://github.com/rsim/mondrian-olap) JRuby library that I have created and open-sourced during eazyBI development.

### No big initial investments

[eazyBI pricing](https://eazybi.com/pricing) is based on monthly subscription starting from $20 per month. There is also free plan for single user as well as you can publish public data for free. And you don't need to make long-term commitments - you just pay for the service when you use it and you can cancel it anytime when you don't need it anymore.

### Try it out

If this sounds interesting to you then please [sign up for eazyBI](https://eazybi.com/sign_up) and try to upload and analyze your data. And if you have any suggestions or questions about eazyBI then [please let me know](mailto:support@eazybi.com).

**P.S.** Also I wanted to mention that by subscribing to eazyBI you will also support [my work on open-source](https://github.com/rsim). If you are user of my open-source libraries then I will appreciate if you will become eazyBI user as well :) But in case if you do not need solution like eazyBI you could support me by tweeting and blogging about eazyBI, will be thankful for that as well.
