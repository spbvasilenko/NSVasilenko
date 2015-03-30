---
layout: post
title:  "История о том, как Apple учила меня делать качественный продукт"
date:   2015-03-30 19:31:00
categories: iOS, objective-c, apple, development
---

Всем привет, суть моего рассказа в том, чтобы рассказать откуда пришла идея, как разрабатывалось приложение и как влияло Apple на разработку. 
<habracut />
<b>Я буду показывать код приложения последней версии.</b>

<i>Идея появилась в обыкновенный день, я был в гостях у родителей летом с моим родным братом. Брат – старший лейтенант. В этот день он хотел себе найти в App Store приложение, которое содержало бы полный воинский устав с несложной и простой навигацией, но ни одного приложения в магазине не нашлось по его поисковым запросам. Тут он сразу же мне предложил реализовать данную идею, так как он считал, что это довольно нужная вещь на телефоне, которая будет всегда с тобой и должен быть неплохой спрос. 
Вы скажете, наверное: “Да в чем проблема? Зайди в Google, скачай себе pdf-ник и закинь в iBooks”. Но зачем столько телодвижений? Если теперь есть приложение, когда у тебя все под рукой, скачивать и искать ничего уже не нужно! К тому же с удобной навигацией, которая будет улучшаться до больших высот.</i>

<b>И так, Вы узнали как появилась идея, но теперь самое интересное – как проходила сама разработка и причем тут влияние компании Apple?</b>

Примерно в середине августа я сел за дело. Я думал как лучше всего реализовать приложение: закидывать все в Label или использовать TextView? 
Подумав несколько дней, я нашел для себя решение – использовать обычный Web View. Для того, чтобы устав отображался во Web View, я нарезал по разным частям устав по отдельным html-файлам. 
Код уже содержит функционал возможности изменить размер шрифта. Я использовал NSUserDefaults для сохранения значения размера шрифта и сделал небольшую проверку, где уже подставлял нужный мне html-файл. Шрифт можно установить следующим образом - маленький, средний, большой.

Контроллер с настройкой шрифта: 
<source lang="objectivec">
- (IBAction)selectFont:(id)sender {
    
    [self saveSetting];
}

- (void) saveSetting {
    
    NSUserDefaults* userDefaults = [NSUserDefaults standardUserDefaults];
    
    // начинаем сохранять значения
    
    [userDefaults setInteger:self.fontSegmentedControl.selectedSegmentIndex forKey:kSettingsFont];
    
    [userDefaults synchronize];
    
}

- (void) loadSetting {
    
    NSUserDefaults* userDefaults = [NSUserDefaults standardUserDefaults];
    
    // загружаем значения
    
    self.fontSegmentedControl.selectedSegmentIndex = [userDefaults integerForKey:kSettingsFont];
}
</source>

Открытие самого webView: 
<source lang="objectivec">
- (void) loadPage {
    
    NSUserDefaults* userDefaults = [NSUserDefaults standardUserDefaults];
    
    NSInteger indexFont = [userDefaults integerForKey:@"font"];
    
    if (indexFont == 0) {
        // узнаем путь к файлу
        NSString* filePath = [[NSBundle mainBundle] pathForResource:@"preOnePart" ofType:@"html"];
        // помещаем в NSString
        NSString* html = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:nil];
        // загружаем в UIWebView содержимое
        [_webView loadHTMLString:html baseURL:nil];
    } else if (indexFont == 1) {
        // узнаем путь к файлу
        NSString* filePath = [[NSBundle mainBundle] pathForResource:@"preOnePartMediumFont" ofType:@"html"];
        // помещаем в NSString
        NSString* html = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:nil];
        // загружаем в UIWebView содержимое
        [_webView loadHTMLString:html baseURL:nil];
        
    } else if (indexFont == 2) {
        // узнаем путь к файлу
        NSString* filePath = [[NSBundle mainBundle] pathForResource:@"preOnePartLargeFont" ofType:@"html"];
        // помещаем в NSString
        NSString* html = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:nil];
        // загружаем в UIWebView содержимое
        [_webView loadHTMLString:html baseURL:nil];
        
    }
</source>

Несколько дней я создавал каркас приложения, меню и прочие мелочи. Сделал нужную мне несложную навигацию по частям и главам устава и в принципе, приложение было готово. Использовал open-source библиотеку <a href="https://github.com/John-Lluch/SWRevealViewController">SWRevealController</a> для меню. 

