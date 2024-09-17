---
title: Валидация в Java-приложениях
updated: 2021-02-28 10:01:56Z
created: 2021-02-28 10:01:50Z
---

# Валидация в Java-приложениях
![](&&&SFLOCALFILEPATH&&&98c3593bc98a7cd97a2df72abb33cfd5.jpeg)
*Этот текст посвящен различным подходам к валидации данных: на какие подводные камни может наткнуться проект и какими методами и технологиями стоит руководствоваться при валидации данных в Java-приложениях.*

![](&&&SFLOCALFILEPATH&&&cjycyj3zf_l1ai9ch_vsgzisstk.png)

Я часто видел проекты, создатели которых вообще не утруждались выбором подхода к валидации данных. Команды работали над проектом под невероятным давлением в виде сроков и размытых требований, и в итоге у них просто не оставалось времени на точную, последовательную валидацию. Поэтому, код валидации у них разбросан повсюду: в сниппетах Javascript, контроллерах экранов, в бинах бизнес-логики, сущностях предметной области, триггерах и database constraints. В этом коде было полно операторов if-else, он выбрасывал кучу исключений, и попробуй разберись, где там у них валидируется этот конкретный кусок данных… Как результат, по мере развития проекта становится тяжело и дорого соблюдать и требования (зачастую довольно путаные), и единообразие подходов к валидации данных.

Так есть ли какой-то простой и изящный способ валидации данных? Способ, который защитит нас от греха нечитаемости, способ, который соберет всю логику валидации воедино, и который уже создан за нас разработчиками популярных Java-фреймворков?

