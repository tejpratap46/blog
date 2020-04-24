## Android: Create PDF without any library

[PDF](https://en.wikipedia.org/wiki/PDF) are simple document format with supports text and images. It is simple to share, non editable can be password protected. Great for receipts and invoices.

As i was looking for a library to create PDF natively on Android but couldn't found one that is free to use.

Some time ago, i had a use case where i needed to create PDF on device and unfortunately no free to use library was available at that time which can create PDF's on user's device.

So, i digged into Android's  [Printing custom documents](https://developer.android.com/training/printing/custom-docs.html) where i come across  [PdfDocument](https://developer.android.com/reference/android/graphics/pdf/PdfDocument) which can generate PDF from `View`.

Here is how it is done:

1. We start by creating a PdfDocument
```java
// Create PDF Document.
PdfDocument pdfDocument = new PdfDocument();
```

2. Create a Page in PdfDocument
```java
PdfDocument.PageInfo pageInfo = new PdfDocument.PageInfo.
                        Builder((int) PDF_PAGE_WIDTH, (int) PDF_PAGE_HEIGHT, i + 1).create();
```

3. Create a view to be printed on PDF
```java
TextView view = new TextView(context);
view.setText("PDF Example");
```

4. Draw view on the page
```java
Canvas pageCanvas = page.getCanvas();
pageCanvas.scale(1f, 1f);
int pageWidth = pageCanvas.getWidth();
int pageHeight = pageCanvas.getHeight();
int measureWidth = View.MeasureSpec.makeMeasureSpec(pageWidth, View.MeasureSpec.EXACTLY);
int measuredHeight = View.MeasureSpec.makeMeasureSpec(pageHeight, View.MeasureSpec.EXACTLY);
contentView.measure(measureWidth, measuredHeight);
contentView.layout(0, 0, pageWidth, pageHeight);
contentView.draw(pageCanvas);
```

5. Finish the page
```java
pdfDocument.finishPage(page);
```

6. Save pdf to file
```java
File pdfFile = new File(mFilePath);
// Write PDFDocument to the file.
FileOutputStream  fos = new FileOutputStream(pdfFile);
pdfDocument.writeTo(fos);
// Close output stream
fos.close();
// close the document
pdfDocument.close();
``` 

Above code will create a simple PDF with one page that contains TextView inside it.

***
Now let's start creating an actual PDF...
#### Creating PDF: Simple

PdfDocument API will renders any views to PDF. It is as simple as creating an activity, getting its `view` and rendering PdfDocument of that view.

> *A Simple PDF page is of `height="842px"` and `width="595px"`*
>
> *Note: As dp and sp are device dependent, make sure you create all of your layout in px as it will look same in all devices.*

Steps:
1. Create an `Activity` which will have a vertical `LinearLayout` with `height="842px"` and `width="595px"`.
2. Add all views with static data inside it.
3. Make sure no view exceeds allotted height as it will be trimmed.
4. If you want to create new page just repeat 1-3.
5. In your `onCreate()`, get reference of all `LinearLayout` and create PDF (using step 4) with each layout as one page.
6. Done.