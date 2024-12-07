# 🤖 Розділ 2. Огляд архітектури інтерфейсу

```{note}
Цей ШІ-переклад ще не відредаговано.
```

Інтерфейс GNAT складається з чотирьох фаз, які взаємодіють за допомогою
компактного дерева абстрактного синтаксису (AST): лексичний аналіз,
синтаксичний аналіз, семантичний аналіз і розширення. У цій главі
наведено огляд архітектури цих фаз. Він побудований наступним чином: У
підрозділі 2.1 представлено архітектуру сканера; у підрозділі 2.2 подано
огляд синтаксичного аналізатора, описано високорівневу специфікацію
вузлів АСТ та механізми, що використовуються для його ресинхронізації у
випадку синтаксичних помилок; у підрозділі 2.3 описано архітектуру
семантичного аналізатора, і, нарешті, у підрозділі 2.4 обговорено
архітектуру розширювача.

## 2.1 Сканер

З міркувань ефективності для створення GNAT-сканера не було використано
жодного автоматичного інструменту.

Це підпрограма синтаксичного аналізатора, яка зчитує вхідні символи,
визначає наступну лексему і повертає її синтаксичному аналізатору. Щоб
забезпечити підтримку різних операційних систем і різних мов, його
розроблено для роботи з різними кодуваннями символів (порівняйте з
пакунком Csets). На рисунку 2.1 показано його архітектуру: пакунок Scn
містить більшу частину реалізації сканера; пакунок Scans містить
визначення токенів і стан автомата. Нарешті, пакет Snames містить
стандартні імена (ключові слова, прагматики та атрибути Ada).
Низькорівневий пакет Namet займається зберіганням та пошуком імен, а
пакет Opt містить глобальні прапори, які встановлюються перемикачами
командного рядка і на які посилається весь компілятор.

![Архітектура сканера GNAT](f2_1.png)

Рисунок 2.1: Архітектура сканера GNAT

## 2.2 Парсер

Синтаксичний аналізатор GNAT - це синтаксичний аналізатор рекурсивного
спуску з ручним кодуванням. Основними причинами, які виправдовують цей
вибір (а не традиційний академічний вибір табличного синтаксичного
аналізатора, згенерованого інструментом), є \[CGS94, розділ 3.2\]:

-   Кращі повідомлення про помилки. GNAT генерує чіткі та зрозумілі
    повідомлення про помилки. Навіть у випадку серйозних структурних
    помилок, зокрема, заміни \";\" та \"is\" між специфікацією та тілом
    підпрограми, синтаксичний аналізатор надсилає точне та зрозуміле
    повідомлення. Висхідні синтаксичні аналізатори мають серйозні
    труднощі з такими помилками.

-   Ясність. Синтаксичний аналізатор GNAT точно дотримується граматики,
    наведеної у довіднику з мови Ada \[AAR95\]. Це має очевидні
    педагогічні переваги, оскільки синтаксичний аналізатор можна легко
    читати разом з ARM, а також полегшує його підтримку, наприклад, для
    додавання нових методів відновлення помилок. Граматика мови Ada,
    надана ARM, є неоднозначною, і синтаксичний аналізатор, керований
    таблицями, був би змушений модифікувати граматику, щоб зробити її
    прийнятною для методів LL (1) або LALR (1). Для синтаксичного
    аналізатора рекурсивного спуску таких проблем не виникає, оскільки
    за необхідності він може виконати довільний перегляд.

-   Продуктивність. Синтаксичний аналізатор GNAT працює так само швидко,
    як і будь-який синтаксичний аналізатор на основі таблиць Ada, і,
    можливо, швидше, ніж синтаксичний аналізатор LALR.

У більшості випадків синтаксичний аналізатор орієнтується на наступну
лексему, надану сканером. Однак у випадку неоднозначних синтаксичних
правил синтаксичний аналізатор зберігає стан сканера і повторно
звертається до нього, щоб переглянути наступні лексеми і таким чином
вирішити неоднозначність (див. Зберегти стан сканування і Відновити стан
сканування у пакунку Сканування).