Да, такой способ существует.

 
Для нас, разработчиков [платформы CUBA](https://www.cuba-platform.ru/), очень важно, чтобы вы могли пользоваться передовыми практиками. Мы считаем, что код валидации должен:

1. Быть переиспользуемым и следовать принципу DRY;
2. Быть естественным и понятным;
3. Помещаться там, где его ожидает увидеть разработчик;
4. Уметь проверять данные из разных источников: пользовательского интерфейса, вызовов SOAP, REST и т.д.
5. Без проблем работать в многопоточной среде;
6. Вызываться внутри приложения автоматически, без необходимости запускать проверки вручную;
7. Выдавать пользователю понятные, локализованные сообщения в лаконичных диалоговых окнах;
8. Следовать стандартам.

Давайте посмотрим, как это можно реализовать на примере приложения, написанного с использованием фреймворка CUBA Platform. Однако, так как CUBA построена на основе Spring и EclipseLink, большинство используемых здесь приемов будет работать и на любой другой Java платформе, поддерживающей спецификации JPA и Bean Validation.

## Валидация с использованием database constraints

Пожалуй, самый распространенный и очевидный способ валидации данных — это использование ограничений на уровне БД, например, флаг required (для полей, значение которых не может быть пустым), длина строки, уникальные индексы и т.д. Такой способ больше всего подходит для корпоративных приложений, так как этот тип ПО обычно строго ориентирован на обработку данных. Тем не менее, даже здесь разработчики часто совершают ошибки, задавая ограничения отдельно для каждого уровня приложения. Чаще всего причина кроется в распределении обязанностей между разработчиками.

Рассмотрим пример, который большинству из нас знаком, некоторым даже по собственному опыту… Если спецификация гласит, что в поле номера паспорта должно быть 10 знаков, весьма вероятно, что проверяться это будет всеми: архитектором БД в DDL, backend-разработчиком в соответствующих Entity и REST сервисах, и наконец, разработчиком UI непосредственно на стороне клиента. Затем это требование меняется, и поле возрастает до 15 знаков. Девопсы меняют значения constraints в БД, но для пользователя ничего не меняется, ведь на стороне клиента ограничение все то же...

Любой разработчик знает, как избежать этой проблемы, — валидация должна быть централизована! В CUBA такая валидация находится в JPA-аннотациях к сущностям. Основываясь на этой метаинформации, CUBA Studio сгенерирует верный DDL-скрипт и применит соответствующие валидаторы на стороне клиента.

![](&&&SFLOCALFILEPATH&&&118a16f1705740a1676eb2c0ea164ef4.png)

Если аннотации изменятся, CUBA обновит DDL-скрипты и сгенерирует миграционные скрипты, поэтому в следующий раз при развертывании проекта новые ограничения на основе JPA вступят в силу и в интерфейсе и в базе данных приложения.

Несмотря на простоту и реализацию на уровне БД, дающую абсолютную надежность этому методу, область применения JPA аннотаций ограничена самыми простыми случаями, которые могут быть выражены в стандарте DDL, и не включают триггеры БД или хранимые процедуры. Так что, ограничения, основанные на JPA могут делать поле сущности уникальным или обязательным или задавать максимальную длину столбца. Можно даже задать уникальное ограничение для комбинации колонок с помощью аннотации `@UniqueConstraint`. Но на этом, пожалуй, все.

Как бы то ни было, в случаях, требующих более сложной логики валидации, вроде проверки поля на минимальное/максимальное значение, валидации при с помощи регулярного выражения, или выполнения кастомной проверки, свойственной только вашему приложению, применяется подход, известный как **"Bean Validation"**.

## Bean validation

Всем известно, что хорошей практикой является следование стандартам, имеющим длинный жизненный цикл, чья эффективность доказана на тысячах проектов. Java Bean Validation — это подход, зафиксированный в [JSR 380, 349 и 303](https://beanvalidation.org/specification/) и их применениях: [Hibernate Validator](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/?v=5.3) и [Apache BVal](http://bval.apache.org/).

Хотя этот подход знаком многим разработчикам, его часто недооценивают. Это простой способ встраивать валидацию данных даже в legacy-проекты, который позволяет выстраивать проверки понятно, просто, надежно и максимально близко к бизнес-логике.

Использование Bean Validation дает проекту немало преимуществ:

* Логика валидации расположена рядом с предметной областью: определение ограничений для полей и методов бина происходит естественным и по-настоящему объектно-ориентированным образом.
* Стандарт Bean Validation дает нам десятки [валидационных аннотаций прямо из коробки](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-defineconstraints-spec), например: `@NotNull`, `@Size`, `@Min`, `@Max`, `@Pattern`, `@Email`, `@Past`, не совсем стандартные `@URL`, `@Length`, мощнейший `@ScriptAssert` и многие другие.
* Стандарт не ограничивает нас готовыми аннотациями и позволяет создавать свои собственные. Мы можем также создать новую аннотацию, объединив несколько других, или определить ее, используя отдельный Java-класс как валидатор.
* Например, в приведенном выше примере мы можем задать аннотацию уровня класса `@ValidPassportNumber`, чтобы проверить, что номер паспорта соответствует формату, зависящему от значения поля `country`.
* Ограничения можно ставить не только на поля или классы, но и на методы и их параметры. Этот подход называется **“validation by contract”** и будет рассмотрен чуть позже.

Когда пользователь отправляет введенную информацию, [CUBA Platform](https://www.cuba-platform.ru/) *(как и некоторые другие фреймворки)* запускает Bean Validation автоматически, поэтому он мгновенно выдает сообщение об ошибке, если валидация не прошла, и нам не нужно запускать валидаторы бинов вручную.

Вернемся к примеру с номером паспорта, но на этот раз дополним его несколькими ограничениями сущности Person:

* Поле `name` должно состоять из 2 или более символов и быть корректным. (Как видно, regexp не простой, зато "Charles Ogier de Batz de Castelmore Comte d'Artagnan" пройдет проверку, а "R2D2" — нет);
* `height` (рост) должен быть в следующем интервале: `0 < height <= 300` см;
* Поле `email` должно содержать строку, соответствующую формату корректного email.

Со всеми этими проверками класс Person будет выглядеть так:

```
@Listeners("passportnumber_PersonEntityListener")
@NamePattern("%s|name")
@Table(name = "PASSPORTNUMBER_PERSON")
@Entity(name = "passportnumber$Person")
@ValidPassportNumber(groups = {Default.class, UiCrossFieldChecks.class})
@FraudDetectionFlag
public class Person extends StandardEntity {
    private static final long serialVersionUID = -9150857881422152651L;

    @Pattern(message = "Bad formed person name: ${validatedValue}",
             regexp = "^[A-Z][a-z]*(\\s(([a-z]{1,3})|(([a-z]+\\')?[A-Z][a-z]*)))*$")
    @Length(min = 2)
    @NotNull
    @Column(name = "NAME", nullable = false)
    protected String name;

    @Email(message = "Email address has invalid format: ${validatedValue}",
           regexp = "^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$")
    @Column(name = "EMAIL", length = 120)
    protected String email;

    @DecimalMax(message = "Person height can not exceed 300 centimeters", 
                value = "300")
    @DecimalMin(message = "Person height should be positive", 
                value = "0", inclusive = false)
    @Column(name = "HEIGHT")
    protected BigDecimal height;

    @NotNull
    @Column(name = "COUNTRY", nullable = false)
    protected Integer country;

    @NotNull
    @Column(name = "PASSPORT_NUMBER", nullable = false, length = 15)
    protected String passportNumber;

    ...
}
```

[Person.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/global/src/com/company/passportnumber/entity/Person.java)

Полагаю, использование таких аннотаций, как `@NotNull`, `@DecimalMin`,`@Length`, `@Pattern` и им подобных вполне очевидно и не требует комментариев. Давайте пристальнее посмотрим на реализацию аннотации `@ValidPassportNumber`.

Наш новенький `@ValidPassportNumber` проверяет, что `Person#passportNumber` соответствует шаблону regexp для каждой страны, задаваемой полем `Person#country`.

Для начала заглянем в документацию (мануалы по [CUBA](https://doc.cuba-platform.com/manual-latest-ru/bean_validation_constraints.html#bean_validation_custom_constraints) или [Hibernate](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-class-level-constraints) — вполне подойдут), согласно ней, нам необходимо пометить наш класс этой новой аннотацией и передать в нее параметр `groups`, где `UiCrossFieldChecks.class` означает, что данная валидация должна быть запущена на этапе кросс-валидации — после проверки всех индивидуальных полей, а `Default.class` сохраняет ограничение в группе валидации по умолчанию.

Описание аннотации выглядит так:

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = ValidPassportNumberValidator.class)
public @interface ValidPassportNumber {
    String message() default "Passport number is not valid";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

[ValidPassportNumber.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/global/src/com/company/passportnumber/entity/ValidPassportNumber.java)

Здесь `@Target(ElementType.TYPE)` говорит, что целью этой runtime-аннотации является класс, а `@Constraint(validatedBy = … )` определяет что проверку выполняет класс `ValidPassportNumberValidator`, реализующий интерфейс `ConstraintValidator<...>`. Сам код валидации находится в методе `isValid(...)`, который и выполняет реальную проверку довольно прямолинейным образом:

```
public class ValidPassportNumberValidator 
                implements ConstraintValidator<ValidPassportNumber, Person> {
    public void initialize(ValidPassportNumber constraint) {
    }

    public boolean isValid(Person person, ConstraintValidatorContext context) {
        if (person == null)
            return false;

        if (person.country == null || person.passportNumber == null)
            return false;

        return doPassportNumberFormatCheck(person.getCountry(), 
                                           person.getPassportNumber());
    }

    private boolean doPassportNumberFormatCheck(CountryCode country, 
                                                String passportNumber) {
        ...
    }
}
```

[ValidPassportNumberValidator.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/global/src/com/company/passportnumber/entity/ValidPassportNumberValidator.java)

Вот и все. С CUBA Platform нам не нужно писать ничего, кроме строки кода, которая заставит нашу кастомную валидацию работать и выдавать пользователю сообщения об ошибке.
Ничего сложного, правда?

Теперь проверим, как это все работает. Здесь у CUBA есть и другие ништяки: она не только показывает пользователю сообщение об ошибке, но и подсвечивает красным поля, не прошедшие bean validation:

![](&&&SFLOCALFILEPATH&&&31083db2b14d67d88de169542b59748a.png)

Не правда ли, изящное решение? Вы получаете адекватное отображение ошибок валидации в UI, добавив лишь пару Java-аннотаций к сущностям предметной области.

Подводя итог раздела, давайте еще раз кратко перечислим плюсы Bean Validation для сущностей:

1. Она понятна и читаема;
2. Позволяет определять ограничения значений прямо в классах сущностей;
3. Ее можно настраивать и дополнять;
4. Интегрирована в популярные ORM, а проверки запускаются автоматически до того, как изменения сохранятся в БД;
5. Некоторые фреймворки также запускают валидацию бинов автоматически, когда пользователь отправляет данные в UI (а если нет, нетрудно вызвать интерфейс `Validator` вручную);
6. Bean Validation — это признанный стандарт, и в Интернете полно документации по нему.

Но что делать, если нужно установить ограничение на метод, конструктор или REST-адрес для валидации данных, исходящих из внешней системы? Или если нужно декларативно проверить значения параметров метода, не прописывая нудный код с множеством if-else условий в каждом проверяемом методе?

Ответ прост: Bean Validation применим и к методам!

## Validation by contract

Иногда бывает нужно пойти дальше валидации состояния модели данных. Многим методам может пойти на пользу автоматическая валидация параметров и возвращаемого значения. Это может быть необходимо не только для проверки данных, идущих в адреса REST или SOAP, но и в тех случаях, когда мы хотим прописать предусловия и постусловия вызовов метода, чтобы убедиться, что введенные данные были проверены до выполнения тела метода, или что возвращаемое значение находится в ожидаемом диапазоне, или нам, например, нужно просто декларативно описать диапазоны значений входных параметров для улучшения читаемости кода.

С помощью bean validation ограничения могут применяться к входным параметрам и возвращаемым значениям методов и конструкторов для проверки предусловий и постусловий их вызовов у любого Java класса. У этого пути есть несколько преимуществ перед традиционными способами проверки правильности параметров и возвращаемых значений:

1. Не нужно проводить проверки вручную в императивном стиле (например, путем выкидывая `IllegalArgumentException` и ему подобных). Можно определить ограничения декларативно, и сделать код был более понятным и выразительным;
2. Ограничения можно настраивать, использовать повторно и конфигурировать: не нужно писать логику валидации для каждой проверки. Меньше кода — меньше багов.
3. Если класс, возвращаемое значение метода или его параметр отмечены аннотацией `@Validated`, то проверки будут автоматически выполняться платформой при каждом вызове метода.
4. Если выполняемый модуль отмечен аннотацией `@Documented`, его пред- и постусловия будут включены в генерируемый JavaDoc.

Используя **'валидацию по контракту’** мы получаем понятный, компактный и легко поддерживаемый код.

Для примера давайте глянем на интерфейс REST-контроллера CUBA-приложения. Интерфейс `PersonApiService` позволяет получить список людей из БД с помощью метода `getPersons()` и добавить нового человека, используя вызов `addNewPerson(...)`. 

И не забываем, что bean validation наследуется! Другими словами, если мы аннотируем некий класс, или поле, или метод, то на все классы, наследующие данный класс или реализующие данный интерфейс, будет распространяться та же самая валидационная аннотация.

```
@Validated
public interface PersonApiService {
    String NAME = "passportnumber_PersonApiService";

    @NotNull
    @Valid
    @RequiredView("_local")
    List<Person> getPersons();

    void addNewPerson(
              @NotNull
              @Length(min = 2, max = 255)
              @Pattern(message = "Bad formed person name: ${validatedValue}",
                       regexp = "^[A-Z][a-z]*(\\s(([a-z]{1,3})|(([a-z]+\\')?[A-Z][a-z]*)))*$")
              String name,

              @DecimalMax(message = "Person height can not exceed 300 cm", 
                                    value = "300")
              @DecimalMin(message = "Person height should be positive", 
                          value = "0", inclusive = false)
              BigDecimal height,

              @NotNull
              CountryCode country,

              @NotNull
              String passportNumber
            );
}
```

[PersonApiService.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/global/src/com/company/passportnumber/service/PersonApiService.java)

Достаточно ли понятен этот фрагмент кода?
_(За исключением аннотации `@RequiredView(“_local”)`, специфичной для CUBA Platform и проверяющей, что возвращаемый объект `Person` содержит все поля из таблицы `PASSPORTNUMBER_PERSON` )._

Аннотация `@Valid` определяет, что каждый объект коллекции, возвращаемой методом `getPersons()`, должен валидироваться также на соответствие ограничениям класса `Person`.

В CUBA приложении эти методы доступны по следующим адресам:

* /app/rest/v2/services/passportnumber_PersonApiService/getPersons
* /app/rest/v2/services/passportnumber_PersonApiService/addNewPerson

Откроем приложение Postman и убедимся, что валидация работает как надо:

![](&&&SFLOCALFILEPATH&&&qgraqjsxfyeqqa6qcjf2pkf7tcq.png)

Как вы, возможно, заметили, в примере выше не валидируется номер паспорта. Все потому, что это поле требует перекрестной проверки параметров метода `addNewPerson`, так как выбор шаблона регулярного выражения для валидации `passportNumber` зависит от значения поля `country`. Такая перекрестная проверка — полный аналог ограничениям сущностей на уровне класса!

Перекрестная валидация параметров поддерживается JSR 349 и 380. Можете ознакомиться с [документацией hibernate](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-cross-parameter-constraints), чтобы узнать, как реализовать собственную перекрестную валидацию методов класса/интерфейса.

## За пределами bean validation

Нет в мире совершенства, вот и у bean validation есть свои недостатки и ограничения:

1. Иногда нам нужно просто проверить состояние сложного графа объектов перед сохранением изменений в БД. Например, нужно убедиться, что все элементы заказа покупателя помещаются в одну посылку. Это достаточно тяжёлая операция, и осуществлять ее каждый раз, когда пользователь добавляет новые предметы в заказ, не лучшая идея. Поэтому такая проверка может понадобиться только один раз: перед сохранением объекта `Order` и его подобъектов `OrderItem` в БД.
2. Некоторые проверки необходимо проводить внутри транзакции. Например, система электронного магазина должна проверять, достаточно ли экземпляров товара в наличии для исполнения заказа до его коммита в БД. Такая проверка может быть произведена только внутри транзакции, т.к. система является многопоточной и количество товара на складе может измениться в любое время.

CUBA Platform предлагает два механизма валидации данных до коммита, которые называются [entity listeners](https://doc.cuba-platform.com/manual-latest-ru/entity_listeners.html) и [transaction listeners](https://doc.cuba-platform.com/manual-latest-ru/transaction_listeners.html). Рассмотрим их подробнее.

### Entity listemers

[Entity listeners в CUBA](https://doc.cuba-platform.com/manual-latest-ru/entity_listeners.html) очень похожи на [PreInsertEvent, PreUpdateEvent и PredDeleteEvent listeners](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-checkconstraints-orm-hibernateevent), которые JPA предлагает разработчику. Оба механизма позволяют проверять объекты сущностей до и после того, как они будут сохранены в БД. 

В CUBA легко создать и подключить entity listener, для этого нужно две вещи:

1. Создать управляемый bean, реализующий один из интерфейсов entity listener. Для валидации важны 3 интерфейса:
1. `BeforeDeleteEntityListener<T>`,
1. `BeforeInsertEntityListener<T>`,
1. `BeforeUpdateEntityListener<T>`
2. Добавить аннотацию `@Listeners` к объекту сущности, который планируется отслеживать.

И все.

По сравнению со стандартом JPA (JSR 338, раздел 3.5), listener-интерфейсы CUBA Platform типизированы, поэтому вам не нужно приводить аргумент с типом `Object` к типу сущности, чтобы начать с ней работать. Платформа CUBA добавляет связанным сущностям или вызывающим EntityManager возможность загружать и изменять другие сущности. Все эти изменения также будут вызывать соответствующий entity listener.

Также платформа CUBA поддерживает ["мягкое удаление"](https://doc.cuba-platform.com/manual-latest-ru/soft_deletion.html), подход, когда вместо реального удаления записей из базы данных они только помечаются как удаленные и становятся недоступными для обычного использования. Так, для мягкого удаления платформа вызывает слушателей `BeforeDeleteEntityListener` / `AfterDeleteEntityListener` в то время, как стандартные реализации вызывали бы слушателей `PreUpdate` / `PostUpdate`.

Давайте посмотрим на пример. Здесь Event listener bean подключается к классу сущности всего одной строкой кода: аннотацией `@Listeners`, которая принимает имя класса слушателя:

```
@Listeners("passportnumber_PersonEntityListener")
@NamePattern("%s|name")
@Table(name = "PASSPORTNUMBER_PERSON")
@Entity(name = "passportnumber$Person")
@ValidPassportNumber(groups = {Default.class, UiCrossFieldChecks.class})
@FraudDetectionFlag
public class Person extends StandardEntity {
    ...
}
```

[Person.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/global/src/com/company/passportnumber/entity/Person.java)

Сама реализация слушателя выглядит так:

```
/**
 * Checks that there are no other persons with the same 
 * passport number and country code
 * Ignores spaces in the passport number for the check.
 * So numbers "12 45 768007" and "1245 768007" and "1245768007" 
 * are the same for the validation purposes.
 */
@Component("passportnumber_PersonEntityListener")
public class PersonEntityListener implements
        BeforeDeleteEntityListener<Person>,
        BeforeInsertEntityListener<Person>,
        BeforeUpdateEntityListener<Person> {

    @Override
    public void onBeforeDelete(Person person, EntityManager entityManager) {
        if (!checkPassportIsUnique(person.getPassportNumber(), 
                                   person.getCountry(), entityManager)) {
            throw new ValidationException(
                "Passport and country code combination isn't unique");
        }
    }

    @Override
    public void onBeforeInsert(Person person, EntityManager entityManager) {
        // use entity argument to validate the Person object
        // entityManager could be used to access database 
        // if you need to check the data
        // throw ValidationException object if validation check failed
        if (!checkPassportIsUnique(person.getPassportNumber(), 
                                   person.getCountry(), entityManager))
            throw new ValidationException(
                "Passport and country code combination isn't unique");
    }

    @Override
    public void onBeforeUpdate(Person person, EntityManager entityManager) {
        if (!checkPassportIsUnique(person.getPassportNumber(), 
                                   person.getCountry(), entityManager))
            throw new ValidationException(
                "Passport and country code combination isn't unique");
    }

    ...
}
```

[PersonEntityListener.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/core/src/com/company/passportnumber/service/PersonEntityListener.java)

Entity listeners — отличный выбор, если:

* Необходимо выполнить проверку данных внутри транзакции до того, как объект сущности будет сохранен в БД;
* Необходимо проверить данные в БД в процессе валидации, например, проверить, что в наличии достаточно товара для принятия заказа;
* Нужно просмотреть не только объект сущности, вроде `Order`, но и объекты в связанные с сущностью, например, объекты `OrderItems` для сущности `Order`;
* Мы хотим отследить операции вставки, обновления или удаления только для некоторых классов сущностей, например, только для сущностей `Order` и `OrderItem`, и нам не нужно проверять изменения в других классах сущности во время транзакции.

### Transaction listeners

[CUBA transaction listeners](https://doc.cuba-platform.com/manual-latest-ru/transaction_listeners.html) также действуют в контексте транзакций, но, по сравнению с entity listeners, они вызываются для каждой транзакции базы данных. 

Эти дает им супер-силу:

* ничто не может ускользнуть от их внимания.

Но это же определяют их недостатки:

* их сложнее писать;
* они могут существенно снизить производительность;
* Их стоит писать очень внимательно: баг в transaction listener может помешать даже первичной загрузке приложения.

Итак, transaction listeners — хорошее решение, когда нужно проинспектировать разные типы сущностей по одному алгоритму, например, проверка всех данных на предмет кибер-мошенничества единым сервисом, который обслуживает все ваши бизнес-объекты.

![](&&&SFLOCALFILEPATH&&&kmv9cougmokpc3opta_dfk4egqk.jpeg)

Взгляните на образец, который проверяет, есть ли у сущности аннотация `@FraudDetectionFlag`, и, если есть, запускает детектор фрода. Повторюсь: имейте в виду, что этот метод вызывается в системе **до коммита каждой транзакции БД**, поэтому код должен стараться проверить как можно меньше объектов как можно быстрее.

```
@Component("passportnumber_ApplicationTransactionListener")
public class ApplicationTransactionListener 
                 implements BeforeCommitTransactionListener {

    private Logger log = LoggerFactory.getLogger(ApplicationTransactionListener.class);

    @Override
    public void beforeCommit(EntityManager entityManager, 
                             Collection<Entity> managedEntities) {
        for (Entity entity : managedEntities) {
            if (entity instanceof StandardEntity
                && !((StandardEntity) entity).isDeleted()
                && entity.getClass().isAnnotationPresent(FraudDetectionFlag.class)
                && !fraudDetectorFeedAndFastCheck(entity))
            {
                logFraudDetectionFailure(log, entity);
                String msg = String.format(
                        "Fraud detection failure in '%s' with id = '%s'",
                        entity.getClass().getSimpleName(), entity.getId());
                throw new ValidationException(msg);
            }
        }
    }

    ...
}
```

[ApplicationTransactionListener.java](https://github.com/dyakonoff/cuba-validation-examples/blob/master/passportnumber/modules/core/src/com/company/passportnumber/service/ApplicationTransactionListener.java)

Чтобы превратиться в transaction listener, управляемый bean должен реализовывать интерфейс `BeforeCommitTransactionListener` и метод `beforeCommit`. Transaction listener’ы связываются автоматически при запуске приложения. CUBA регистрирует все классы, реализующие `BeforeCommitTransactionListener` или `AfterCompleteTransactionListener` в качестве transaction listener’ов.

## Заключение

Bean validation [(JPA 303, 349 и 980)](https://beanvalidation.org/specification/) — это подход, который может служить надежной основой для 95% случаев валидации данных, встречающихся в корпоративном проекте. Главное преимущество такого подхода состоит в том, что большая часть логики валидации сконцентрирована прямо в классах доменной модели. Поэтому ее легко найти, легко читать и легко поддерживать. Spring, CUBA и многие другие библиотеки поддерживают эти стандарты и автоматически выполняют проверки в рамках валидации во время получения данных на UI слое, вызова validated-методов или процесса сохранения данных через ORM, поэтому Bean validation с точки зрения разработчика часто выглядит как магия.

Некоторые разработчики ПО рассматривают валидацию на уровне классов предметной модели как неестественную и слишком сложную, говорят, что проверки данных на уровне UI — достаточно эффективная стратегия. Однако, я считаю, что многочисленные точки валидации в компонентах и контроллерах UI — не самый рациональный подход. К тому же, методы валидации, перечисленные здесь, не выглядят такими неестественными, когда они интегрированы в платформу, в которой есть валидаторы бинов и listener’ы и которая автоматически интегрирует их с клиентским слоем.

В заключение, сформулируем правила, помогающие выбрать лучший метод валидации:

1. **JPA валидация** обладает ограниченной функциональностью, но является хорошим выбором для простейших ограничений в классах сущности, если такие ограничения могут быть отображены на DDL.
2. **Bean Validation** — гибкий, лаконичный, декларативный, многоразовый и удобный для чтения способ настроить большинство проверок в классах предметной области. В большинстве случаев это лучший выбор, если не нужно запускать валидации внутри транзакций.
3. **Валидация по контракту** это bean validation, но для вызовов методов. Используйте ее для входных и выходных параметров метода, например, в контроллерах REST.
4. **Entity listeners:** хотя они и не так декларативны, как аннотации Bean Validation, они отлично подходят для проверки больших графов объектов или проверок внутри транзакции БД. Например, когда нужно считать данные из БД для принятия решения. У Hibernate есть аналог таких слушателей.
5. **Transaction listeners** — опасное, но мощное оружие, работающее внутри контекста транзакции. Используйте его, когда в процессе исполнения нужно решить, какие объекты должны быть проверены или когда нужно проверить много разных типов сущностей по одному и тому же алгоритму валидации.

*PS: Надеюсь, эта статья освежила ваши знания о различных методах валидации в корпоративных приложениях на Java, и подкинула несколько идей о том, как оптимизировать архитектуру проектов, над которыми вы работаете.*

## Полезные ссылки

[Валидация в Java-приложениях](https://habr.com/ru/company/haulmont/blog/427543/)