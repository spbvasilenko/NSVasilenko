---
layout: post
title:  "История о том, как Apple учила меня делать качественный продукт"
date:   2015-06-25 14:00:00
categories: ios
---

Всем привет! На днях потребовалось написать класс, который бы умел выполнять анимации в сериальном порядке, которые мы бы передавали в массиве. Так же требовалось, чтобы по окончанию всех анимаций был общий callback блок. Хотелось бы поделиться с вами получившимся результатом.



Во-первых, какие свойства должен уметь класс, который выполняет анимации? Я обозначил для себя несколько свойств, которые определил в интерфейсе класса:

{% highlight objective-c %}
@property (nonatomic, assign) NSTimeInterval duration;
@property (nonatomic, assign) NSTimeInterval delay;
{% endhighlight %}

Собственно, я посчитал, что этих свойств вполне достаточно для работы класса.

Во-вторых, я обозначил два метода:

Метод инициализатор: 

{% highlight objective-c %}
- (instancetype)initWithDuration:(NSTimeInterval)duration
                           delay:(NSTimeInterval)delay;
{% endhighlight %}

И метод, в который бы мы передавали массив с блоками анимаций и completionBlock:

{% highlight objective-c %}
- (void)doAnimations:(NSArray *)animations completion:(CompletionBlock)completion;
{% endhighlight %}

Перейдем к реализации класса!

С методом инициализации в принципе все довольно ожидаемо:

{% highlight objective-c %}
- (instancetype)initWithDuration:(NSTimeInterval)duration
                           delay:(NSTimeInterval)delay
{
    if (self = [super init]) {
        _duration = duration;
        _delay = delay;
    }
    return self;
}
{% endhighlight %}

И наконец, сам метод, который выполняет анимации:

{% highlight objective-c %}
AnimationBlock animationsBlock = ^{
//Перебираем массив с блоками анимаций. Используем метод addKeyframeWithRelativeStartTime, для того, чтобы анимации выполнялись в нужном нам порядке по времени.
        [animations enumerateObjectsUsingBlock:^(AnimationBlock animation, NSUInteger index, BOOL *stop) {
            [UIView addKeyframeWithRelativeStartTime:index/(CGFloat)animations.count
                                    relativeDuration:1/(CGFloat)animations.count
                                          animations:animation];
        }];
    };
//В конце передаем блок с нашими анимациями в метод animateKeyframesWithDuration, где duration мы рассчитываем как длительность анимации умноженное на количество переданных анимаций. И в итоге получаем конечный callback блок.
    [UIView animateKeyframesWithDuration:self.duration * animations.count
                                   delay:self.delay
                                 options:UIViewKeyframeAnimationOptionCalculationModeLinear | UIViewAnimationOptionCurveLinear
                              animations:animationsBlock
                              completion:completion];
{% endhighlight %}

На этом все! Возможно есть еще варианты решения данной задачи, но долгими поисками, я для себя нашел именно такое.
Всем спасибо за внимание!