На додаток до перевірки синтаксису, синтаксичний аналізатор GNAT будує
дерево абстрактного синтаксису (АСТ), тобто структуроване представлення
вихідної програми. Це дерево згодом використовується семантичним
аналізатором для виконання всіх статичних перевірок програми, тобто всіх
контекстно-залежних правил легальності мови.

На архітектурному рівні основною підпрограмою синтаксичного аналізатора
GNAT є функція Ada (Par), яка викликається для аналізу кожної одиниці
компіляції. Код синтаксичного аналізатора організовано у вигляді набору
пакетів (підрозділів функції верхнього рівня), кожен з яких містить
підпрограми синтаксичного аналізу, пов\'язані з однією главою довідника
з мови Ada \[AAR95\]. Наприклад, пакет Par.Ch2 містить усі підпрограми
синтаксичного аналізу, пов\'язані з другою главою довідника Ada (див.
Рисунок 2.2). Крім того, назви підпрограм синтаксичного аналізу
відповідають фіксованому правилу: Префікс \"P\", за яким слідує назва
відповідного правила синтаксису Ada (наприклад, P Compilation Unit).

![Структура синтаксичного аналізатора GNAT](f2_2.png)

Рисунок 2.2: Структура синтаксичного аналізатора GNAT

Синтаксичний аналізатор GNAT також має декілька додаткових підрозділів:
Пакет Endh, містить підпрограми для аналізу завершення синтаксичних
областей; пакет Sync, містить підпрограми для пересинхронізації
синтаксичного аналізатора після синтаксичних помилок (див. розділ 3);
пакет Tchk, містить підпрограми для спрощення перевірки лексем. Розділ
3); пакет Tchk містить підпрограми, що спрощують перевірку лексем;
процедура Labl обробляє неявні оголошення міток; процедура Load керує
завантаженням до основної пам\'яті послідовних блоків компіляції;
функція Prag аналізує прагматику, що впливає на поведінку синтаксичного
аналізатора (наприклад, перевірку лексичного стилю), і, нарешті, пакет
Util містить підпрограми загального призначення, що використовуються
усіма процедурами синтаксичного аналізу.

Кожна процедура синтаксичного аналізу виконує два основні завдання: (1)
перевіряє, що частина вихідного тексту підпорядковується синтаксису
одного конкретного правила мови, і (2) будує відповідне піддерево
абстрактного синтаксису (див. Малюнок 2.3).

![Побудова абстрактного дерева синтаксису](f2_3.png)

Рисунок 2.3: Побудова абстрактного дерева синтаксису.

### 2.2.1 Дерево абстрактного синтаксису

Дерево абстрактного синтаксису GNAT має два типи вузлів: внутрішні
(структурні) вузли, які представляють синтаксичну структуру програми, і
розширені вузли, які зберігають інформацію про об\'єкти мови Ada
(ідентифікатори, символи операторів і оголошення символьних літералів).
Внутрішні вузли мають 5 полів загального призначення, які можна
використовувати для посилань на інші вузли, списки вузлів (тобто список
операторів у блоці Ada), імена, літерали, універсальні цілі числа, числа
з плаваючою комою або коди символів. Вузли сутностей мають 23 поля
загального призначення та велику кількість булевих прапорів, які
використовуються для зберігання у дереві всіх релевантних семантичних
атрибутів кожної сутності. В інших компіляторах ця інформація зазвичай
зберігається в окремій таблиці символів.

На рисунку 2.4 описано пакети GNAT, що беруть участь у роботі з AST.
Низькорівневий пакет Atree реалізує абстрактний тип даних, який містить
визначення, пов\'язані з вузлами та сутностями структури, а також
підпрограми для створення, копіювання та видалення вузлів і підпрограми
для читання та модифікації полів загального призначення.

Низькорівневий пакет Nlists забезпечує підтримку роботи зі списками
вузлів. Пакети Sinfo та Einfo містять високорівневу специфікацію вузлів,
тобто високорівневі імена, пов\'язані з низькорівневими полями вузлів
загального призначення, та підпрограми для читання і модифікації цих
полів з їхніми високорівневими іменами. У пакунку Nmake є підпрограми
для створення високорівневих вузлів із синтаксичною та семантичною
інформацією.

