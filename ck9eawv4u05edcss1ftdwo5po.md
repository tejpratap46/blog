## Android: Create PDF without any library [Part 3]

In  [last part](https://blog.tejpratapsingh.com/android-create-pdf-without-any-library-part-2-ck9eads8g05e3css1bx1mlns2)  we successfully created a complex Pdf with dynamic data.

After we create a Pdf, we should be able to show contents of Pdf to User, to do that we can use `PdfRenderer` class and get bitmap of every page inside Pdf.

Here is a simple function you can use to get Pdf pages as Bitmap

```java
  /**
     * Convert PDF to bitmap, only works on devices above LOLLIPOP
     *
     * @param pdfFile pdf file
     * @return list of bitmap of every page
     * @throws Exception
     */
    public static ArrayList<Bitmap> pdfToBitmap(File pdfFile) throws Exception, IllegalStateException {
        if (pdfFile == null || pdfFile.exists() == false) {
            throw new IllegalStateException("");
        }
        if (android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.LOLLIPOP) {
            throw new Exception("PDF preview image cannot be generated in this device");
        }
        if (android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.LOLLIPOP) {
            return null;
        }

        ArrayList<Bitmap> bitmaps = new ArrayList<>();

        try {
            PdfRenderer renderer = new PdfRenderer(ParcelFileDescriptor.open(pdfFile, ParcelFileDescriptor.MODE_READ_ONLY));

            Bitmap bitmap;
            final int pageCount = renderer.getPageCount();
            for (int i = 0; i < pageCount; i++) {
                PdfRenderer.Page page = renderer.openPage(i);
                
                int width = page.getWidth();
                int height = page.getHeight();

                /* FOR HIGHER QUALITY IMAGES, USE:
                int width = context.getResources().getDisplayMetrics().densityDpi / 72 * page.getWidth();
                int height = context.getResources().getDisplayMetrics().densityDpi / 72 * page.getHeight();
                */

                bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
                page.render(bitmap, null, null, PdfRenderer.Page.RENDER_MODE_FOR_DISPLAY);

                bitmaps.add(bitmap);

                // close the page
                page.close();
            }
            // close the renderer
            renderer.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return bitmaps;
    }
```

***

To make your work easier i have created a Library to generate dynamic PDF using above methods [View on Github](https://github.com/tejpratap46/PDFCreatorAndroid)

 [![Simple library to generate and view PDF in Android.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1587719632043/mXw6_IETk.png)](https://github.com/tejpratap46/PDFCreatorAndroid) 