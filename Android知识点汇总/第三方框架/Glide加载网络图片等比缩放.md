在做android图片加载的时候，由于手机屏幕受限，很多大图加载过来的时候，我们要求等比例缩放，比如按照固定的宽度，等比例缩放高度，使得图片的尺寸比例得到相应的缩放，但图片没有变形。

显然按照android:scaleType不能实现，因为会有很多限制，所以必须要自己写算法。 

**通过Glide来缩放**

其实glide提供了这样的方法。具体是显示继承Transformation 的 setResource 方法。 

    (1) 先获取网络或本地图片的宽高 
    (2) 获取需要的目标宽 
    (3) 按比例得到目标的高度 
    (4) 按照目标的宽高创建新图
    
    
    /**
     * ===========================================
     * 版    本：1.0
     * 描    述：设置图片等比缩放
     * <p>glide处理图片.</p>
     * ===========================================
     */
    public class TransformationUtils extends ImageViewTarget<Bitmap> {

        private ImageView target;

        public TransformationUtils(ImageView target) {
            super(target);
            this.target = target;
        }

        @Override
        protected void setResource(Bitmap resource) {
            view.setImageBitmap(resource);

            //获取原图的宽高
            int width = resource.getWidth();
            int height = resource.getHeight();

            //获取imageView的宽
            int imageViewWidth = target.getWidth();

            //计算缩放比例
            float sy = (float) (imageViewWidth * 0.1) / (float) (width * 0.1);

            //计算图片等比例放大后的高
            int imageViewHeight = (int) (height * sy);
            ViewGroup.LayoutParams params = target.getLayoutParams();
            params.height = imageViewHeight;
            target.setLayoutParams(params);
        }
    }
    
    
之后在Glide设置transform

    Glide.with(this)
        .load(newActiviteLeftBannerUrl)
        .asBitmap()
        .placeholder(R.drawable.placeholder)
        .into(new TransformationUtils(target));

Transformation 这是Glide的一个非常强大的功能了，它允许你在load图片 -> into ImageView 中间这个过成对图片做一系列的变换。
比如你要做图片高斯模糊、添加圆角、做度灰处理、圆形图片等等都可以通过Transformation来完成。

转载：https://www.cnblogs.com/jiangzhishan/p/9429154.html
