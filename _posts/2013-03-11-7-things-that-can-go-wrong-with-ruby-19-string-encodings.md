---
layout: post
title: 7 things that can go wrong with Ruby 1.9 string encodings
tags: ruby jruby encoding jira
---

Good news, I am back in blogging :) In recent years I have spent my time primarily on [eazyBI](https://eazybi.com) business intelligence application development where I use [JRuby](http://jruby.org/), [Ruby on Rails](http://rubyonrails.org/), [mondrian-olap](https://github.com/rsim/mondrian-olap) and many other technologies and libraries and have gathered new experience that I wanted to share with others.

Recently I did eazyBI migration from JRuby 1.6.8 to latest JRuby 1.7.3 version as well as finally migrated from Ruby 1.8 mode to Ruby 1.9 mode. Initial migration was not so difficult and was done in one day (thanks to unit tests which caught majority of differences between Ruby 1.8 and 1.9 syntax and behavior).

But then when I thought that everything is working fine I got quite many issues related to Ruby 1.9 string encodings which unfortunately were not identified by test suite and also not by my initial manual tests. Therefore I wanted to share these issues which might help you to avoid these issues in your Ruby 1.9 applications.

If you are new to Ruby 1.9 string encodings then at first read, for example, tutorials about [Ruby 1.9 String](http://blog.grayproductions.net/articles/ruby_19s_string) and [Ruby 1.9 Three Default Encodings](http://blog.grayproductions.net/articles/ruby_19s_three_default_encodings), as well as [Ruby 1.9 Encodings: A Primer and the Solution for Rails](http://yehudakatz.com/2010/05/05/ruby-1-9-encodings-a-primer-and-the-solution-for-rails/) is useful.

### 1. Encoding header in source files

I will start with the easy one - if you use any Unicode characters in your Ruby source files then you need to add

```ruby
# encoding: utf-8
```

magic comment line in the beginning of your source file. This was easy as it was caught by unit tests :)


### 2. Nokogiri XML generation

The next issues were with XML generation using Nokogiri gem when XML contains Unicode characters. For example,

```ruby
require "nokogiri"
doc = Nokogiri::XML::Builder.new do |xml|
  xml.dummy :name => "āčē"
end
puts doc.to_xml
```

will give the following result when using MRI 1.9:

```xml
<?xml version="1.0"?>
<dummy name="&#x101;&#x10D;&#x113;"/>
```

which might not be what you expect if you would like to use UTF-8 encoding also for Unicode characters in generated XML file. If you execute the same ruby code in JRuby 1.7.3 in default Ruby 1.9 mode then you get:

```xml
<?xml version="1.0"?>
<dummy name="āčē"/>
```

which seems OK. But actually it is not OK if you look at generated string encoding:

```ruby
doc.to_xml.encoding # => #<Encoding:US-ASCII>
doc.to_xml.inspect  # => "<?xml version=\"1.0\"?>\n<dummy name=\"\xC4\x81\xC4\x8D\xC4\x93\"/>\n"
```

In case of JRuby you see that `doc.to_xml` encoding is US-ASCII (which is 7 bit encoding) but actual content is using UTF-8 8-bit encoded characters. As a result you might get `ArgumentError: invalid byte sequence in US-ASCII` exceptions later in your code.

Therefore it is better to tell Nokogiri explicitly that you would like to use UTF-8 encoding in generated XML:

```ruby
doc = Nokogiri::XML::Builder.new(:encoding => "UTF-8") do |xml|
  xml.dummy :name => "āčē"
end
doc.to_xml.encoding # => #<Encoding:UTF-8>
puts doc.to_xml
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<dummy name="āčē"/>
```

### 3. CSV parsing

If you do CSV file parsing in your application then the first thing you have to do is to replace FasterCSV gem (that you probably used in Ruby 1.8 application) with standard Ruby 1.9 CSV library.

If you process user uploaded CSV files then typical problem is that even if you ask to upload files in UTF-8 encoding then quite often you will get files in different encodings (as Excel is quite bad at producing UTF-8 encoded CSV files).

If you used FasterCSV library with non-UTF-8 encoded strings then you get ugly result but nothing will blow up:

```ruby
FasterCSV.parse "\xE2"
# => [["\342"]]
```

If you do the same in Ruby 1.9 with CSV library then you will get ArgumentError exception.

```ruby
CSV.parse "\xE2"
# => ArgumentError: invalid byte sequence in UTF-8
```

It means that now you need to rescue and handle ArgumentError exceptions in all places where you try to parse user uploaded CSV files to be able to show user friendly error messages.

The problem with standard CSV library is that it is not handling ArgumentError exceptions and is not wrapping them in MalformedCSVError exception with information in which line this error happened (as it is done with other CSV format exceptions) which makes debugging very hard. Therefore I also "monkey patched" `CSV#shift` method to add ArgumentError exception handling.

### 4. YAML serialized columns

ActiveRecord has standard way how to serialize more complex data types (like Array or Hash) in database text column. You use `serialize` method to declare serializable attributes in your ActiveRecord model class definition. By default YAML format (using `YAML.dump` method for serialization) is used to serialize Ruby object to text that is stored in database.

But you can get big problems if your data contains string with Unicode characters as YAML implementation significantly changed between Ruby 1.8 and 1.9 versions:

* Ruby 1.8 used so-called Syck library
* JRuby in 1.8 mode used Java based implementation that tried to ack like Syck
* Ruby 1.9 and JRuby in 1.9 mode use new Psych library

Lets try to see results what happens with YAML serialization of simple Hash with string value which contains Unicode characters.

On MRI 1.8:

```ruby
YAML.dump({:name => "ace āčē"})
# => "--- \n:name: !binary |\n  YWNlIMSBxI3Ekw==\n\n"
```

On JRuby 1.6.8 in Ruby 1.8 mode:

```ruby
YAML.dump({:name => "ace āčē"})
# => "--- \n:name: \"ace \\xC4\\x81\\xC4\\x8D\\xC4\\x93\"\n"
```

On MRI 1.9 or JRuby 1.7.3 in Ruby 1.9 mode:

```ruby
YAML.dump({:name => "ace āčē"})
# => "---\n:name: ace āčē\n"
```

So as we see all results are different. But now lets see what happens after we migrated our Rails application from Ruby 1.8 to Ruby 1.9. All our data in database is serialized using old YAML implementations but now when loaded in our application they are deserialized back using new Ruby 1.9 YAML implementation.

When using MRI 1.9:

```ruby
YAML.load("--- \n:name: !binary |\n  YWNlIMSBxI3Ekw==\n\n")
# => {:name=>"ace \xC4\x81\xC4\x8D\xC4\x93"}
YAML.load("--- \n:name: !binary |\n  YWNlIMSBxI3Ekw==\n\n")[:name].encoding
# => #<Encoding:ASCII-8BIT>
```

So the string that we get back from database is no more in UTF-8 encoding but in ASCII-8BIT encoding and when we will try to concatenate it with UTF-8 encoded strings we will get `Encoding::CompatibilityError: incompatible character encodings: ASCII-8BIT and UTF-8` exceptions.

When using JRuby 1.7.3 in Ruby 1.9 mode then result again will be different:

```ruby
YAML.load("--- \n:name: \"ace \\xC4\\x81\\xC4\\x8D\\xC4\\x93\"\n")
# => {:name=>"ace Ä\u0081Ä\u008DÄ\u0093"}
YAML.load("--- \n:name: \"ace \\xC4\\x81\\xC4\\x8D\\xC4\\x93\"\n")[:name].encoding
# => #<Encoding:UTF-8>
```

So now result string has UTF-8 encoding but the actual string is damaged. It means that we will not even get exceptions when concatenating result with other UTF-8 strings, we will just notice some strange garbage instead of Unicode characters.

The problem is that there is no good solution how to convert your database data from old YAML serialization to new one. In MRI 1.9 at least it is possible to switch back YAML to old Syck implementation but in JRuby 1.7 when using Ruby 1.9 mode it is not possible to switch to old Syck implementation.

Current workaround that I did is that I made modified serialization class that I used in all model class definitions (this works in Rails 3.2 and maybe in earlier Rails 3.x versions as well):

```ruby
serialize :some_column, YAMLColumn.new
```

`YAMLColumn` implementation is a copy from original `ActiveRecord::Coders::YAMLColumn` implementation. I modified `load` method to the following:

```ruby
def load(yaml)
  return object_class.new if object_class != Object && yaml.nil?
  return yaml unless yaml.is_a?(String) && yaml =~ /^---/
  begin
    # if yaml sting contains old Syck-style encoded UTF-8 characters
    # then replace them with corresponding UTF-8 characters
    # FIXME: is there better alternative to eval?
    if yaml =~ /\\x[0-9A-F]{2}/
      yaml = yaml.gsub(/(\\x[0-9A-F]{2})+/){|m| eval "\"#{m}\""}.force_encoding("UTF-8")
    end
    obj = YAML.load(yaml)

    unless obj.is_a?(object_class) || obj.nil?
      raise SerializationTypeMismatch,
        "Attribute was supposed to be a #{object_class}, but was a #{obj.class}"
    end
    obj ||= object_class.new if object_class != Object

    obj
  rescue *RESCUE_ERRORS
    yaml
  end
end
```

Currently this patched version will work on JRuby where just non-ASCII characters are replaced by `\xNN` style fragments (byte with hex code NN). When loading existing data from database we check if it has any such \xNN fragment and if yes then these fragments are replaced with corresponding UTF-8 encoded characters. If anyone has better suggestion for implementation without using `eval` then please let me know in comments :)

