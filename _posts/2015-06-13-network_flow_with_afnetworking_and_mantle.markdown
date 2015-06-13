---
layout: post
title:  "Сетевой слой во главе с тандемом AFNetworking & Mantle"
date:   2015-06-13 21:55:00
categories: ios
---

Всем привет! 

Недавно, в обучающих целях, в моей компании мне дали задание попробовать написать максимально гибкий сетевой слой для API Инстаграмма. И теперь я хотел бы поделиться с Вами своим результатом, проделанным в несколько дней. Вероятно, глазами другими, написанный вариант будет выглядеть не таким уж хорошим и гибким, но мне понравилось то, до чего я дошел в конечном результате.

Начнем с того, что я сделал выбор в пользу замечательных AFNetworking и Mantle. Почему? Ответ очевиден: AFNetworking умеет правильно составлять NSURLRequest'ы с нужными нами параметрами и хэдэрами, а так же парсить ответ от сервера в правильный JSON. Mantle же тоже делает свое дело, парсит JSON в нужную нам модель класса. Это действительно волшебный тандем "AFNetworking & Mantle", который глупо не использовать.

Какие главные цели когда мы пишем свой сетевой слой? Я хочу чтобы мой сетевой слой был эластичным, простым, с высокой сочетаемостью и гибким.

Поехали!

Структура слоя получилась следующая: 

1. VASResourceManager - менеджер, который отдает нам конечный ответ относительно конкретного запроса с нашими параметрами и методом, с ним будут работать непосредственно вью модели. 
2. VASNetworkRequestManager - умеет посылать запросы, который в качестве аргументов принимает наши параметры.

Помощниками Network Request Manager'a являются классы VASOperationManager, родителем которого является AFRequestOperationManager и VASJSONResponseSerializer, тоже родителем которого является AFJSONResponseSerializer. 

Вероятно, было бы удобно, получать ответ от реквестов сразу в виде модели какого-либо класса? Мы вызываем метод, передаем ему аргументы с параметрами запроса, а так же передаем класс модели, в виде которой мы хотели бы получить уже готовый ответ. 

В этом нам поможет AFJSONResponseSerializer, который собственно парсит ответ от сервера в JSON. Все, что нам потребуется, так это засабкласиться от данного класса, написать свой метод для инициализации, где в качестве аргумента мы укажем класс нашей модели, и дописать свою логику в следующем методе: 
<source lang="Objective C">- (id)responseObjectForResponse:(NSURLResponse *)response data:(NSData *)data error:(NSError *__autoreleasing *)error</source>

Инициализатор класса будет следующим:
{% highlight objectivec %}
#import "AFURLResponseSerialization.h"

@interface VASJSONResponseSerializer : AFJSONResponseSerializer

- (instancetype)initWithResultClass:(Class)resultClass;

@end
{% endhighlight %}

Теперь перейдем в файл имплементации и допишем нашу логику для парсинга JSON в готовую модель класса с помощью Mantle:
{% highlight objectivec %}
#import "VASJSONResponseSerializer.h"

@interface VASJSONResponseSerializer()

@property (nonatomic, strong) Class resultClass;
@property (nonatomic, strong) id resultResponseObject;

@end

@implementation VASJSONResponseSerializer

- (instancetype)initWithResultClass:(Class)resultClass
{
    if (self = [super init]) {
        self.resultClass = resultClass;
        self.acceptableContentTypes = [NSSet setWithObjects:@"application/json", nil];
    }
    return self;
}

- (id)responseObjectForResponse:(NSURLResponse *)response data:(NSData *)data error:(NSError *__autoreleasing *)error
{
// Используем super метод класса, который возвращает нам готовый JSON
    id responseObject = [super responseObjectForResponse:response data:data error:error];
    
// Если мы передали класс модели, то парсим JSON в модель, иначе возвращаем обычный чистый JSON
    if (self.resultClass)
    {
        if ([responseObject[@"data"] isKindOfClass:[NSArray class]])
        {
            self.resultResponseObject = [MTLJSONAdapter modelsOfClass:self.resultClass
                                                        fromJSONArray:responseObject[@"data"]
                                                                error:NULL];
        }
        else if ([responseObject[@"data"] isKindOfClass:[NSDictionary class]])
        {
            self.resultResponseObject = [MTLJSONAdapter modelOfClass:self.resultClass
                                                  fromJSONDictionary:responseObject[@"data"]
                                                               error:NULL];
        }
    }
    else
    {
        return responseObject;
    }
    
    return self.resultResponseObject;
}

@end
{% endhighlight %}

На этом завершается работа с данным классом.

Перейдем к следующему классу - VASOperationManager. Данный класс менеджер будет уметь возвращать AFHTTPRequestOperation в методе, в который мы передадим параметры для запроса к серверу. Так же нужно добавить свой инициализатор.

