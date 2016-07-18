title=Unit Testing Mule DataWeave Scripts
date=07-17-2016
category=MUnit
tags=DataWeave, Transform Message, Mule ESB
status=draft
~~~~~~

[DataWeave](https://docs.mulesoft.com/mule-user-guide/v/3.8/dataweave) is a powerful transformation language introduced with Mule Enterprise Edition 3.7. It allows to transform data from one format to another and supports CSV, XML, JSON, Flat/Fixed Width (v3.8+) and Java. You can look at [these DataWeave Examples](https://docs.mulesoft.com/mule-user-guide/v/3.8/dataweave-examples) to see it in action.

Like any other code of programming world, it is a good idea to unit test the DataWeave script you write. In this post, we will see how we can unit test the DataWeave code.

DataWeave script can be included in two ways into Mule flow - 

####1. Add an inline script -

```
<dw:transform-message doc:name="Transform Message">
            <dw:set-payload><![CDATA[%dw 1.0
%output application/java
---
{
	employees: payload.root.*employee map {
		
			name: $.fname ++ ' ' ++ $.lname,
			dob: $.dob,
			age: (now as :string {format: "yyyy"}) - 
					(($.dob as :date {format:"MM-dd-yyyy"}) as :string {format:"yyyy"})
		
	}
}]]></dw:set-payload>
        </dw:transform-message>
        
```

####2. Add script to file (.dwl) and refer with `resource` attribute -

```
 <dw:transform-message doc:name="Transform Message">
            <dw:set-payload resource="classpath:dwl/employees.dwl"/>
        </dw:transform-message>
```

I like to use `resource` option for writing my DataWeave scripts. This has few advantages over `inline` option -
1. Script (.dwl) is reusable by other trasnform messages components by referring to same file.
2. Mule configuration xml file remains clean and readable.
3. Most important for us, it makes it possible to test the script as an unit of code.
4. Ah, there may be more but I just don't know them yet :).


## Flow with DataWeave component.
To keep it simple, we will use below flow that consumes an employees.xml and transforms it to a CSV file. During transformation it also calculate employee's age.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:file="http://www.mulesoft.org/schema/mule/file" xmlns:dw="http://www.mulesoft.org/schema/mule/ee/dw" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
	xmlns:spring="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-current.xsd
http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
http://www.mulesoft.org/schema/mule/file http://www.mulesoft.org/schema/mule/file/current/mule-file.xsd
http://www.mulesoft.org/schema/mule/ee/dw http://www.mulesoft.org/schema/mule/ee/dw/current/dw.xsd">

    <flow name="dataweave-testingFlow">
        <file:inbound-endpoint path="input" moveToDirectory="output" responseTimeout="10000" doc:name="File"/>
        <dw:transform-message doc:name="Transform Message">
            <dw:input-payload doc:sample="sample_data/empty.xml"/>
            <dw:set-payload resource="classpath:dwl/employees.dwl"/>
        </dw:transform-message>
        <file:outbound-endpoint path="outbound" outputPattern="sample.csv" responseTimeout="10000" doc:name="File"/>
    </flow>
</mule>
```

**DataWeave Script** - `employees.dwl`

```
%dw 1.0
%output application/java
---
{
	employees: payload.root.*employee map {
		
			name: $.fname ++ ' ' ++ $.lname,
			dob: $.dob,
			age: (now as :string {format: "yyyy"}) - 
					(($.dob as :date {format:"MM-dd-yyyy"}) as :string {format:"yyyy"})
		
	}
}
```


Note: CSV input with windows might not work, need to convert the file to unix format.
