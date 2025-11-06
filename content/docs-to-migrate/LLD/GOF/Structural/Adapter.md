---
# 786
weight: 300
---
# GOF Design Patterns : Adapter

## Problem

Let's say we have an application that processes financial data. This application fetches
financial transactions from external services as XML files, and the code is written to
process these XML files.

Over the time, the application has extended its domain. Now the application gets transaction
information from another external source.

But the issue is that the new source is sending JSON instead of XML.

So let's try to solve this issue. The straight forward way is to write another set of code to
process JSON data. But when we look at the long term large picture, it is not a good design decision.

Because let's assume that after a while we decide to add a new source, and this source is
using CSV. Now do we have to write separate code for CSV ?


![](/images/adapter1.png)

Yeah, then it will be harder to maintain separate processing logics for XML, JSON, and CSV. So
our design decision should be able to easily cater to new requirements which may come in the
future.

Simply put, our problem is how to deal with incompatible interfaces. Here it is between the XML
reader from the application and data format provided format the source.

## Solution

This is where the Adapter Design Pattern comes into use. The Adapter Design Pattern is a
structural design pattern that allows objects with incompatible interfaces to collaborate.

Which means we are neither changing our XML processing logic nor adding new logic to
process JSON.

Instead of that, we just develop adapters to wrap around incompatible interfaces to make
them usable by our XML processing logic (make it compatible to deal with).

![](/images/adapter2.png)

Let's look at the implementation

![](/images/adapter3.png)

## Code

```java

class FinancialDataProcessor {
    String getRootAccount(XMLSourceData xmlData) {
        String value = xmlData.readXPath("/financialData/account/id");
        return value;
    }
}

class XMLSourceData {
    private Document doc;
    public XMLSourceData(Document doc) {
        this.doc = doc;
    }
    public String readXPath(String expression) {
        try {
            XPathFactory xpathFactory = XPathFactory.newInstance();
            XPath xpath = xpathFactory.newXPath();
            return xpath.evaluate(expression, doc);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

class JsonToXmlSourceDataAdapter extends XMLSourceData {
    public JsonToXmlSourceDataAdapter(JSONSourceData jsonData) {
        super(convertJsonToXml(jsonData));
    }
    private static Document convertJsonToXml(JSONSourceData jsonData) {
        // JSON => XML Conversion Logic
    }
}

class CsvToXmlSourceDataAdapter extends XMLSourceData {
    public CsvToXmlSourceDataAdapter(CSVSourceData csvData) {
        super(convertCsvToXml(csvData));
    }
    private static Document convertCsvToXml(CSVSourceData csvData) {
        // CSV to XML Conversion Logic
    }
}

public class Main {

    public static void main(String[] args) {

        try {

            // XML Source
            String xml = "<financialData><account><id>ACC2345452321</id></account></financialData>";
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
            DocumentBuilder builder = factory.newDocumentBuilder();
            Document xmlDoc = builder.parse(new InputSource(new StringReader(xml)));
            XMLSourceData xmlData = new XMLSourceData(xmlDoc);

            // JSON Source
            String json = "{ \"financialData\": { \"account\": { \"id\": \"ACC2345452321\" } } }";
            JSONSourceData jsonData = new JSONSourceData(json);
            XMLSourceData jsonAdapter = new JsonToXmlSourceDataAdapter(jsonData);

            // CSV Source
            String csv = "account_id\nACC2345452321";
            CSVSourceData csvData = new CSVSourceData(csv);
            XMLSourceData csvAdapter = new CsvToXmlSourceDataAdapter(csvData);

            // Now We Processing Different Data Sources with Different Formats
            FinancialDataProcessor processor = new FinancialDataProcessor();
            System.out.println("Root Account: " + processor.getRootAccount(xmlData));
            System.out.println("Root Account: " + processor.getRootAccount(jsonAdapter));
            System.out.println("Root Account: " + processor.getRootAccount(csvAdapter));

        } catch (Exception e) {

            e.printStackTrace();

        }

    }

}
```

As we can see now the system can handle different types of input data sources easily, without much effort.

Hope this article was helpful.
Thank you !

Happy Coding 🙌