Код заголовочного файла: 

{% highlight objectivec %}
#import "AFHTTPRequestOperationManager.h"

typedef void(^OperationManagerCompletionBlockWithSuccess)(AFHTTPRequestOperation *operation, id responseObject);
typedef void(^OperationManagerCompletionBlockWithFailure)(AFHTTPRequestOperation *operation, NSError *error);

@interface VASOperationManager : AFHTTPRequestOperationManager

#pragma mark - Initialize

- (instancetype)initWithBaseURL:(NSURL *)url
               configurationAPI:(id)configuration;

#pragma mark - Operations
#pragma mark GET

- (AFHTTPRequestOperation *)operationWithGET:(NSString *)method
                                  parameters:(id)parameters
                                 resultClass:(Class)resultClass
                                     success:(OperationManagerCompletionBlockWithSuccess)success
                                     failure:(OperationManagerCompletionBlockWithFailure)failure;

@end
{% endhighlight %}

Файл имплементации: 
{% highlight objectivec %}
#import "VASOperationManager.h"

#import "VASJSONResponseSerializer.h"

@interface VASOperationManager()

@property (nonatomic, strong) NSMutableDictionary *parameters;

@end

@implementation VASOperationManager

#pragma mark - Initialize

- (instancetype)initWithBaseURL:(NSURL *)url
               configurationAPI:(id)configuration
{
    if (self = [super initWithBaseURL:url]) {
        _parameters = [NSMutableDictionary dictionaryWithDictionary:configuration];
    }
    return self;
}

#pragma mark - Operations

- (AFHTTPRequestOperation *)operationWithGET:(NSString *)method
                                  parameters:(id)parameters
                                 resultClass:(Class)resultClass
                                     success:(OperationManagerCompletionBlockWithSuccess)success
                                     failure:(OperationManagerCompletionBlockWithFailure)failure
{
// Создаем реквест с методом и параметрами
    NSURLRequest *urlRequest = [self requestWithGET:method parameters:parameters];

// Затем создаем операцию c нашим реквестом
    AFHTTPRequestOperation *operation = [self HTTPRequestOperationWithRequest:urlRequest
                                                                      success:^(AFHTTPRequestOperation *operation, id responseObject) {
                                                                          if (responseObject) {
                                                                              if (success)
                                                                                  success(operation, responseObject);
                                                                          }
                                                                      } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                                                                          if (error) {
                                                                              if (failure)
                                                                                  failure(operation, error);
                                                                          }
                                                                      }];
// У класса AFRequestOperation имеется свойство с responseSerializer. Устанавливая наш сериализатор, операция в данном случае будет использовать наш сабкласс, и в итоге мы будем получать ответ в нужном нам виде. 
    VASJSONResponseSerializer *responseSerializer = [[VASJSONResponseSerializer alloc] initWithResultClass:resultClass];
    operation.responseSerializer = responseSerializer;
    
    return operation;
}

#pragma mark - Request's

- (NSURLRequest *)requestWithGET:(NSString *)method parameters:(id)parameters
{
    [self.parameters addEntriesFromDictionary:parameters];
    NSString *urlString = [[NSURL URLWithString:method? : [NSString string] relativeToURL:self.baseURL] absoluteString];
    
// То, зачем нужно использовать AFNetworking. Это способность создать правильный реквест с методом, параметрами и возможно еще дополнительными хэдэрами. 
    NSURLRequest *urlRequest = [[AFHTTPRequestSerializer serializer] requestWithMethod:@"GET"
                                                                             URLString:urlString
                                                                            parameters:self.parameters
                                                                                 error:NULL];
    return urlRequest;
}
{% endhighlight %}

На этом моменте мы уже можем делать запросы к серверу, используя данный менеджер.

Перейдем к реализации VASNetworkRequestManager:

Он будет иметь почти идентичный заголовочный файл, как и в случае с VASOperationManager:

{% highlight objectivec %}
#import <Foundation/Foundation.h>

@class AFHTTPRequestOperation;

typedef void(^NetworkRequestCompletionBlockWithSuccess)(id responseObject);
typedef void(^NetworkRequestCompletionBlockWithFailure)(NSError *error);

@interface VASNetworkRequestManager : NSObject

#pragma mark - Initialize

- (instancetype)initWithBaseURL:(NSURL *)baseURL
          baseRequestParameters:(NSDictionary *)parameters;

#pragma mark - Request's

- (AFHTTPRequestOperation *)sendGetRequestWithMethod:(NSString *)method
                                          parameters:(id)parameters
                                         resultClass:(Class)resultClass
                                             success:(NetworkRequestCompletionBlockWithSuccess)success
                                             failure:(NetworkRequestCompletionBlockWithFailure)failure;

@end
{% endhighlight %}