Розглянемо формат високорівневої специфікації вузлів на прикладі.
Правило синтаксису мови Ada для тіла пакунка наступне:

![Пакунки з абстрактним деревом синтаксису](f2_4.png)

Рисунок 2.4: Пакунки з абстрактним деревом синтаксису.

Відповідний високорівневий вузол задається у пакунку Sinfo наступним
чином:

\...

У першому рядку вказується тип вузла (N Package Body), який є
перелічуваним літералом типу Sinfo.Node Kind; у другому рядку
вказується, що координата джерела (Sloc) для вузла є координатою джерела
ключового слова, яке є першою лексемою у постановці. Ця координата
джерела використовується щоразу, коли на даній конструкції з\'являються
помилки або попередження. Наступні рядки визначають високорівневі імена,
надані полям загального призначення. Їхній формат такий: (1)
високорівнева назва поля (вибрана за синтаксичним значенням) і, (2) тип
даних та розміщення відповідного низькорівневого поля загального
призначення (ця інформація міститься у круглих дужках). Крім того, деякі
поля можуть визначати значення ініціалізації за замовчуванням Наприклад,
поле з назвою Handled Statement Sequence посилається на вузол, який
представляє послідовність операторів та обробників виключень, знайдених
у необов\'язковій ініціалізації пакунка. Це поле розміщується у
четвертому низькорівневому полі загального призначення вузла і
встановлюється у значення Empty, якщо тіло пакунка не містить коду
ініціалізації. Для однакової обробки AST ідентичні поля вузла,
пов\'язані з різними вузлами, завжди призначаються до тих самих
низькорівневих полів загального призначення. Останні два рядки
визначають два семантичні атрибути (позначені суфіксом \"-Sem\").
Семантичні атрибути обчислюються і додаються до дерева семантичним
аналізатором на наступному етапі роботи компілятора (див. розділ 2.3).
На рисунку 2.5 показано піддерево, пов\'язане з цією високорівневою
специфікацією вузла.

![Абстрактне синтаксичне піддерево](f2_5.png)

Рисунок 2.5: Абстрактне синтаксичне піддерево, пов\'язане з правилом
тіла пакунка.

Високорівнева специфікація вузлів AST зчитується інструментами GNAT
xsinfo, xtreeprs та xnmake, які генерують деякі додаткові пакунки
інтерфейсу Ada, що використовують низькорівневий пакунок Atree для
забезпечення вказаної функціональності.

## 2.3 Семантичний аналізатор

Аналізатор семантики GNAT обходить дерево абстрактного синтаксису,
побудоване синтаксичним аналізатором, перевіряє статичну семантику
програми і декорує АСТ, тобто додає семантичну інформацію до вузлів АСТ
(див. Рисунок 2.6).

![Оформлення дерева абстрактного синтаксису](f2_6.png)

Рисунок 2.6: Оформлення дерева абстрактного синтаксису.

Загалом, статичний аналіз, який виконує компілятор, передбачає виконання
таких завдань: (1) групування об\'єктів у областях видимості та
визначення імен, на які посилаються, (2) обробка закритих типів, (3)
обробка дискримінантів, (4) аналіз та інстанціювання типових об\'єктів,
(5) обробка заморожування об\'єктів. Ці теми будуть детально розглянуті
у наступних розділах.

На рисунку 2.7 показано архітектуру семантичного аналізатора GNAT.
Загальна структура подібна до структури синтаксичного аналізатора, тобто
вона паралельна організації ARM. Наприклад, Sem Ch3 займається обробкою
типів та декларацій, Sem Ch9 - паралелізмом, а Sem Ch12 - узагальненнями
та екземплярами. Назви окремих підпрограм семантичного аналізу
підпорядковуються фіксованому правилу: вони мають префікс \"Analyze \"і
суфікс, що означає аналізовану мовну конструкцію, тобто. Analyze
Compilation Unit. Виняток з цього загального правила становлять пакети
Sem Prag і Sem Attr, які відповідають елементам мови, описаним в ARM, а
також є базовим механізмом розширення для компілятора.

![Структура семантичного аналізатора](f2_7.png)

Рисунок 2.7: Структура семантичного аналізатора.

Крім того, семантичний аналізатор GNAT має кілька утиліт для
спеціалізованих цілей. Sem Disp містить підпрограми, що займаються
аналізом тегованих типів та динамічною диспетчеризацією; Sem Dist
містить підпрограми, що аналізують Додаток розподілених систем Ada
\[AAR95, Додаток E\]; Sem Elab містить підпрограми, що займаються
порядком розробки набору одиниць компіляції; Sem Eval містить
підпрограми, що займаються оцінкою статичних виразів під час компіляції
та перевіркою легальності статичності виразів і типів. Sem Intr аналізує
оголошення внутрішніх операцій; Sem Mech має одну підпрограму, яка
аналізує оголошення механізмів виклику підпрограм (необхідна для VMS
версії GNAT). Пакет Sem Case містить підпрограми для обробки списків
дискретних варіантів (такі списки можуть мати 3 різні конструкції:
оператори case, агрегати масивів і варіанти записів); пакет Sem Util
містить підпрограми загального призначення, які використовуються всіма
пакетами семантики. Нарешті, пакет Sem Type містить підпрограми для
обробки наборів перевантажених сутностей, а пакет Sem Res - реалізацію
відомого двопрохідного алгоритму, який вирішує проблему перевантажених
сутностей (описано у Розділі 5). Пакет Sem Aggr концептуально є
розширенням Sem Res; його було винесено в окремий пакет через складність
коду, який обробляє агрегати.

Основний пакет (Sem) реалізує диспетчер, який отримує один вузол AST і
викликає відповідну підпрограму аналізу (див. Рисунок 2.8). Викликані
підпрограми рекурсивно викликають Analyze для обходу дерева зверху вниз.

![Диспетчеризація вузлів АСД](f2_8.png)

Рисунок 2.8: Диспетчеризація вузлів абстрактного синтаксичного дерева.

Процедури розв\'язання викликаються процедурами аналізу для розв\'язання
неоднозначних вузлів або перевантажених об\'єктів. Наприклад, синтаксис
Ada для оператора виклику процедури точно такий самий, як і для
оператора виклику запису. Враховуючи цю неоднозначну синтаксичну
специфікацію, синтаксичний аналізатор GNAT генерує той самий вузол N
виклику процедури для обох випадків, а семантичний аналізатор повинен
проаналізувати контекст, визначити природу об\'єкта, що викликається, і,
за необхідності, замінити початковий вузол на вузол оператора виклику
запису (який буде піддано зовсім іншому розширенню, ніж виклик
процедури). Для розв\'язання перевантажених сутностей GNAT реалізує
добре відомий двопрохідний алгоритм. Під час першого (висхідного)
проходу збирається множина можливих значень імені. Під час другого
проходу використовується тип, заданий контекстом, для вирішення
неоднозначностей і вибору унікального значення для кожного
перевантаженого ідентифікатора у виразі. Це детально описано у Розділі
5.

## 2.4 Експандер

Розширювач GNAT виконує AST-перетворення для тих конструкцій, які не
мають близького еквівалента у семантиці C-рівня бекенда. (див. рисунок
2.9). Основними його розширеннями є \[CGS94, розділ 3.3\]:

-   Замінити вузли, які представляють високорівневі конструкції Ada, на
    вузли, які представляють абстракції нижчого рівня. Наприклад, вузол,
    який представляє тіло задачі на Ada, замінюється одним вузлом, який
    представляє процедуру, плюс один виклик підпрограми Бібліотеки часу
    виконання, яка створює відповідний потік керування (див. глави
    9-13).

-   Замініть вузли, які представляють узагальнену екземпляризацію,
    копією відповідного AST, з відповідними замінами (див. Розділ 7).

-   Підпрограми підтримки типів (Build Type Support Subprograms, TSS),
    які є внутрішньо згенерованими підпрограмами, пов\'язаними з певними
    типами. Наприклад, підпрограми неявної ініціалізації та підпрограми
    для реалізації атрибутів вводу/виводу Ada (див. Package Exp TSS).

![Розширення АСД](f2_9.png)

Рисунок 2.9: Розширення дерева абстрактного синтаксису.

Оскільки генерація коду вимагає, щоб кожен вузол АСТ містив усі
необхідні семантичні атрибути, кожна операція розширення повинна
супроводжуватися додатковою семантичною обробкою згенерованих фрагментів
(див. стрілки у зворотному напрямку на рисунку 2.9). В результаті дві
фази (аналіз і розширення) рекурсивно інтегруються в один обхід АПД.
Після того, як весь АСТ проаналізовано та розширено, отриманий АСТ
передається до Gigi для генерації фрагментів дерева GCC, які є вхідними
даними для внутрішнього генератора коду.

Архітектура GNAT Expander відповідає тій самій схемі, що й на попередніх
етапах: підпрограми розширення згруповано у пакети відповідно до
розділів довідника Ada (див. рис. 2.10), а назви підпрограм відповідають
фіксованому правилу: Префікс \"Expand \", за яким слідує відповідне
правило мови (тобто. Expand Compilation Unit). Подібно до семантичного
аналізатора, пакет Expander реалізує диспетчер, який отримує один вузол
і викликає відповідну підпрограму розширювача.

![Архітектура GNAT Expander](f2_10.png)

Рисунок 2.10: Архітектура GNAT Expander

Розширювач GNAT має такі пакунки: Exp Prag, який групує підпрограми для
розширення прагм; Exp Attr містить підпрограми для розширення атрибутів
Ada; Exp Aggr містить підпрограми для розширення агрегатів Ada; Exp Disp
з підпрограмами для розширення тегованих типів та динамічної
диспетчеризації; Exp Dist містить підпрограми для генерації заглушок Ada
Distribution Annex \[AAR95\]; Exp Fixd - підпрограми для розширення
операцій над типами даних з фіксованою комою; Exp Pakd - підпрограми для
розширення упакованих масивів; Exp Strm - підпрограми для побудови
потокових підпрограм для складених типів (масивів і записів); Exp TSS -
підпрограми для роботи з підпрограмою підтримки типів (TSS); Exp Code -
підпрограми для розширення оператора коду Ada \[AAR95, розділ 13.8\], що
використовуються для додавання інструкцій машинного коду до вихідних
програм на Ada; Exp Util, підпрограми утиліт, що використовуються
спільно з підпрограмами розширення; і, нарешті, Exp Dbug - пакет з
підпрограмами, що генерують спеціальні декларації, які використовуються
налагоджувачем.

## 2.5 Підсумок

Сканер GNAT реалізує детермінований автомат, який викликається
синтаксичним аналізатором для отримання наступної лексеми. Синтаксичний
аналізатор GNAT - це синтаксичний аналізатор рекурсивного спуску, який
не лише перевіряє синтаксис вихідних текстів, але й генерує відповідне
дерево абстрактного синтаксису (АСТ). АСТ має два типи вузлів:
структурні вузли, які представляють структуру програми, і вузли
сутностей, які зберігають інформацію про сутності Ada. Тому GNAT не має
окремої таблиці символів; вся інформація, яка традиційно зберігається у
цій таблиці, зберігається у сутностях AST.

Фаза семантичного аналізу виконує низхідний обхід АСТ для статичного
аналізу програми та оформлення АСТ. Ця фаза реалізує добре відомий
алгоритм двох проходів для вирішення перевантажених сутностей. На етапі
розширення високорівневі вузли замінюються піддеревами з низькорівневими
вузлами, які забезпечують еквівалентну семантику і можуть бути оброблені
генератором коду GCC.

Для полегшення читання вихідних текстів архітектура синтаксичного
аналізатора, семантики та розширювача відповідає фіксованій схемі:
підпрограми згруповано у пакети відповідно до довідника Ada, а їхні
імена відповідають фіксованому правилу: один фіксований префікс плюс
відповідне правило довідника Ada. Основний пакет семантики та розширювач
реалізують диспетчер, який отримує на вхід вузол AST і викликає
відповідну процедуру обробки.

