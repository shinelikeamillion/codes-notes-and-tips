```
<shape>
  <!-- ʵ�� -->
  <solid android:color="#ff9d77"/>

  <!-- ���� -->
  <gradient
    android:startColor="#ff8c00"
    android:endColor="#FFFFFF"
    android:angle="270" />

  <!-- ��� -->
  <stroke
    android:width="2dp"
    android:color="#dcdcdc" />

  <!-- Բ�� -->
  <corners
    android:radius="2dp" />

</shape>
```
angle��ָ����ĽǶȣ�����Ϊ45����������Ĭ�Ͻ���ģʽ��android:type="linear"��,�����Խ���
������Ϊradial�����򽥱䣬��Ҫָ���뾶��android��gradientRadius="50".

corners���Ū������ dashWith:һ�����ߵĿ�ȣ�dashGap:��ʾ�����ļ��