А так же файл имплементации:
{% highlight objectivec %}
#import "VASNetworkRequestManager.h"

#import "AFNetworking.h"
#import "VASOperationManager.h"

@interface VASNetworkRequestManager()

@property (nonatomic, strong) VASOperationManager *operationManager;

@end

@implementation VASNetworkRequestManager

#pragma mark - Initialize

- (instancetype)initWithBaseURL:(NSURL *)baseURL
          baseRequestParameters:(NSDictionary *)parameters
{
    if (self = [super init]) {
        _operationManager = [[VASOperationManager alloc] initWithBaseURL:baseURL
                                                        configurationAPI:parameters];
    }
    return self;
}

#pragma mark - Request's

- (AFHTTPRequestOperation *)sendGetRequestWithMethod:(NSString *)method
                                          parameters:(id)parameters
                                         resultClass:(Class)resultClass
                                             success:(NetworkRequestCompletionBlockWithSuccess)success
                                             failure:(NetworkRequestCompletionBlockWithFailure)failure
{
//  Используем метод нашего кастомного менеджера, в который передаем метод, параметры и класс модели
    AFHTTPRequestOperation *operation = [self.operationManager operationWithGET:method
                                                                     parameters:parameters
                                                                    resultClass:resultClass
                                                                        success:^(AFHTTPRequestOperation *operation, id responseObject) {
                                                                            if (responseObject) {
                                                                                if (success)
                                                                                    success(responseObject);
                                                                            }
                                                                        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                                                                            if (error) {
                                                                                if (failure)
                                                                                    failure(error);
                                                                            }
                                                                        }];
// После запускаем операцию и возвращаем ее
    [operation start];
    
    return operation;
}

@end
{% endhighlight %}

И наконец мы добрались до финального класса, который будем всем этим воротить - VASResourceManager. В случае с API Инстаграмма, я хотел получать полную информацию о пользователе и его последние медиа.

Собственно, два метода: 

{% highlight objectivec %}
#import <Foundation/Foundation.h>

typedef void(^CompletionBlockWithSuccess)(id responseObject);
typedef void(^CompletionBlockWithFailure)(NSError *error);

@interface VASResourceManager : NSObject

- (void)requestUserInfoWithSuccess:(CompletionBlockWithSuccess)success
                           failure:(CompletionBlockWithFailure)failure;
- (void)requestRecentUserMediaListWithSuccess:(CompletionBlockWithSuccess)success
                                      failure:(CompletionBlockWithFailure)failure;

@end
{% endhighlight %}

Перейдем к реализации: 

{% highlight objectivec %}
#import "VASResourceManager.h"

#import "AFNetworking.h"
#import "VASNetworkRequestManager.h"
#import "VASUser.h"
#import "VASMedia.h"

static NSString *const kUserRecentMediaAPIMethod = @"media/recent";

@interface VASResourceManager()

@property (nonatomic, strong) VASNetworkRequestManager *manager;

@end

@implementation VASResourceManager

- (instancetype)init
{
    if (self = [super init])
    {
        _manager = [[VASNetworkRequestManager alloc] initWithBaseURL:[NSURL URLWithString:kInstagramBaseAPIUrl]
                                               baseRequestParameters:@{
                                                                       @"client_id" : kInstagramAPIClientID
                                                                       }];
    }
    return self;
}

- (void)requestUserInfoWithSuccess:(CompletionBlockWithSuccess)success
                           failure:(CompletionBlockWithFailure)failure
{
    [self.manager sendGetRequestWithMethod:nil
                                parameters:nil
                               resultClass:[VASUser class]
                                   success:^(id responseObject) {
                                       if (responseObject) {
                                           if (success)
                                               success(responseObject);
                                       }
                                   }
                                   failure:^(NSError *error) {
                                       
                                   }];
}

- (void)requestRecentUserMediaListWithSuccess:(CompletionBlockWithSuccess)success
                                      failure:(CompletionBlockWithFailure)failure
{
    [self.manager sendGetRequestWithMethod:kUserRecentMediaAPIMethod
                                parameters:nil
                               resultClass:[VASMedia class]
                                   success:^(id responseObject) {
                                       if (responseObject) {
                                           if (success)
                                               success(responseObject);
                                       }
                                   }
                                   failure:^(NSError *error) {
                                       
                                   }];
}

@end
{% endhighlight %}

В данном менеджере мы используем наш VASNetworkRequestManager, и его метод, который возвращает нам запущенную операцию, и в случае чего мы сможем ее остановить и проделать многие другие манипуляции в соответствии с определенной ситуацией.

На этом собственно все, что хотелось показать. С данным классом, как я уже написал, должны уже работать вьюмодели и получать готовые данные в виде моделей.

Пример кода лежит <a href="https://github.com/spbvasilenko/InstagramSample">здесь</a>. Всем спасибо за внимание!
