---
layout: post
title: Использование параметризованных юнит тестов в iOS разработке
excerpt: "После прочтения одной из статей в блоге Сергея Теплякова о параметризованных юнит тестах - я был восхищен данным инструментом и решил попробовать в среде Objective-C."
categories: [testing]
comments: true
---

Я очень часто листаю [блог Сергея Теплякова](http://sergeyteplyakov.blogspot.ru/), и считаю его одним из самых невероятных русскоязычных блогов, который я когда-либо читал в своей карьере программиста. 

Сегодняшним вечером, после рабочего дня, я как обычно просматривал блог Сергея в поисках чего-то нового для себя и наткулся на потрясающую статью [Параметризованные юнит тесты](http://sergeyteplyakov.blogspot.ru/2012/08/blog-post_28.html), в которой Сергей рассказал о том, что такой инструмент написания тестов незаслуженно многие обходят стороной.

После прочтения статьи, я был тоже удивлен, почему же я только сегодня узнал о таком потрясном и удобном инструменте. В ту же минуту я понял, что я бы непременно хотел использовать то же самое в iOS разработке. Но к сожалению Apple меня расстроили и Xcode не поддерживает данный вид тестов (после получаса гугления). Тем не менее, поиск не закончился неудачей! В процессе поиска я наткнулся на библиотеку [KNMParametrizedTest](https://github.com/konoma/xctest-parametrized-tests), которая вполне себе выполняет поставленную задачу. 

Если коротко: 

* Библиотека парсит написанный перед тестом макрос, в котором мы перечисляем название теста и параметры.
* Затем парсит тесты с определенным названием и одним параметром в текущем XCTTestCase
* Из макроса перебирает все параметры, которые мы перечислили и выполняет Invocation теста ( который попадает под правила "параметризованный") с переданным параметром.

В результате, у нас выполняется тест столько раз, сколько мы передали параметров в макросе.

Я сразу же решил попробовать интегрировать библиотеку в свой текущий проект и опробовать на своих же тестах и "О Боги!" - после рефакторинга некоторых тестов на теперь уже параметризированные, я был восхищен получившимся результатом.

Что же, слов мало, давайте посмотрим на вариант тестов до рефакторинга. Я приведу вам пример тестов текст форматтера, и тесты, проверяющие поведение, когда мы вводим символы [1, 2, 7] и если это будет первый символ в текущем тексте TextField'а, то форматтер должен отсеивать такой символ и возвращать текущий текст из TextField'a. 

```objective-c
- (void)testThatPhoneTextFormatterShouldReturnCurrentTextIfPhoneCodeIsStartFrom1 {
    // given
    NSString *currentText = @"";
    NSString *newString = @"1";

    // when
    NSString *formattedText = [self.phoneTextFormatter formatedStringFromString:newString currentText:currentText range:NSMakeRange(0, 0)];

    // then
    XCTAssertEqualObjects(formattedText, currentText);
}

- (void)testThatPhoneTextFormatterShouldReturnCurrentTextIfPhoneCodeIsStartFrom2 {
    // given
    NSString *currentText = @"";
    NSString *newString = @"2";

    // when
    NSString *formattedText = [self.phoneTextFormatter formatedStringFromString:newString currentText:currentText range:NSMakeRange(0, 0)];

    // then
    XCTAssertEqualObjects(formattedText, currentText);
}

- (void)testThatPhoneTextFormatterShouldReturnCurrentTextIfPhoneCodeIsStartFrom7 {
    // given
    NSString *currentText = @"";
    NSString *newString = @"7";

    // when
    NSString *formattedText = [self.phoneTextFormatter formatedStringFromString:newString currentText:currentText range:NSMakeRange(0, 0)];

    // then
    XCTAssertEqualObjects(formattedText, currentText);
}
```

Согласитесь, не самое разборчивое название тестов? Да и дублирование кода наблюдается.

Теперь давайте посморим как будет выглядеть такие тесты, а точнее уже тест после того как мы здесь используем параметризованный тест:

```objective-c
KNMParametersFor(
        test_whenFormattedStringFromString_thatTextFormatterShouldReturnCurrentText_withCurrentEmptyText_andWithTestValue,
        @[@"1", @"2", @"7"]
);
- (void)test_whenFormattedStringFromString_thatTextFormatterShouldReturnCurrentText_withCurrentEmptyText_andWithTestValue:(NSString *)testValue {
    // given
    NSString *currentText = @"";

    // when
    NSString *formattedText = [self.phoneTextFormatter formatedStringFromString:testValue currentText:currentText range:NSMakeRange(0, 0)];

    // then
    XCTAssertEqualObjects(formattedText, currentText, @"Test failed with test value: %@", testValue);
}
```

Теперь намного ведь круче и читаеме, верно? Такой же подход я советую теперь и вам, друзья. Это очень облегчит нам жизнь во мноих случаях.

Сергею большое спасибо, за то, что он раскрыл мне глаза на такую чудную вещь в своей статье.
