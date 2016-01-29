---
layout: post
title:  "ReactiveCocoa. Concurrency. Multithreading."
date:   2016-01-30 00:00:00
categories: ios
---

Сегодня хотелось бы поговорить о работе с потоками в ReactiveCocoa. Я не буду вдаваться в подробности основ фреймворка и полагаю, что вы уже знакомы с базовыми принципами реактивного программирования в iOS. 

Работа с потоками в мобильном приложении наиважнейшая тема и это все знают. Стандарнтыми инструментами для этого, являются GCD или NSOperation. Но при использовании ReactiveCocoa в нашем проекте, все становится несколько иначе. Нет, вам никто не запрещает использовать стандартные инструменты, но при работе с RACSignal/RACCommand и etc., нам нужно уметь управлять, в каком потоке должен запуститься тот или инной сигнал, иначе получится глубокий хаос. 

Для работы с многопоточностью в ReactiveCocoa существует класс RACSCheduler. По сути, это обертка над GCD... и имеет те же самые приоритеты потоков, что и у GCD: 
{% highlight objective-c %}
typedef enum : long {
	RACSchedulerPriorityHigh = DISPATCH_QUEUE_PRIORITY_HIGH,
	RACSchedulerPriorityDefault = DISPATCH_QUEUE_PRIORITY_DEFAULT,
	RACSchedulerPriorityLow = DISPATCH_QUEUE_PRIORITY_LOW,
	RACSchedulerPriorityBackground = DISPATCH_QUEUE_PRIORITY_BACKGROUND,
} RACSchedulerPriority;
{% endhighlight %}

Рассмотрим основные методы RACScheduler, которые могут нам понадобиться при работе с ним:

Из названия, в принципе, становится ясно, что нам возвращается RACScheduler, который будет выполнять работу в главном потоке.
{% highlight objective-c %}
+ (RACScheduler *)mainThreadScheduler;
{% endhighlight %}
В данном случае, нам возвращается RACSCheduler с указанным приоритетом и уже не в главном потоке.
{% highlight objective-c %}
+ (RACScheduler *)schedulerWithPriority:(RACSchedulerPriority)priority;
{% endhighlight %}
Возвращает RACScheduler c приоритетом RACSchedulerPriorityDefault.
{% highlight objective-c %}
+ (RACScheduler *)scheduler;
{% endhighlight %}
Возвращает текущий RACScheduler из текущего NSThread. 
{% highlight objective-c %}
+ (RACScheduler *)currentScheduler;
{% endhighlight %}
Блок, который RACSCheduler может выполнить где угодно. И к этому мы еще вернемся.
{% highlight objective-c %}
- (RACDisposable *)schedule:(void (^)(void))block;
{% endhighlight %}

Далее приведу основные функции для RACSignal, которые могут использоваться нами для управления многопоточностью:
Данный метод RACSignal, говорит о том, что блоки получения данных subscribeNext/doNext/subscribeError/etc. будут выполняться в том RACSCheduler, который мы вернем.
{% highlight objective-c %}
- (RACSignal *)deliverOn:(RACScheduler *)scheduler
{% endhighlight %}
Данный метод RACSignal, говорит о том, в каком RACScheduler будет выполняться блок, созданный при создании подписки (если мы говорим про ReactiveCocoa, то это: +[RACSignal createSignal:])
{% highlight objective-c %}
- (RACSignal *)subscribeOn:(RACScheduler *)scheduler
{% endhighlight %}

Приведу два коротких примера и мы на этом закончим

Создадим простой сигнал:
{% highlight objective-c %}
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id <RACSubscriber> subscriber) {
// block executes on other thread with default priority
        for (NSInteger i = 0; i < 5000; i++) {
            NSLog(@"LOL");
            if (i == 5000) {
            [subscriber sendNext:@(YES)];
            }
        }
        return nil;
    }];
{% endhighlight %}

Очевидно, что при создании подписки на данный сигнал, пока цикл не закончится, то мы не получим нам не отправится ни единого значения. Для кого-то это может быть критичным. У кого-то, код выполняющийся в этом блоке, будет довольно ресурсоемким. Попробуем разнести по тредам.

Создадим подписку на сигнал и укажем сигналу subscribeOn/deliverOn
{% highlight objective-c %}
[[[signal subscribeOn:[RACScheduler scheduler]]
                deliverOn:[RACScheduler mainThreadScheduler]] subscribeNext:^(id x) {
                // block executes on main thread
}];
{% endhighlight %}

В данном случае, как видно по комментариям, значения мы будем получать в главном потоке, где можно, к примеру, обновлять UI. А в блоке создании подписки код будет выполняться в другом потоке, чем поможет снизить потребление памяти в главном потоке. Все очевидно и просто :)

И последний пример, я покажу вам как из бэкграунд потока запустить код в главном потоке. 
С GCD это выглядело бы как все уже знают следующим образом:
{% highlight objective-c %}
dispatch_async(dispatch_get_global_queue(0, DISPATCH_QUEUE_PRIORITY_DEFAULT), ^{
        // do something
        dispatch_async(dispatch_get_main_queue(), ^{
            // do something
        });
    });
{% endhighlight %}

И как это можно реализовать с RACSheduler:
Как мы помним, при создании подписки на этот сигнал, мы указали, что он будет выполняться не в главном потоке. Но что делать, если в каком то месте, нам все понадобиться выполнить часть кода на главном потоке? Очень просто :) Здесь нам поможет - (RACDisposable *)schedule:(void (^)(void))block;

{% highlight objective-c %}
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id <RACSubscriber> subscriber) {
// block executes on other thread with default priority
        for (NSInteger i = 0; i < 5000; i++) {
            NSLog(@"LOL");
            if (i == 5000) {
            [subscriber sendNext:@(YES)];
            }
        }
        [[RACScheduler mainThreadScheduler] schedule:^(void v) {
            // do something on main thread
        }];
        return nil;
    }];
{% endhighlight %}

На этом все :) Спасибо большое за внимание!
