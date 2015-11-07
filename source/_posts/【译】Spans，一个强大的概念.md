title: 【译】Spans，一个强大的概念
date: 2015-03-04 00:11:25
category: Android
tags: [Span, 文字, 动画, 译文]
thumbnail: http://rocko-blog.qiniudn.com/Spans%EF%BC%8C%E4%B8%80%E4%B8%AA%E5%BC%BA%E5%A4%A7%E7%9A%84%E6%A6%82%E5%BF%B5-27.gif?imageView2/2/w/400/h/300/q/100
toc: true
---

### 前言

原文：[Spans, a Powerful Concept.](http://flavienlaurent.com/blog/2014/01/31/spans/)

最近，我写了一篇关于NewStand app和app上ActionBar的图标的翻转动效的文章。[Cyril Mottier](http://cyrilmottier.com/)建议我采用一个很优雅的方案，即使用Spans去淡入淡出ActionBar的标题。
此外，我一直想尝试所有可用的Sapn色的类型：[ImageSpan](http://developer.android.com/reference/android/text/style/ImageSpan.html)、[BackgroundColorSpan](http://developer.android.com/reference/android/text/style/BackgroundColorSpan.html)等。他们非常简单易用但是（也）没有任何关于它们的文档和详细信息。
因此，在这篇文章中，我将探索在Spans的框架下什么是可以做的，然后，我将会告诉你怎么去进阶使用Spans。
你可以下载和安装[demo程序](https://github.com/flavienlaurent/spans/raw/master/sample.apk)，查看[源码](https://github.com/flavienlaurent/spans)。

<!--more-->

### 框架

#### 层次

主要规则：
- 如果一个Span影响字符级的文本格式，则继承[CharacterStyle](http://developer.android.com/reference/android/text/style/CharacterStyle.html)。
- 如果一个Span影响段落层次的文本格式，则实现[ParagraphStyle](http://developer.android.com/reference/android/text/style/ParagraphStyle.html)
- 如果一个Span修改字符级别的文本外观，则实现[UpdateAppearance](http://developer.android.com/reference/android/text/style/UpdateAppearance.html)
- 如果一个Span修改字符级文本度量|大小，则实现[UpdateLayout](http://developer.android.com/reference/android/text/style/UpdateLayout.html)

它为我们提供了下面这些美丽的类图：
![characterstyle](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-1.png?imageView2/2/w/850/h/600/q/100)
![paragraphstyle](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-2.png?imageView2/2/w/850/h/600/q/100)
![updateappearance](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-3.png?imageView2/2/w/850/h/600/q/100)
![updatelayout](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-4.png)
因为它有一点复杂所以我建议你使用像这样的可视化类图，以充分理解它的层次结构。

### 它是如何工作的?

#### 布局（Layout）

当你给一个TextView设置文本时，它使用基类[布局](http://developer.android.com/reference/android/text/Layout.html)去管理文本的渲染。
布局类包含一个布尔`mSpannedText`:真，当文本是一个[Spanned](http://developer.android.com/reference/android/text/Spanned.html)的实例时（[SpannableString](http://developer.android.com/reference/android/text/SpannableString.html)实现[Spanned](http://developer.android.com/reference/android/text/Spanned.html)）。这个类只处理[ParagraphStyle](http://developer.android.com/reference/android/text/style/ParagraphStyle.html) Spans。
[draw](http://developer.android.com/reference/android/text/Layout.html#draw(android.graphics.Canvas,%20android.graphics.Path,%20android.graphics.Paint,%20int)方法调用了其它两个方法：
- **drawBackground**
对于文本的每一行，如果有一个[LineBackgroundSpan](http://developer.android.com/reference/android/text/style/LineBackgroundSpan.html)用于当前行，[LineBackgroundSpan#drawBackground](http://developer.android.com/reference/android/text/style/LineBackgroundSpan.html#drawBackground(android.graphics.Canvas,%20android.graphics.Paint,%20int,%20int,%20int,%20int,%20int,%20java.lang.CharSequence,%20int,%20int,%20int))方法将被调用。
- **drawText**
对于文本的每一行，它计算[LeadingMarginSpan](http://developer.android.com/reference/android/text/style/LeadingMarginSpan.html)和[LeadingMarginSpan2](http://developer.android.com/reference/android/text/style/LeadingMarginSpan.LeadingMarginSpan2.html)，并调用[LeadingMarginSpan＃drawLeadingMargin](http://developer.android.com/reference/android/text/style/LeadingMarginSpan.html#drawLeadingMargin(android.graphics.Canvas,%20android.graphics.Paint,%20int,%20int,%20int,%20int,%20int,%20java.lang.CharSequence,%20int,%20int,%20boolean,%20android.text.Layout))方法当它是必要的时候。这也用于确定文本对齐。最后，如果当前行是跨行的，布局将调用 TextLine#draw方法（每一行都会创建一个TextLine对象）。

#### 文本行(TextLine)

[android.text.TextLine](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/TextLine.java)的文档这么说：代表一行样式的文字，用于测量视觉顺序和为了渲染。
TextLine类包含3个Spans的集合：

- MetricAffectingSpan set
- CharacterStyle set
- ReplacementSpan set

其中有趣的方法：TextLine#handleRun，这也是所有的Spans用来渲染文本的。相对于Span的类型，TextLine调用：

- [CharacterStyle#updateDrawState](http://flavienlaurent.com/blog/2014/01/31/spans/)方法更改MetricAffectingSpan和CharacterStyle两个Spans的TextPaint配置。
- TextLine#handleReplacement方法处理ReplacementSpan。它调用[Replacement#getSize](http://developer.android.com/reference/android/text/style/ReplacementSpan.html#getSize(android.graphics.Paint,%20java.lang.CharSequence,%20int,%20int,%20android.graphics.Paint.FontMetricsInt))得到replacement的宽度，如果它需要更新字体规格最终会调用[Replacement#draw](http://developer.android.com/reference/android/text/style/ReplacementSpan.html#draw(android.graphics.Canvas,%20java.lang.CharSequence,%20int,%20int,%20float,%20int,%20int,%20int,%20android.graphics.Paint)

#### 字体规格（Font Metrics）

如果你想知道更多什么是字体规格，那么看下面的图解：
![fontmetrics](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-5.png)

### 耍起来

#### BulletSpan
[android.text.style.BulletSpan](http://developer.android.com/reference/android/text/style/BulletSpan.html)
BulletSpan影响段落层次的文本格式。它可以给段落的开始处加上项目符号。
``` java
/**
 * gapWidth:项目符号和文本之间的间隙
 * color: 项目符号的颜色，默认为透明
 */

//创建一个黑色的BulletSpan，间隙为15px
span = new BulletSpan(15, Color.BLACK);
```
![BulletSpan的效果](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-6.png?imageView2/2/w/400/h/300/q/100)

#### QuoteSpan

[android.text.style.QuoteSpan](http://developer.android.com/reference/android/text/style/QuoteSpan.html)
QuoteSpan影响段落层次的文本格式。它可以给一个段落加上垂直的引用线。
``` java
/**
 * color: 垂直的引用线颜色，默认是蓝色
 */

//创建一个红色的引用
span = new QuoteSpan(Color.RED);
```
![QuoteSpan的效果](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-7.png?imageView2/2/w/400/h/300/q/100)

#### AlignmentSpan.Standard

[android.text.style.AlignmentSpan.Standard](http://developer.android.com/reference/android/text/style/AlignmentSpan.Standard.html)
AlignmentSpan.Standard影响段落层次的文本格式。它可以把段落的每一行文本按正常、居中、相反的方式对齐。
``` Java
/**
 * align: 对齐方式
 */

//居中对齐的段落
span = new AlignmentSpan.Standard(Layout.Alignment.ALIGN_CENTER);
```
![AlignmentSpan.Standard的效果](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-8.png?imageView2/2/w/400/h/300/q/100)

#### UnderlineSpan

[android.text.style.UnderlineSpan](http://developer.android.com/reference/android/text/style/UnderlineSpan.html)
UnderlineSpan影响字符级的文本格式。它可以为字符集加上下划线，归功于[Paint#setUnderlineText(true)](http://developer.android.com/reference/android/graphics/Paint.html#setUnderlineText(boolean))。
``` Java
//下划线
span = new UnderlineSpan();
```
![UnderlineSpan效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-9.png?imageView2/2/w/400/h/300/q/100)

#### StrikethroughSpan

[android.text.style.StrikethroughSpan](http://developer.android.com/reference/android/text/style/StrikethroughSpan.html)
StrikethroughSpan影响字符级的文本格式。它可以给字符集加上删除线，归功于[Paint#setStrikeThruText(true))](http://developer.android.com/reference/android/graphics/Paint.html#setStrikeThruText(boolean)。
``` Java
//删除线
span = new StrikethroughSpan();
```
![StrikethroughSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-10.png?imageView2/2/w/400/h/300/q/100)

#### SubscriptSpan

[android.text.style.SubscriptSpan](http://developer.android.com/reference/android/text/style/SubscriptSpan.html)
SubscriptSpan影响字符级的文本格式，它可以通过减小[TextPaint#baselineShift](http://developer.android.com/reference/android/text/TextPaint.html#baselineShift)给字符集加下标。
``` Java
//下标
span = new SubscriptSpan();
```
![SubscriptSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-11.png?imageView2/2/w/400/h/300/q/100)

#### SuperscriptSpan

[android.text.style.SuperscriptSpan](http://developer.android.com/reference/android/text/style/SuperscriptSpan.html)
SuperscriptSpan影响字符级的文本格式。它可以通过增加[TextPaint#baselineShift ](http://developer.android.com/reference/android/text/TextPaint.html#baselineShift)给字符集加上标。
``` Java
//上标
span = new SuperscriptSpan();
```
![SuperscriptSpan效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-12.png?imageView2/2/w/400/h/300/q/100)

#### BackgroundColorSpan

[android.text.style.BackgroundColorSpan](http://developer.android.com/reference/android/text/style/BackgroundColorSpan.html)
BackgroundColorSpan影响字符级的文本格式。它可以给字符集加上背景颜色。
``` Java
/**
 * color: 背景颜色
 */

//设置字符背景颜色
span = new BackgroundColorSpan(Color.GREEN);
```
![BackgroundColorSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-13.png?imageView2/2/w/400/h/300/q/100)

#### ForegroundColorSpan

[android.text.style.ForegroundColorSpan](http://developer.android.com/reference/android/text/style/ForegroundColorSpan.html)
ForegroundColorSpan影响字符级的文本格式，它可以设置字符集的前景颜色也即文字颜色。
``` Java
/**
 * color: 前景颜色
 */

//设置红色的前景
span = new ForegroundColorSpan(Color.RED);
```
![ForegroundColorSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-14.png?imageView2/2/w/400/h/300/q/100)

#### ImageSpan

[android.text.style.ImageSpan](http://developer.android.com/reference/android/text/style/ImageSpan.html)
ImageSpan影响字符级的文本格式。它可以生成图像字符。这是为数不多的文档齐全的Span所以enjoy it!
``` Java
/**
 * Context: 上下文
 * resourceId: 图像资源id
 */

//用一个小图像代替字符
span = new ImageSpan(this, R.drawable.pic1_small);
```
![ImageSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-15.png?imageView2/2/w/400/h/300/q/100)

#### StyleSpan

[android.text.style.StyleSpan](http://developer.android.com/reference/android/text/style/StyleSpan.html)
StyleSpan影响字符级的文本格式，它可以给字符集设置样式（blod、italic、normal）。
``` Java
//设置bold+italic的字符样式
span = new StyleSpan(Typeface.BOLD | Typeface.ITALIC);
```
![StyleSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-16.png?imageView2/2/w/400/h/300/q/100)

#### TypefaceSpan

[android.text.style.TypefaceSpan](http://developer.android.com/reference/android/text/style/TypefaceSpan.html)
TypefaceSpan影响字符级的文本格式。它可以给字符设置字体集（monospace、serif等）。
``` Java
//设置serif family
span = new TypefaceSpan("serif");
```
![TypefaceSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-17.png?imageView2/2/w/400/h/300/q/100)

#### TextAppearanceSpan

[android.text.style.TextAppearanceSpan](http://developer.android.com/reference/android/text/style/TextAppearanceSpan.html)
TextAppearanceSpan影响字符级的文本格式。它可以给字符集设置外观（appearance）。
``` Java
/**
 * TextAppearanceSpan(Context context, int appearance, int colorList)
 * 		context: 上下文
 *		appearance：appearance资源id（例如：android.R.style.TextAppearance_Small）
 *		colorList：文本的颜色资源id（例如：android.R.styleable.Theme_textColorPrimary）
 *
 * TextAppearanceSpan(String family, int style, int size, ColorStateList color, ColorStateList linkColor)
 *		family：字体family
 *		style：描述样式（例如：android.graphics.Typeface）
 *		size：文字大小
 *		color：文字颜色
 *		linkColor：连接文本的颜色
 */

//设置serif family
span = new TextAppearanceSpan(this/*a context*/, R.style.SpecialTextAppearance);
```
``` xml
<-- style.xml -->
<style name="SpecialTextAppearance" parent="@android:style/TextAppearance">
    <item name="android:textColor">@color/color1</item>
    <item name="android:textColorHighlight">@color/color2</item>
    <item name="android:textColorHint">@color/color3</item>
    <item name="android:textColorLink">@color/color4</item>
    <item name="android:textSize">28sp</item>
    <item name="android:textStyle">italic</item>
</style>
```
![TextAppearanceSpan效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-18.png?imageView2/2/w/400/h/300/q/100)

#### AbsoluteSizeSpan

[android.text.style.AbsoluteSizeSpan](http://developer.android.com/reference/android/text/style/AbsoluteSizeSpan.html)
AbsoluteSizeSpan影响字符级的文本格式。它可以设置一个字符集的绝对文字大小。
``` Java
/**
 * size: 大小
 * dip: false，size单位为px，true，size单位为dip（默认为false）。
 */

//设置文字大小为24dp
span = new AbsoluteSizeSpan(24, true);
```
![AbsoluteSizeSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-19.png?imageView2/2/w/400/h/300/q/100)

#### RelativeSizeSpan

[android.text.style.RelativeSizeSpan](http://developer.android.com/reference/android/text/style/RelativeSizeSpan.html)
RelativeSizeSpan影响字符水平的文本格式。它可以设置字符集的文本大小。
``` Java
//设置文字大小为大2倍
span = new RelativeSizeSpan(2.0f);
```
![RelativeSizeSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-20.png?imageView2/2/w/400/h/300/q/100)

#### ScaleXSpan

[android.text.style.ScaleXSpan](http://developer.android.com/reference/android/text/style/ScaleXSpan.html)
ScaleXSpan印象字符集的文本格式。它可以在x轴方向上缩放字符集。
``` Java
//设置水平方向上放大3倍
span = new ScaleXSpan(3.0f);
```
![ScaleXSpan的效果图](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-21.png?imageView2/2/w/400/h/300/q/100)

#### MaskFilterSpan

[android.text.style.MaskFilterSpan](http://developer.android.com/reference/android/text/style/MaskFilterSpan.html)
MaskFilterSpan影响字符集文本格式。它可以给字符集设置[android.graphics.MaskFilter](http://developer.android.com/reference/android/graphics/MaskFilter.html)。
**警告：BlurMaskFilter不支持硬件加速**
``` Java
//模糊字符集
span = new MaskFilterSpan(new BlurMaskFilter(density*2, BlurMaskFilter.Blur.NORMAL));
//浮雕字符集
span = new MaskFilterSpan(new EmbossMaskFilter(new float[] { 1, 1, 1 }, 0.4f, 6, 3.5f));
```
![MaskFilterSpan的效果图: BlurMaskFilter](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-22.png?imageView2/2/w/400/h/300/q/100)
![MaskFilterSpan的效果图: EmbossMaskFilter](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-23.png?imageView2/2/w/400/h/300/q/100)

### Spans进阶

#### 前景色（文字颜色）动画

![前景色（文字颜色）动画](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-24.gif?imageView2/2/w/400/h/300/q/100)
ForegroundColorSpan为只读。这意味实例化之后着你不能改变你不能改变前景色。所以，要做的第一件事就是编写一个MutableForegroundColorSpan。
*MutableForegroundColorSpan.java*
``` Java
public class MutableForegroundColorSpan extends ForegroundColorSpan
{

    private int mAlpha = 255;
    private int mForegroundColor;

    public MutableForegroundColorSpan(int alpha, int color)
    {
        super(color);
        mAlpha = alpha;
        mForegroundColor = color;
    }

    public MutableForegroundColorSpan(Parcel src)
    {
        super(src);
        mForegroundColor = src.readInt();
        mAlpha = src.readInt();
    }

    public void writeToParcel(Parcel dest, int flags)
    {
        super.writeToParcel(dest, flags);
        dest.writeInt(mForegroundColor);
        dest.writeFloat(mAlpha);
    }

    @Override
    public void updateDrawState(TextPaint ds)
    {
        ds.setColor(getForegroundColor());
    }

    /**
     * @param alpha from 0 to 255
     */
    public void setAlpha(int alpha)
    {
        mAlpha = alpha;
    }

    public void setForegroundColor(int foregroundColor)
    {
        mForegroundColor = foregroundColor;
    }

    public float getAlpha()
    {
        return mAlpha;
    }

    @Override
    public int getForegroundColor()
    {
        return Color.argb(mAlpha, Color.red(mForegroundColor), Color.green(mForegroundColor), Color.blue(mForegroundColor));
    }
}
```
现在，我们可以在同一个实例改变透明度和前景色了。但是，当你设置这些属性，它并不会刷新视图，你必须通过重新设置SpannableString才能刷新视图。
``` Java
MutableForegroundColorSpan span = new MutableForegroundColorSpan(255, Color.BLACK);
spannableString.setSpan(span, 0, text.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
textView.setText(spannableString);
//黑色完全不透明（译者注：上面代码的效果）
span.setAlpha(100);
span.setForegroundColor(Color.RED);
//到这一步文字没有变化
textView.setText(spannableString);
//最后，文字变为红色和透明
```
现在我们要前景色的动画。我们可以自定义[android.util.Property](http://developer.android.com/reference/android/util/Property.html)。
``` Java
private static final Property<MutableForegroundColorSpan, Integer> MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY =
new Property<MutableForegroundColorSpan, Integer>(Integer.class, "MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY") {

    @Override
    public void set(MutableForegroundColorSpan span, Integer value) {
        span.setForegroundColor(value);
    }

    @Override
    public Integer get(MutableForegroundColorSpan span) {
        return span.getForegroundColor();
    }
};
```
最后，我们使用属性动画（[ObjectAnimator](http://developer.android.com/reference/android/animation/ObjectAnimator.html)）让自定义属性动起来。不要忘记更新视图。
``` Java
MutableForegroundColorSpan span = new MutableForegroundColorSpan(255, Color.BLACK);
mSpannableString.setSpan(span, 0, text.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
ObjectAnimator objectAnimator = ObjectAnimator.ofInt(span, MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY, Color.BLACK, Color.RED);
objectAnimator.setEvaluator(new ArgbEvaluator());
objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        //refresh
        mText.setText(mSpannableString);
    }
});
objectAnimator.start();
```

#### ActionBar"烟火"

![ActionBar"烟火"](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-25.gif?imageView2/2/w/400/h/200/q/100)
"烟火"动画是让文字随机淡入。首先，把文字切断成多个spans（例如，一个character的span），淡入spans后再淡入其它的spans。用前面介绍的MutableForegroundColorSpan，我们将创建一组特殊的span对象。在span组调用对应的setAlpha方法，我们随机设置每个span的透明度。
``` Java
private static final class FireworksSpanGroup {
        private final float mAlpha;
        private final ArrayList<MutableForegroundColorSpan> mSpans;

        private FireworksSpanGroup(float alpha) {
            mAlpha = alpha;
            mSpans = new ArrayList<MutableForegroundColorSpan>();
        }

        public void addSpan(MutableForegroundColorSpan span) {
            span.setAlpha((int) (mAlpha * 255));
            mSpans.add(span);
        }

        public void init() {
            Collections.shuffle(mSpans);
        }

        public void setAlpha(float alpha) {
            int size = mSpans.size();
            float total = 1.0f * size * alpha;

            for(int index = 0 ; index < size; index++) {
                MutableForegroundColorSpan span = mSpans.get(index);
                if(total >= 1.0f) {
                    span.setAlpha(255);
                    total -= 1.0f;
                } else {
                    span.setAlpha((int) (total * 255));
                    total = 0.0f;
                }
            }
        }

        public float getAlpha() { return mAlpha; }
    }
```
我们创建一个自定义属性动画的属性去更改FireworksSpanGroup的透明度
``` Java
private static final Property<FireworksSpanGroup, Float> FIREWORKS_GROUP_PROGRESS_PROPERTY =
new Property<FireworksSpanGroup, Float>(Float.class, "FIREWORKS_GROUP_PROGRESS_PROPERTY") {

    @Override
    public void set(FireworksSpanGroup spanGroup, Float value) {
        spanGroup.setProgress(value);
    }

    @Override
    public Float get(FireworksSpanGroup spanGroup) {
        return spanGroup.getProgress();
    }
};
```
最后，我们创建span组并使用一个ObjectAnimator给其加上动画。
``` Java
final FireworksSpanGroup spanGroup = new FireworksSpanGroup();
//初始化包含多个spans的grop
//spanGroup.addSpan(span);
//给ActionBar的标题设置spans
//mActionBarTitleSpannableString.setSpan(span, start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
spanGroup.init();
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(spanGroup, FIREWORKS_GROUP_PROGRESS_PROPERTY, 0.0f, 1.0f);
objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener()
{
    @Override
    public void onAnimationUpdate(ValueAnimator animation)
    {
        //更新标题
        setTitle(mActionBarTitleSpannableString);
    }
});
objectAnimator.start();
```

#### 使用自定义的span

在本节中，我们将看到使用自定义span来绘制的方式。这是文本定制很好的方式。
首先，我们要创建一个继承[ReplacementSpan](http://developer.android.com/reference/android/text/style/ReplacementSpan.html)抽象类的自定义Span。
如果你想画一个自定义的背景，你可以实现[LineBackgroundSpan](http://developer.android.com/reference/android/text/style/LineBackgroundSpan.html)
,这是影响段落级的文本格式。
我们必须实现2个方法：

- [getSize](http://developer.android.com/reference/android/text/style/ReplacementSpan.html#getSize(android.graphics.Paint,%20java.lang.CharSequence,%20int,%20int,%20android.graphics.Paint.FontMetricsInt))：这个方法返回新的你更换后的size。
text：Span管理的文本
start：文本开始处
end：文本结尾处
fm：字体规格，**可以为空**
- [draw](http://developer.android.com/reference/android/text/style/ReplacementSpan.html#draw(android.graphics.Canvas,%20java.lang.CharSequence,%20int,%20int,%20float,%20int,%20int,%20int,%20android.graphics.Paint))：可以使用Canvas绘制。
x：绘制文本的x坐标
top：线（line）的顶部（译者注：line的定义参看前面*字体规格*这一节）
y：基线
bottom：线的底部、

让我们看一个例子，画一个包围文本的蓝色矩形。
*FrameSpan.java*
``` Java
@Override
public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fm)
{
    //将返回相对于Paint画笔的文本
    mWidth = (int) paint.measureText(text, start, end);
    return mWidth;
}

@Override
public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint)
{
    //使用自定义的画笔绘制在画布上
    canvas.drawRect(x, top, x + mWidth, bottom, mPaint);
}
```
![自定义Span的效果](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-24.png?imageView2/2/w/400/h/300/q/100)

#### 附加

Sample app包含了一些Spans进阶的例子，如下：
![Progressive blur](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-26.gif?imageView2/2/w/400/h/300/q/100)
![Typewriter](http://rocko-blog.qiniudn.com/Spans，一个强大的概念-27.gif?imageView2/2/w/400/h/300/q/100)

### 总结
在编写这篇文章的过程中，我意识到Spans是真的像Drawable那样强大的，我认为它们还没有被充分运用。文本是一个应用程序的主要内容，它无处不在，所以不要忘记，通过Spans让它变得更具活力和吸引力！