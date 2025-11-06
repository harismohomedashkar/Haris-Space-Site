---
# 786
weight: 700
---
# GOF Design Patterns : Bridge

## Introduction

Bridge Design Pattern is one of the GOF Design Patterns, which falls under the Structural Design Patterns. We can define the Bridge Design Pattern as,

> A Design Pattern that decouples an abstraction from its implementation so that the two can vary independently.

The main purpose of this design pattern is to allow users to maintain separate hierarchies instead of maintaining multiple subclasses for each combination of abstraction. So the developers can keep things flexible by introducing separation of concerns. And also allows these classes to evolve independently throughout time. 

## Example

To get a better understanding, let's try to apply this pattern for an example.

Assume we have a system that allows users to export two kinds of document types,

- Invoice
- Monthly Summary

And these documents can be exported either as a `PDF` or as an `XLSX` format. Following these requirements, let's try to develop a hierarchy as follows.

```java
record InvoiceData(){}
record MonthlySummaryData() {}

interface Report{
	void export();
}

abstract class InvoiceReport implements Report{
	InvoiceData fetchData() {return new InvoiceData();}
}

abstract class MonthlySummaryReport implements Report{
	MonthlySummaryData fetchData() {return new MonthlySummaryData();}
}

class InvoicePDFReport extends InvoiceReport{

	@Override
	public void export() {
		InvoiceData data = fetchData();
		// PDF Generation Logic Here
	}
	
}

class InvoiceExcelReport extends InvoiceReport{

	@Override
	public void export() {
		InvoiceData data = fetchData();
		// Excel Generation Logic Here
	}
	
}

class MonthlySummaryPDFReport extends MonthlySummaryReport{

	@Override
	public void export() {
		MonthlySummaryData data = fetchData();
		// PDF Generation Logic Here
	}
	
}

class MonthlySummaryExcelReport extends MonthlySummaryReport{

	@Override
	public void export() {
		MonthlySummaryData data = fetchData();
		// Excel Generation Logic Here
	}
	
}
```

The class diagram looks as below

![](/images/before-bridge.png)

## Problem

As we can see, the above example has put both format and report type into the same class hierarchy. This makes classes tightly coupled and won't allow format and report type to evolve independently. Also, have reduced the flexibility and reusability of the code by maintaining multiple subclasses for each combination of abstraction. 

Assume document format (PDF/Excel) is maintained by one team and document type (Invoice/Monthly Summary) is maintained by another team. According to the above hiearchy can these teams work independantly ? The answer is **NO**.

Let's see how we can use **Bridge Design Pattern** to introduce robust hiearchy, which solves these issues.

## Solution

Bridge Design Pattern Classes
```java
interface ReportData {}
record InvoiceData() implements ReportData{}
record MonthlySummaryData() implements ReportData{}

abstract class Report{
	protected Exporter exporter;
	public Report(Exporter exporter){
		this.exporter = exporter;
	}
	public void export() {
		ReportData reportData = fetchData();
		exporter.export(reportData);
	}
	abstract ReportData fetchData();
}

class InvoiceReport extends Report{
	public InvoiceReport(Exporter exporter) {
		super(exporter);
	}
	@Override
	ReportData fetchData() {
		System.out.println("Fetching Invoice Data . . .");
		return new InvoiceData();
	}
}

class MonthlySummaryReport extends Report{
	public MonthlySummaryReport(Exporter exporter) {
		super(exporter);
	}
	@Override
	ReportData fetchData() {
		System.out.println("Fetching Monthly Summary Data . . .");
		return new MonthlySummaryData();
		}
}

interface Exporter{
	void export(ReportData reportData);
}

class PDFExporter implements Exporter{
	@Override
	public void export(ReportData reportData) {
		System.out.println("Exporting as a PDF . . .\n");
		// PDF EXPORT LOGIC	
	}
}

class ExcelExporter implements Exporter{
	@Override
	public void export(ReportData reportData) {
		System.out.println("Exporting as a EXCEL . . .\n");
		// EXCEL EXPORT LOGI>C
	}
}
```

Now let's run this example
```java
public class BridgeDemo {
	public static void main(String[] args) {
		new InvoiceReport(new PDFExporter()).export(); // InvoiceReport PDF
		new MonthlySummaryReport(new PDFExporter()).export(); // MonthlySummaryReport PDF
		new InvoiceReport(new ExcelExporter()).export(); // InvoiceReport Excel
		new MonthlySummaryReport(new ExcelExporter()).export(); // MonthlySummaryReport Excel
	}
}
```

Output
```text
Fetching Invoice Data . . .
Exporting as a PDF . . .

Fetching Monthly Summary Data . . .
Exporting as a PDF . . .

Fetching Invoice Data . . .
Exporting as a EXCEL . . .

Fetching Monthly Summary Data . . .
Exporting as a EXCEL . . .
```

## Class Diagram

After applying the bridge design pattern, we can see a more robust, well architectured class diagram as below.

![](/images/after-bridge.png)

As we can see, we have successfully built a robust class hierarchy where,

- Combinational subclassing is avoided
- Abstraction and implementation are separated   
- Each hierarchy can be developed separately   
- Code bases can be extended/developed separately by different developer teams  
- lower coupling / flexibility / reusability improved

That's all for Bridge Pattern for now.

Hope this article was helpful.\
Thank you !\
Happy Coding 🙌