If you need to create something similar for MRI then you would probably need to search if database text contains `!binary |` fragment and if yes then somehow transform it to corresponding UTF-8 string. Anyone has some working example for this?

### 5. Sending binary data with default UTF-8 encoding

I am using [spreadsheet gem](https://rubygems.org/gems/spreadsheet) to generate dynamic Excel export files. The following code was used to get generated spreadsheet as String:

```ruby
book = Spreadsheet::Workbook.new
# ... generate spreadsheet ...
buffer = StringIO.new
book.write buffer
buffer.seek(0)
buffer.read
```

And then this string was sent back to browser using controller `send_data` method.

The problem was that in Ruby 1.9 mode by default `StringIO` will generate strings with UTF-8 encoding. But Excel format is binary format and as a result `send_data` failed with exceptions that UTF-8 encoded string contains non-UTF-8 byte sequences.

The fix was to set StringIO buffer encoding to `ASCII-8BIT` (or you can use alias `BINARY`):

```ruby
buffer = StringIO.new
buffer.set_encoding('ASCII-8BIT')
```

So you need to remember that in all places where you handle binary data you cannot use strings with default UTF-8 encoding but need to specify ASCII-8BIT encoding.

### 6. JRuby Java file.encoding property

Last two issues were JRuby and Java specific. Java has system property `file.encoding` which is not related just to file encoding but determines default character set and string encoding in many places.

If you do not specify `file.encoding` explicitly then Java VM on startup will try to determine its default value based on host operating system "locale". On Linux it might be that it will be set to UTF-8, on Mac OS X by default it will be MacRoman, on Windows it will depend on Windows default locale setting (which will not be UTF-8). Therefore it is always better to set explicitly `file.encoding` property for Java applications (e.g. using `-Dfile.encoding=UTF-8` command line flag).

`file.encoding` will determine which default character set `java.nio.charset.Charset.defaultCharset()` method call will return. And even if you change `file.encoding` property during runtime it will not change `java.nio.charset.Charset.defaultCharset()` result which is cached during startup.

JRuby uses `java.nio.charset.Charset.defaultCharset()` in very many places to get default system encoding and uses it in many places when constructing Ruby strings. If `java.nio.charset.Charset.defaultCharset()` will not return UTF-8 character set then it might result in problems when using Ruby strings with UTF-8 encoding. Therefore in JRuby startup scripts (`jruby`, `jirb` and others) `file.encoding` property is always set to `UTF-8`.

So if you start your JRuby application in standard way using `jruby` script then you should have `file.encoding` set to `UTF-8`. You can check it in your application using `ENV_JAVA['file.encoding']`.

But if you start your JRuby application in non-standard way (e.g. you have JRuby based plugin for some other Java application) then you might not have `file.encoding` set to `UTF-8` and then you need to worry about it :)

