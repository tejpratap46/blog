## Android: Create PDF without any library [Part 2]

In  [last part](https://blog.tejpratapsingh.com/android-create-pdf-without-any-library-ck9dz0vdg05agcxs14l1ix0rg)  we created a static PDF in Android.

But creating PDF from static data has a big disadvantage. You cannot set data at runtime, i.e. you cannot set text from DB or load image from internet.

#### Creating PDF: Advanced

To solve this problem:
- We will create empty Activity with `LinearLayout` of size **[595xwrap_content]**, we will get reference of that `LinearLayout` and Add our Views programmatically.
- We only need above LinearLayout so we can calculate height of views we are about to add. As `View`s 
created programmatically cannot give height they will take after rendering. i.e. rendering of each `View` is mandatory.
- Now we know why we need to first render `View` to layout, let's proceed:
- Let's sat we need to add a `TextView` to Pdf, we will call it 'textView1'.
- 'textView1' will be added to **595xwrap_content** `LinearLayout`, So it can render and we take a note of its height.

```java
private final Map<String, int> viewToHeightMap = new HashMap<>();

final LinearLayout layout595xwrap_content = findViewById(R.id.layout595xwrap_content);

final TextView textView1 = new TextView(context);
layout595xwrap_content.add(textView1);
textView1.post(new Runnable() {
  void run() {
    layout595xwrap_content.put('textView1', textView1.getHeight())
  }
})
```
- Accordingly add all view to **595xwrap_content** `LinearLayout` one-by-one and take a note of height of each view.
- Create `LinearLayout` of size **595x842px** which will be Single Page in Pdf. Add Views one-by-one until height sum reaches 842px.
- When 842px height is reached, again create new LinearLayout of size **595x842px** which will be Second Page in the Pdf and so on.
- After you have create List of `LineatLayout`s as Pdf Pages, we simply have to  [use methods](https://blog.tejpratapsingh.com/android-create-pdf-without-any-library-ck9dz0vdg05agcxs14l1ix0rg)  in previous part to generate Pdf.

In  [Next Part](https://blog.tejpratapsingh.com/android-create-pdf-without-any-library-part-3-ck9eawv4u05edcss1ftdwo5po)  we will generate preview of any Pdf File using `PdfRenderer`

***

To make your work easier i have created a Library to generate dynamic PDF using above methods [View on Github](https://github.com/tejpratap46/PDFCreatorAndroid)

 [![Simple library to generate and view PDF in Android.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1587719632043/mXw6_IETk.png)](https://github.com/tejpratap46/PDFCreatorAndroid) 