Добавил в меню раздел "О приложении". 
Здесь пользователь может рассказать в Twitter и Facebook о приложении:

<source lang="objectivec">
#pragma mark - share actions

- (IBAction)twitterShare:(UIButton *)sender {
    
    if ([SLComposeViewController
         isAvailableForServiceType:SLServiceTypeTwitter]) {
        SLComposeViewController* controller = [SLComposeViewController composeViewControllerForServiceType:SLServiceTypeTwitter];
        
        [controller setInitialText:@" #AppStore Я пользуюсь приложением «Military! - Новости & Устав Внутренней Службы Вооруженных Сил РФ» в App Store. Советую и вам ;)"];
        [controller addURL:[NSURL URLWithString:@"https://itunes.apple.com/ru/app/military!-novosti-ustav-vnutrennej/id914651209?l=en&mt=8"]];
        
        
        controller.completionHandler = ^(SLComposeViewControllerResult result) {
            NSLog(@"Completed");
        };
        
        [self presentViewController:controller animated:YES completion:nil];
    } else {
        
        UIAlertView* error = [[UIAlertView alloc]
                              initWithTitle:@"Ошибка"
                              message:@"Вы не настроили Twitter-аккаунт в настройках. \nПройдите авторизацию, пожалуйста!"
                              delegate:self
                              cancelButtonTitle:@"OK"
                              otherButtonTitles:nil,nil];
        
        [error show];
        
        NSLog(@"The twitter sevice is not evailalble");


    }
}

- (IBAction)facebookShare:(UIButton *)sender {
    
    // Put together the dialog parameters
    NSMutableDictionary *params = [NSMutableDictionary dictionaryWithObjectsAndKeys:
                                   @"Military!", @"name",
                                   @"Новости & Устав Внутренней Службы Вооруженных Сил России", @"caption",
                                   @"Если вы военнослужащий Российской Федерации и всегда хотите быть в курсе новостей Вооруженных Сил РФ, а так же всегда иметь при себе устав Внутренней Службы, то это приложение для Вас!", @"description",
                                   @"https://itunes.apple.com/ru/app/military!-novosti-ustav-vnutrennej/id914651209?l=en&mt=8", @"link",
                                   @"http://cs619627.vk.me/v619627853/19d6e/xdHr-XTVs8k.jpg", @"picture",
                                   nil];
    
    // Show the feed dialog
    [FBWebDialogs presentFeedDialogModallyWithSession:nil
                                           parameters:params
                                              handler:^(FBWebDialogResult result, NSURL *resultURL, NSError *error) {
                                                  if (error) {
                                                      // An error occurred, we need to handle the error
                                                      // See: https://developers.facebook.com/docs/ios/errors
                                                      NSLog(@"Error publishing story: %@", error.description);
                                                  } else {
                                                      if (result == FBWebDialogResultDialogNotCompleted) {
                                                          // User cancelled.
                                                          NSLog(@"User cancelled.");
                                                      } else {
                                                          // Handle the publish feed callback
                                                          NSDictionary *urlParams = [self parseURLParams:[resultURL query]];
                                                          
                                                          if (![urlParams valueForKey:@"post_id"]) {
                                                              // User cancelled.
                                                              NSLog(@"User cancelled.");
                                                              
                                                          } else {
                                                              // User clicked the Share button
                                                              NSString *result = [NSString stringWithFormat: @"Posted story, id: %@", [urlParams valueForKey:@"post_id"]];
                                                              NSLog(@"result %@", result);
                                                          }
                                                      }
                                                  }
                                              }];
}


//------------------Login implementation starts here------------------