### 7. JRuby Java string to Ruby string conversion

I got `file.encoding` related issue in [eazyBI reports and charts plugin for JIRA](https://marketplace.atlassian.com/plugins/com.eazybi.jira.plugins.eazybi-jira). In this case eazyBI plugin is OSGi based plugin for [JIRA issue tracking system](http://atlassian.com/software/jira/overview/) and JRuby is running as a scripting container inside OSGi bundle.

JIRA startup scripts do not specify `file.encoding` default value and as a result it typically is set to operating system default value. For example, on my Windows test environment it is set to Windows-1252 character set.

If you call Java methods of Java objects from JRuby then it will automatically convert `java.lang.String` objects to Ruby `String` objects but Ruby strings in this case will use encoding based on `java.nio.charset.Charset.defaultCharset()`. So even when Java string (which internally uses UTF-16 character set for all strings) can contain any Unicode character it will be returned to Ruby not as string with UTF-8 encoding but in my case will return with Windows-1252 encoding. As a result all Unicode characters which are not in this Windows-1252 character set will be lost.

And this is very bad because everywhere else in JIRA it does not use `java.nio.charset.Charset.defaultCharset()` and can handle and store all Unicode characters even when `file.encoding` is not set to UTF-8.

Therefore I finally managed to create a workaround which forces that all Java strings are converted to Ruby strings using UTF-8 encoding.

I created custom Java string converter based on standard one in `org.jruby.javasupport.JavaUtil` class:

```java
package com.eazybi.jira.plugins;

import org.jruby.javasupport.JavaUtil;
import org.jruby.Ruby;
import org.jruby.RubyString;
import org.jruby.runtime.builtin.IRubyObject;

public class RailsPluginJavaUtil {
    public static final JavaUtil.JavaConverter JAVA_STRING_CONVERTER = new JavaUtil.JavaConverter(String.class) {
        public IRubyObject convert(Ruby runtime, Object object) {
            if (object == null) return runtime.getNil();
            // PATCH: always convert Java string to Ruby string with UTF-8 encoding
            // return RubyString.newString(runtime, (String)object);
            return RubyString.newUnicodeString(runtime, (String)object);
        }
        public IRubyObject get(Ruby runtime, Object array, int i) {
            return convert(runtime, ((String[]) array)[i]);
        }
        public void set(Ruby runtime, Object array, int i, IRubyObject value) {
            ((String[])array)[i] = (String)value.toJava(String.class);
        }
    };
}
```

Then in my plugin initialization Ruby code I dynamically replaced standard Java string converter to my customized converter:

```ruby
java_converters_field = org.jruby.javasupport.JavaUtil.java_class.declared_field("JAVA_CONVERTERS")
java_converters_field.accessible = true
java_converters = java_converters_field.static_value.to_java
java_converters.put(java.lang.String.java_class, com.eazybi.jira.plugins.RailsPluginJavaUtil::JAVA_STRING_CONVERTER)
```

And as a result now all Java strings that were returned by Java methods were converted to Ruby strings using UTF-8 encoding and not using encoding from `file.encoding` Java property.

### Final thoughts

My main conclusions from solving all these string encoding issues are the following:

* Use UTF-8 encoding as much as possible. Handling conversions between different encodings will be much more harder than you will expect.
* Use example strings with Unicode characters in your tests. I didn't identify all these issues initially when running tests after migration because not all tests were using example strings with Unicode characters. So next time instead of using `"dummy"` string in your test use `"dummy āčē"` everywhere :)

And please let me know (in comments) if you have better or alternative solutions for the issues that I described here.
