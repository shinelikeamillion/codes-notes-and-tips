```
<shape>
  <!-- 实心 -->
  <solid android:color="#ff9d77"/>

  <!-- 渐变 -->
  <gradient
    android:startColor="#ff8c00"
    android:endColor="#FFFFFF"
    android:angle="270" />

  <!-- 描边 -->
  <stroke
    android:width="2dp"
    android:color="#dcdcdc" />

  <!-- 圆角 -->
  <corners
    android:radius="2dp" />

</shape>
```
angle是指渐变的角度，必须为45的整数倍，默认渐变模式（android:type="linear"）,即线性渐变
可设置为radial，径向渐变，需要指定半径，android：gradientRadius="50".

corners描边弄成虚线 dashWith:一条横线的宽度，dashGap:表示隔开的间距