- (void)loginView:(FBLoginView *)loginView handleError:(NSError *)error {
    NSString *alertMessage, *alertTitle;
    
    // If the user should perform an action outside of you app to recover,
    // the SDK will provide a message for the user, you just need to surface it.
    // This conveniently handles cases like Facebook password change or unverified Facebook accounts.
    if ([FBErrorUtility shouldNotifyUserForError:error]) {
        alertTitle = @"Facebook error";
        alertMessage = [FBErrorUtility userMessageForError:error];
        
        // This code will handle session closures since that happen outside of the app.
        // You can take a look at our error handling guide to know more about it
        // https://developers.facebook.com/docs/ios/errors
    } else if ([FBErrorUtility errorCategoryForError:error] == FBErrorCategoryAuthenticationReopenSession) {
        alertTitle = @"Session Error";
        alertMessage = @"Your current session is no longer valid. Please log in again.";
        
        // If the user has cancelled a login, we will do nothing.
        // You can also choose to show the user a message if cancelling login will result in
        // the user not being able to complete a task they had initiated in your app
        // (like accessing FB-stored information or posting to Facebook)
    } else if ([FBErrorUtility errorCategoryForError:error] == FBErrorCategoryUserCancelled) {
        NSLog(@"user cancelled login");
        
        // For simplicity, this sample handles other errors with a generic message
        // You can checkout our error handling guide for more detailed information
        // https://developers.facebook.com/docs/ios/errors
    } else {
        alertTitle  = @"Something went wrong";
        alertMessage = @"Please try again later.";
        NSLog(@"Unexpected error:%@", error);
    }
    
    if (alertMessage) {
        [[[UIAlertView alloc] initWithTitle:alertTitle
                                    message:alertMessage
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    }
}


- (NSDictionary*)parseURLParams:(NSString *)query {
    NSArray *pairs = [query componentsSeparatedByString:@"&"];
    NSMutableDictionary *params = [[NSMutableDictionary alloc] init];
    for (NSString *pair in pairs) {
        NSArray *kv = [pair componentsSeparatedByString:@"="];
        NSString *val =
        [kv[1] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        params[kv[0]] = val;
    }
    return params;
}

</source>

Еще два дня я наводил маленькую красоту в приложении. Спешу отметить, что в первоначальной версии, которая была отправлено на Review в Apple, приложение не имело раздела “Свежие новости” - был просто один устав и все. 

Приложение было отправлено на Review в начале сентября и уже через неделю я получил первый Reject от Apple. В чем же дело? Apple трактовала свою точку зрения таким образом - “Приложения, представляющие из себя песню или фильм, должны быть отправлены в iTunes Store. Приложения, представляющие из себя книгу, должны быть отправлены в iBooks Store”. 

Да, в принципе, Apple были правы, так как приложение было больше похоже на книгу. Я решил добавить в функционал что-нибудь еще... Например, поток свежих новостей той же тематики, что и приложение. Взял для этого RSS-поток "РИА Новости" и парсил его c помощью NSXMLParser:
<source lang="objectivec">
#pragma mark - NSXMLParser

- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict {
    
    element = elementName;
    
    if ([element isEqualToString:@"item"]) {
        
        item    = [[NSMutableDictionary alloc] init];
        title   = [[NSMutableString alloc] init];
        link    = [[NSMutableString alloc] init];
        image = [[NSMutableDictionary alloc] init];
        description = [[NSMutableString alloc] init];
        date = [[NSMutableString alloc] init];
    }
    
    if ([element isEqualToString:@"enclosure"]) {
        linkImage   =   [attributeDict objectForKey:@"url"];
        typeImage    =   [attributeDict objectForKey:@"type"];
        
    }
    
   // NSLog(@"RSS Utility: didStartElement: %@", elementName);
}

- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName {
    
    if ([elementName isEqualToString:@"item"]) {
        
        [item setObject:title forKey:@"title"];
        [item setObject:link forKey:@"link"];
        [item setObject:description forKey:@"description"];
        [item setObject:image forKey:@"enclosure"];
        [item setObject:typeImage forKey:@"imageType"];
        [item setObject:linkImage forKey:@"imageUrl"];
        [item setObject:date forKey:@"pubDate"];
        
        [feeds addObject:[item copy]];
        
    }
    
}

- (void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string {
    
    if ([element isEqualToString:@"title"]) {
        [title appendString:string];
    } else if ([element isEqualToString:@"link"]) {
        [link appendString:string];
    } else if ([element isEqualToString:@"imageUrl"]) {
        [linkImage appendString:string];
    } else if ([element isEqualToString:@"description"]) {
        [description appendString:string];
    } else if ([element isEqualToString:@"pubDate"]) {
        [date appendString:string];
    }
    
}

- (void)parserDidEndDocument:(NSXMLParser *)parser {
    
    [self.tableView reloadData];
    [self.refreshControl endRefreshing];
    
}
</source> 

Для того, чтобы посмотреть новость полностью, по нажатию на ячейку таблицы, открывался контроллер с Web View

Метод, который передает ссылку на новость в другой контроллер: 
<source lang="objectivec">
#pragma mark - Passed data to another controller

- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    if ([[segue identifier] isEqualToString:@"CustomSegue"]) {
        
        NSIndexPath *indexPath = [self.tableView indexPathForSelectedRow];
        NSString *string = [feeds[indexPath.row] objectForKey: @"link"];
        [[segue destinationViewController] setUrl:string];
        NSString* string2 = [feeds [indexPath.row] objectForKey:@"title"];
        [[segue destinationViewController] setTitleForShare:string2];
        
    }
}
</source>

Наши действия в контроллере с Web View: 
<source lang="objectivec">
@implementation IVDetailViewController

@synthesize titleForShare;

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSURL *myURL = [NSURL URLWithString: [self.url stringByAddingPercentEscapesUsingEncoding:
                                          NSUTF8StringEncoding]];
    NSURLRequest *request = [NSURLRequest requestWithURL:myURL];
    
    [self.webView loadRequest:request];
    
    [self.navigationController.navigationBar addGestureRecognizer:self.revealViewController.panGestureRecognizer];
    // Do any additional setup after loading the view.
}
</source>


Реализовав данный функционал, я вновь отправил на проверку в Apple. Через полторы неделю получаю второй отказ. Что же не так в этот раз?! Читаю комментарии проверяющих приложение - “Apple и наши клиенты высоко ценят простой, изысканный, творческий, хорошо продуманный интерфейс. Они требуют больше усилий, но оно того стоит. Apple устанавливает высокую планку. Если пользовательский интерфейс недостаточно хорош, приложение может быть отклонено”. 

Подумав своей головой, я кинулся снова дорабатывать свое детище. Были добавлены фотографии в ленту новостей и шаринг, здесь я использовал Custom Table Cells.

После этого добавил еще навигацию по Web View в контролере, где можно посмотреть новость полностью. То бишь - на страницу вперед, на страницу назад, остановить загрузку, обновить. И еще открывающийся UIActivityViewController:

<source lang="objectivec">
- (IBAction)activityAction:(id)sender {
    
    
        NSString *textToShare = titleForShare;
        NSURL *myWebsite = [NSURL URLWithString:self.url];
    
        NSArray *objectsToShare = @[textToShare, myWebsite];
        
        UIActivityViewController *activityVC = [[UIActivityViewController alloc] initWithActivityItems:objectsToShare applicationActivities:nil];
        
        NSArray *excludeActivities = @[UIActivityTypeAirDrop,
                                       UIActivityTypePrint,
                                       UIActivityTypeAssignToContact,
                                       UIActivityTypeSaveToCameraRoll,
                                       UIActivityTypeAddToReadingList,
                                       UIActivityTypePostToFlickr,
                                       UIActivityTypePostToVimeo];
    
        activityVC.excludedActivityTypes = excludeActivities;
        
        [self presentViewController:activityVC animated:YES completion:nil];
}
</source>

Доработал интерфейс, обновил так же иконку приложения и добавил splash view (это то, что появляется перед запуском приложения).
Приложение было отправлено в третий раз. И в этот раз приложение наконец-то загорело счастливым мне статусом “Ready For Sale”. 

<b>Почему я считаю, что Apple сильно повлияло на вид и функционал приложения? Думаю из рассказа все стало понятно. И для себя я понял, почему Apple так трепетно относится к проверке приложений, отправленых им на проверку. Не нужно никуда спешить, нужно делать качественный продукт сразу! Apple любит качество и в этом мы еще раз убедились! Выводы для себя, естественно, сделаны.</b>

P.S. Спешу заметить для Вас, дорогие читатели: мой опыт разработки под iOS всего-то ничего – буквально с мая 2014 года.

P.S.P.S Почти сразу же во второй день после дня релиза приложения я получил первый плохой отзыв приложению. Основые претензии были такими: старый устав за 2007 год, без обновлений за 2014 год и в приложении не было возможности изменить размер шрифта для удобства чтения устава. Вина полностью лежала на мне и я это понимал. В этот же день, ценою своего сна, я все это реализовал и поправил прочие мелочи. Обновление уже отправлено на проверку.
После этого была еще создана специальная support-группа для данного приложения, где пользователь сможет лично убедиться, что я их услышал. Группу можно найти на странице приложения в App Store в разделе “Сайт разработчика”.

UPDATE!

Сейчас приложение находится на первом месте в топ платных приложений категории "Новости". 
<img src="http://habrastorage.org/getpro/habr/post_images/62e/241/697/62e24169719beba930240d19afb3e731.jpg" alt="image"/>
