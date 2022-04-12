***1.质量压缩***

    private void compressQuality() {
        Bitmap bm = BitmapFactory.decodeResource(getResources(), R.drawable.test);
        mSrcSize = bm.getByteCount() + "byte";
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        bm.compress(Bitmap.CompressFormat.JPEG, 100, bos);
        byte[] bytes = bos.toByteArray();
        mSrcBitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);
    }
    
质量压缩不会减少图片的像素，它是在保持像素的前提下改变图片的位深及透明度，来达到压缩图片的目的，图片的长，宽，像素都不会改变，那么bitmap所占内存大小是不会变的。

我们可以看到有个参数：quality，可以调节你压缩的比例，但是还要注意一点就是，质量压缩堆png格式这种图片没有作用，因为png是无损压缩。

***2.采样率压缩***

    private void compressSampling() {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inSampleSize = 2;
        mSrcBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test, options);
    }
    
采样率压缩其原理其实也是缩放bitamp的尺寸，通过调节其inSampleSize参数，比如调节为2，宽高会为原来的1/2，内存变回原来的1/4.

***3.放缩法压缩***

    private void compressMatrix() {
        Matrix matrix = new Matrix();
        matrix.setScale(0.5f, 0.5f);
        Bitmap bm = BitmapFactory.decodeResource(getResources(), R.drawable.test);
        mSrcBitmap = Bitmap.createBitmap(bm, 0, 0, bm.getWidth(), bm.getHeight(), matrix, true);
        bm = null;
    }
    
放缩法压缩使用的是通过矩阵对图片进行裁剪，也是通过缩放图片尺寸，来达到压缩图片的效果，和采样率的原理一样。

***4.RGB_565压缩***

    private void compressRGB565() {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inPreferredConfig = Bitmap.Config.RGB_565;
        mSrcBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test, options);
    }
    
这是通过压缩像素占用的内存来达到压缩的效果，一般不建议使用ARGB_4444，因为画质实在是辣鸡，如果对透明度没有要求，建议可以改成RGB_565，相比ARGB_8888将节省一半的内存开销。

***5.createScaledBitmap***

    private void compressScaleBitmap() {
        Bitmap bm = BitmapFactory.decodeResource(getResources(), R.drawable.test);
        mSrcBitmap = Bitmap.createScaledBitmap(bm, 600, 900, true);
        bm = null;
    }
    
将图片的大小压缩成用户的期望大小，来减少占用内存。



链接：https://www.jianshu.com/p/08ed0e3c4e71
