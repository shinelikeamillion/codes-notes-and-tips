## ApiDemo
1,Animate background color --/animation/BouncingBalls
```
ValueAnimator colorAnim = ObjectAnimator.ofInt(this, "backgroundColor", RED, BLUE);
colorAnim.setDuration(3000); // 
colorAnim.setEvaluator(new ArgbEvaluator());
colorAnim.setRepeatCount(ValueAnimator.INFINITE);
colorAnim.setRepeatMode(ValueAnimator.REVERSE);
colorAnim.start();
```

2,NotifyingController --/app/NotifyingServerce
```
ConditionVariable mCondition = new CondtitionVariable();
提供三个方法，block(),open(),close();阻塞/释放/关闭
```