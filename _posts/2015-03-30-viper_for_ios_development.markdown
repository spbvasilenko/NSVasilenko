---
layout: post
title:  "Разработка iOS приложений с VIPER"
date:   2015-03-30 19:47:00
---

<center><img src="http://habrastorage.org/getpro/habr/post_images/26f/487/66c/26f48766c665ffac9b10257b2c454bb4.jpg" alt="image"/></center>

В последнее время я стал много слышать об архитектуре под названием "VIPER". Я настолько заинтересовался этой темой, что решил изучить данную тему поглубже и рассказать о ней подробнее всем остальным.
Статья представляет собой частичный перевод уже имеющейся статьи об архитектуре VIPER на всем известном ресурсе <a href="http://www.objc.io/issue-13/viper.html">objc.io</a>

<b>Что такое VIPER?</b>

VIPER применяется в iOS приложениях, построенных на так называемой <a href="http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html">Clean Architecture</a>. Мир VIPER основан на View, Interactor, Presenter, Entity, и Routing. Clean Architecture делит логику приложения на отдельные уровни ответственности, которые показаны ниже.
<img src="http://habrastorage.org/getpro/habr/post_images/37e/86b/8fb/37e86b8fb8cc2e81e54bf6cb409d638d.jpg" alt="image"/>
Рассмотрим главные части архитектуры подробнее:

<b>View </b>- Отображает то, что ему говорит Presenter и возвращает обратно данные тому же Presenter, введенные пользователем.  

<b>Interactor</b>- Включает в себя всю бизнес-логику.

<b>Presenter</b>- Включает в себя View логику подготовки данных для отображения и реагирует на обратные действия пользователя. 

<b>Entity</b>- Включает в себя базовую модель объектов, которые использует Interactor.

<b>Routing</b>- Отвечает за логику навигации, какие экраны показывать и в каком порядке.

Это разделение так же соответствует принципу "единоличная ответственность" или <a href="http://www.objectmentor.com/resources/articles/srp.pdf">Single Responsibility Principle</a>. То есть Interactor можно представить как бизнес-аналитика, Presenter взаимодействует с дизайнером, а View отвечает за визуального-дизайнера.

Ниже приведена схема, где показано как отдельные части архитектуры связаны между собой:
<img src="http://habrastorage.org/getpro/habr/post_images/6ec/b19/1c3/6ecb191c3ea7a1136b1c9ace02a0e73f.png" alt="image"/>
Теперь попробуем рассмотреть главные части архитектуры на реальном примере в тестовом проекте, который вы можете скачать по следующей <a href="https://github.com/spbvasilenko/ExampleVIPER">ссылке</a>:

<b>Interactor</b>

Представляет собой как единственный экземпляр в iOS приложении. Он включает в себя работу бизнес-логику манипулирования моделей объектов и не выходит за другие рамки задач. Работать с Interactor довольно удобно, потому что он может быть использован как и в iOS приложении, так и в OS X. 

{% highlight objectivec %}
- (void)findUpcomingItems
{
    __weak typeof(self) welf = self;
    NSDate* today = [self.clock today];
    NSDate* endOfNextWeek = [[NSCalendar currentCalendar] dateForEndOfFollowingWeekWithDate:today];
    [self.dataManager todoItemsBetweenStartDate:today endDate:endOfNextWeek completionBlock:^(NSArray* todoItems) {
        [welf.output foundUpcomingItems:[welf upcomingItemsFromToDoItems:todoItems]];
    }];
}
{% endhighlight %}

<b>Entity</b>

Объекты моделей, которые обрабатывает Interactor. Сущности могут взаимодействовать только с Interactor. Interactor никогда не передает сущности уровню отображения (т.е. Presenter). Сущности так же склонны к термину PONSOs, и если вы используете Core Data, вы нуждаетесь в управляемых объектах ваших данных. Interactors не должны работать с NSManagedObjects. 

{% highlight objectivec %}
@interface VTDTodoItem : NSObject

@property (nonatomic, strong)   NSDate*     dueDate;
@property (nonatomic, copy)     NSString*   name;

+ (instancetype)todoItemWithDueDate:(NSDate*)dueDate name:(NSString*)name;

@end
{% endhighlight %}

<b>Presenter</b>

Presenter так же придерживается POSNO и содержит логику управления UI. Он знает когда показывать пользовательский интерфейс. Собирает входящие сигналы жестов пользователя и может обновлять UI, так же посылает реквесты Interactor'у. Presenter так же получает результаты из Interactor и преобразует результаты введенные из различных форм UI. Entities никогда не передаются из Interactor Presenter'у. Presenter может только подготавливать данные для отображения в UI.

{% highlight objectivec %}
- (void)addNewEntry
{
    [self.listWireframe presentAddInterface];
}

- (void)foundUpcomingItems:(NSArray*)upcomingItems
{
    if ([upcomingItems count] == 0)
    {
        [self.userInterface showNoContentMessage];
    }
    else
    {
        [self updateUserInterfaceWithUpcomingItems:upcomingItems];
    }
}
{% endhighlight %}

<b>View</b>

View пассивен. Он ожидает пока Presenter ему не отдаст данные для отображения, при этом, никогда не просит его об этом. Методы, определенные во View должны позволять Presenter общаться с ним на уровне высокой абстракции. Presenter не знает об существовании таких элементов UI как UILabel, UIButton и так далее. Presenter знает только контенте и когда он должен быть отображен, а View определяет как его отобразить.

{% highlight objectivec %}
@protocol VTDAddViewInterface <NSObject>

- (void)setEntryName:(NSString *)name;
- (void)setEntryDueDate:(NSDate *)date;

@end
{% endhighlight %}

<b>Routing</b>

В VIPER ответственность Routing разделяется на два объекта: Presenter и wireframe. Wireframe - объект, который владеет UIWindow, UINavigationController, UIViewController и так далее. Он отвечает за создание View/ViewController и установку окна. 
Так как Presenter содержит логику на реакции пользователя, Presenter знает когда переходить на другой экран и с какого экрана. Между тем, wireframe знает как сделать эту навигацию. Значит, что Presenter будет использовать wireframe для выполнения навигаций. 

{% highlight objectivec %}
@implementation VTDAddWireframe

- (void)presentAddInterfaceFromViewController:(UIViewController *)viewController 
{
    VTDAddViewController *addViewController = [self addViewController];
    addViewController.eventHandler = self.addPresenter;
    addViewController.modalPresentationStyle = UIModalPresentationCustom;
    addViewController.transitioningDelegate = self;

    [viewController presentViewController:addViewController animated:YES completion:nil];

    self.presentedViewController = viewController;
}

#pragma mark - UIViewControllerTransitioningDelegate Methods

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed 
{
    return [[VTDAddDismissalTransition alloc] init];
}

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented
                                                                  presentingController:(UIViewController *)presenting
                                                                      sourceController:(UIViewController *)source 
{
    return [[VTDAddPresentationTransition alloc] init];
}

@end
{% endhighlight %}

<b>Подытожим</b>

Надеюсь, я помог вам немного больше разобраться в этой архитектуре и почему бы не испробовать ее в следующем вашем приложении, которое вы собираетесь написать?
Всем спасибо за внимание!
