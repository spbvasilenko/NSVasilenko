---
layout: post
title: LLDB. Использование expr и переменных
excerpt: "Давненько не писал ничего интересного и решил исправить ситуацию. Люблю делиться тем, чего сам совсем недавно узнал и попробовал на практике. И сегодня мы поговорим об LLDB в Xcode, а в частности про команду expr."
categories: [lldb, debugging]
comments: true
---

Давненько не писал ничего интересного и решил исправить ситуацию. Люблю делиться тем, чего сам совсем недавно узнал и попробовал на практике. И сегодня мы поговорим об LLDB в Xcode, а в частности про команду expr.

Как часто мы тестируем какие-либо кейсы бизнес-логики в наших приложениях? Как часто мы хардкодим ветки if, только для того, чтобы зайти в эту ветку и посмотреть поведение приложения? И как часто, мы перекомпилируем наш проект? Очень огромное количество раз!

В этом случае нам может помочь как раз команда expr, если поставить breakepoint в нужном вам месте кода. Приведу реальный кейс: 

В контроллере у нас лежит какая-либо UIView, при переходе в этот контроллер, мы хотим посмотреть на UIView в разных состояниях, к примеру разного цвета. Для того, чтобы это сделать, вы, наверное, каждый раз меняли цвет у view и пересобирали проект. Но можно это сделать гораздо быстрее и проще. Достаточно поставить брейкпоинт во viewDidLoad и выполнить в LLDB консоли следущую команду:
{% highlight objective-c %}
expr self.view.backgroundColor = [UIColor blueColor];
{% endhighlight %}
затем
{% highlight objective-c %}
continue
{% endhighlight %}
В конечном результате мы меняем цвет нашей view буквально на лету! Хотя, буквально сейчас узнал еще от друга еще один способ менять свойства UI эелементов. Это найти во UI дебаггере нашу view, скопировать ее адрес и уже используя адрес модифицировать вьюху. Но команда expr умеет гораздо больше!

Двигаемся дальше.
Приведу еще один пример, нечто гораздо больше. Снова ставим брэйкпоинт во viewDidLoad в каком либо контроллере и выполним команду:
{% highlight objective-c %}
[self prepareForSegue:<#(nonnull UIStoryboardSegue *)#> sender:<#(nullable id)#>]
{% endhighlight %}
затем
{% highlight objective-c %}
continue
{% endhighlight %}
После этого нас перекинет на тот экран, который мы указали :) Впечатляет, не правда ли?

Следующий пример. У нас есть несколько кейсов, выполняющих разные действия, точнее какой-нибудь switch или ветки с if/else.
Опять же, для того, чтобы зайти насильно в какой либо case или ветку if мы будем хардкодить в коде и пересобирать проект? Не нужно!
Снова ставим breakpoint, допустим, перед выполнением swith или начало ветки if. 
{% highlight objective-c %}
if (self.dataIsReady) {
    // do something        
  } else {
    // do something                
  }
{% endhighlight %}
Представим, что dataIsReady у нас будет NO и мы это знаем.
Выполним следущее после того как мы попадем в брейкпоинт:
{% highlight objective-c %}
expr self.dataIsReady = YES
{% endhighlight %}
затем
{% highlight objective-c %}
continue
{% endhighlight %}
После этого, мы переходим в ту ветку if, которая нам нужна :)

Приведу следущую возможность команды expr. Выполнение методов и создание переменных. 

Выводим в лог сообщение:
{% highlight objective-c %}
expr (void) NSLog(@"hello world!")
{% endhighlight %}

Проверяем массив на содержание объекта:
{% highlight objective-c %}
expr (BOOL) [self.myArray containsObject:@“CarKeys"]
{% endhighlight %}

Распечатываем frame нашей view:
{% highlight objective-c %}
expr -- (CGRect) [self.view frame]
{% endhighlight %}
Стоит отметить "--" указывает, что мы хотим выполнить фактическую команду.

Создание переменных осуществляется следующим способом:
{% highlight objective-c %}
expr int $meaningOfLife = 42
expr 100 + $meaningOfLife
(int) $0 = 142 
(lldb) p $0 + 200 (int) 
$1 = 24
{% endhighlight %}

Ключевым символом создания переменной является "$".

И последний пример. Пишем и выполняем код на лету:
{% highlight objective-c %}
(lldb) expr NSString * $json = [self fetchRemoteData]; 
(lldb) expr NSData * $data = [$json dataUsingEncoding:4]
(lldb) expr NSDictionary * $parsedJson = [NSJSONSerialization JSONObjectWithData: $data options:0 error:NULL];
(lldb) po parsedData
{ username : Brian, password : 1234 }
{% endhighlight %}

На этом все. Надеюсь вам пригодится данная фича LLDB и вы будете ее активно использовать. Спасибо за внимание! :)
