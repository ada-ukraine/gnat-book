# Розділ 3 Відновлення після помилок

```{note}
Цей ШІ-переклад ще не відредаговано.
```

Сканер GNAT реалізує деякі базові методи відновлення помилок, які
спрощують реалізацію синтаксичного аналізатора. Синтаксичний аналізатор
GNAT має, як вважається, найкращий набір стратегій відновлення помилок
серед усіх використовуваних компіляторів Ada. У цій главі ми коротко
опишемо ці стратегії. Вона побудована наступним чином: Підрозділ 3.1
знайомить з методами відновлення помилок, реалізованими у сканері, а
підрозділ 3.2 описує механізми, що використовуються для ресинхронізації
синтаксичного аналізатора. Підрозділ 3.2.1 описує стек областей
видимості синтаксичного аналізатора, який використовується для обробки
вкладених областей видимості; підрозділ 3.1.1 обговорює використання
угоди про програмну оболонку для розрізнення ключових слів і визначених
користувачем ідентифікаторів, і, нарешті, підрозділи 3.2.3 і 3.2.4
обговорюють особливу обробку ключового слова \'is\', яке
використовується замість крапки з комою.

## 3.1 Відновлення помилок сканера

Сканер GNAT реалізує просту техніку відновлення помилок, яка спрощує
передачу помилок синтаксичному аналізатору. Після виявлення лексичної
помилки сканер видає відповідне повідомлення про помилку і повертає один
евристичний токен, який маскує помилку на наступних етапах компілятора.
Наприклад:

-   Якщо знайдено символ \"\[\", програма припускає, що програміст мав
    намір написати \"(\" (правильний символ, який використовується в
    мові Ada). Тому він видає відповідне повідомлення про помилку і
    повертає синтаксичному аналізатору токен \"ліва дужка\".

-   Якщо сканер виявляє невірну послідовність символів Ada, яка є
    дійсною в іншій стандартній мові, він припускає, що програміст мав
    намір використати відповідний еквівалент (якщо такий є) в Ada.
    Наприклад, перед послідовністю \"&&\" (яка є синтаксисом оператора і
    в мові C), програма припускає, що програміст мав намір використати
    оператор і в мові Ada. Тому він видає точне повідомлення про помилку
    \"&& should be \'and then\' \" і повертає токен \"and then\"
    синтаксичному аналізатору. Сканер має багато повідомлень про
    помилки, специфічних для програмістів на C.

У більшості випадків цей простий, але потужний механізм допомагає
маскувати лексичні помилки для синтаксичного аналізатора. Це спрощує
реалізацію синтаксичного аналізатора, якому не потрібно повторно
обробляти їх у багатьох контекстах. За додатковою інформацією зверніться
до підпрограм Scan, Nlit та Slit у пакунку Scn.

## 3.1.1 Використання оболонки ідентифікаторів

Сканер GNAT припускає, що користувач має певну узгоджену політику щодо
умовних позначень, які використовуються для розрізнення ключових слів і
визначених користувачем об\'єктів.

Сканер виводить цю угоду з першого ключового слова та ідентифікатора, з
яким він стикається (див. Обгортка пакунка). У наступному прикладі GNAT
використовує правило верхнього/нижнього регістру для ідентифікаторів,
щоб вважати слово exception ідентифікатором з відступом, а не початком
обробника винятків (див. Підпрограма сканування зарезервованого
ідентифікатора).

## 3.2 Відновлення помилок парсеру

Синтаксичний аналізатор GNAT містить складну систему виправлення
помилок, яка, серед іншого, враховує відступи при спробі виправити
помилки області видимості. Коли виникає помилка, виконується виклик
однієї процедури синтаксичного аналізатора для запису помилки (див.
пакет Errout). Якщо синтаксичний аналізатор може відновитися локально,
він маскує помилку для наступних етапів інтерфейсу, генеруючи вузли AST
так, ніби синтаксис є правильним, і синтаксичний аналіз продовжується
безперешкодно. З іншого боку, якщо помилка являє собою ситуацію, з якої
синтаксичний аналізатор не може відновитися локально, то

виключення Error Resync) генерується після виклику підпрограми, яка
фіксує помилку. Обробники винятків розташовані у стратегічних точках для
ресинхронізації синтаксичного аналізатора.

Наприклад, коли в операторі виникає помилка, обробник переходить до
наступної крапки з комою і продовжує сканування з неї.

У вихідних текстах GNAT кожна процедура синтаксичного аналізу має
примітку із заголовком \"Відновлення після помилки\", яка показує, чи
може вона поширювати виключення Error Resync (див. файли par-ch2.adb -
par-ch13.adb). Щоб уникнути поширення виключення, процедура повинна або
містити власний обробник для цього виключення, або не викликати інші
процедури, які поширюють виключення.

### 3.2.1 Стек області видимості синтаксичного аналізатора

Багато правил мови визначають синтаксичну область застосування (правила,
що закінчуються символом \"end\"). Наприклад, правила синтаксису
пакунків, підпрограм і всіх операторів керування потоком. Синтаксичний
аналізатор GNAT використовує стек області видимості для запису контексту
області видимості. Запис робиться, коли синтаксичний аналізатор
зустрічає відкриття вкладеної конструкції, а потім пакет Endh
використовує цей стек для обробки рядків, що закінчуються символом
\'end\' (у тому числі для коректної обробки помилок вкладеності символу
\'end\').

У випадку ресинхронізації синтаксичного аналізу за допомогою механізму
винятків (див. розділ 3.2), розташування обробників винятків є таким, що
ніколи не повинно бути можливості передати керування через процедуру,
яка зробила запис у стеку області видимості, що робить вміст стеку
недійсним.

У деяких випадках в кінці синтаксичної області видимості програмісту
дозволяється вказати ім\'я (наприклад, в кінці тіла підпрограми); інші
правила області видимості мають жорсткий формат (наприклад, в кінці
визначення типу запису). У першому випадку семантичною помилкою є
відкриття синтаксичної області видимості з іменем, а закриття з іншим
іменем. Хоча багато компіляторів Ada виявляють цю помилку на етапі
семантичного аналізу, GNAT використовує стек області видимості
синтаксичного аналізатора, щоб виявити її якнайшвидше і таким чином
спростити семантику.

### 3.2.2 Приклад 1: Використання відступу

Наступний приклад поєднує використання стека області видимості з
відступом до

збігаються з твердженням:

Зауважте, що більш традиційний підхід до виправлення помилок призвів би
до двох помилок: він би визначив кінець за допомогою оператора if,
поскаржився б на відсутність \"if\", а потім поскаржився б на
відсутність кінця для самої процедури.

### 3.2.3 Приклад 2: Обробка крапки з комою, що використовується замість \"is

Два контексти, в яких крапка з комою могла бути помилково використана
замість \"is\", знаходяться на зовнішньому рівні та в межах
декларативної області. Перший випадок відповідає наступному прикладу:

У цьому випадку синтаксичний аналізатор GNAT знає, що щось не так, як
тільки зустрічає Q (або \'begin\', якщо немає декларацій), і відразу ж
може діагностувати, що крапка з комою мала б бути \'is\'. Ситуація в
області декларацій складніша і відповідає наступному прикладу:

У цьому випадку синтаксична помилка (рядок \<1\>) має синтаксис
оголошення підпрограми \[AAR95, розділ 6-1\]. Тому оголошення Q
читається як таке, що належить до зовнішньої області. Синтаксичний
аналізатор не знає, що це була помилка, доки не зустрінеться з \'begin\'
(рядок \<2\>). На цьому етапі з синтаксичної точки зору все ще не
зрозуміло, що щось не так, оскільки \'begin\' може належати до
охоплюючої синтаксичної області. Однак синтаксичний аналізатор GNAT
використовує трохи семантичних знань і помічає, що тіло X відсутнє, тому
діагностує помилку як крапку з комою на місці \'is\' у рядку
підпрограми. Для контролю цього аналізу використовуються деякі глобальні
змінні з префіксом \"SIS\", які вказують на те, що ми маємо оголошення
підпрограми, тіло якої є необхідним і ще не знайдене. Наприклад, змінна
SIS Entry Active вказує на те, що знайдено оголошення підпрограми, але
ще не знайдено тіло цієї підпрограми. Префікс означає обробку
\"Підпрограма IS\". З активним записом SIS може статися п\'ять речей:

1.  Якщо зустрічається \'begin\' при активному записі в SIS, то ми маємо
    саме ту ситуацію, в якій ми знаємо, що тіло підпрограми відсутнє.
    Після видачі повідомлення про помилку синтаксичний аналізатор
    перебудовує AST: він змінює специфікацію на тіло, повторно
    з\'єднуючи оголошення, знайдені між специфікацією і словом begin.

2.  Зустрічається інша декларація або тіло підпрограми. У цьому випадку
    запис перезаписується інформацією для нового оголошення підпрограми.
    Таким чином, синтаксичний аналізатор GNAT не розпізнає деякі
    вкладені випадки, але це і не варто зусиль.

3.  Зустрічається вкладена декларативна область (наприклад, декларація
    пакунка або тіло пакунка). На початку такої вкладеної області
    індикація активності SIS скидається. Знову ж таки, як і в
    попередньому випадку, синтаксичний аналізатор пропускає деякі
    вкладені регістри, але не варто витрачати зусилля на складання та
    розкладання інформації SIS.

4.  Правильна прагма \'interface\' або \'import\' надає тіло, якого
    бракує. У цьому випадку синтаксичний аналізатор скидає запис.

5.  Синтаксичний аналізатор натрапляє на кінець декларативної області,
    не зустрівши спочатку \"begin\". У такій ситуації синтаксичний
    аналізатор просто скидає запис: у ньому відсутня частина тіла, але
    розумніше дозволити подальшій семантичній перевірці виявити це.

### 3.2.4 Приклад 3: Обробка \"is\" використовується замість крапки з комою

Це дещо складніша ситуація, і хоча синтаксичний аналізатор GNAT не може
виявити її у всіх випадках, він робить все можливе, щоб виявити типові
ситуації, які виникають у результаті операції \"вирізання та вставки\",
коли забувається змінити \"is\" на крапку з комою. Розглянемо наступний
приклад:

Проблема полягає у тому, що ділянка тексту від рядка \<1\> до рядка
\<2\> синтаксично є правильним тілом процедури, і небезпека полягає у
тому, що синтаксичний аналізатор занадто пізно виявить, що щось не так
(дійсно, більшість компіляторів поводитимуться незручно у наведеному
вище прикладі).

Щоб контролювати цю ситуацію, синтаксичний аналізатор GNAT уникає
ковтання останнього \'end\', якщо є впевненість, що це призведе до
помилки. Зокрема, синтаксичний аналізатор не прийме \"кінець\", якщо за
ним одразу слідує кінець файлу, \"з\" або \"окремий\" (усі лексеми, які
сигналізують про початок блоку компіляції, а отже, дозволяють залишити
\"кінець\" на зовнішньому рівні). За більш детальною інформацією про цей
аспект обробки зверніться до пакунка Endh. Аналогічно, якщо у пакунку,
що його містить, відсутній begin, то результатом буде повідомлення про
відсутність begin, яке відсилає до заголовка підпрограми. Таке
повідомлення про помилку не так вже й погано (це вже значне покращення
порівняно з тим, що роблять багато синтаксичних аналізаторів), але воно
не є ідеальним, оскільки оголошення, що слідують за \'is\', асоціюються
з неправильною областю видимості.

Щоб відловити принаймні деякі з цих випадків, синтаксичний аналізатор
GNAT виконує наступні додаткові кроки. По-перше, тіло підпрограми
позначається як таке, що містить підозрілий символ \'is\', якщо за
рядком оголошення слідує рядок, який починається з символу, що може
починати оголошення у тому самому стовпчику або ліворуч від стовпчика, у
якому починається \'функція\' або \'процедура\' (нормальним стилем є
відступ для оголошень, які насправді належать до підпрограми). Якщо у
такій підпрограмі зустрічається пропущений початок або кінець, то
синтаксичний аналізатор вирішує, що \"is\" має бути крапкою з комою, і
вузол тіла підпрограми позначається (встановлюючи прапорець Bad Is
Detected у true). Це не робиться для процедур бібліотечного рівня,
оскільки вони повинні мати тіло.

Обробка декларативної частини перевіряє, чи останню відскановану
декларацію було позначено таким чином, і якщо так, то дерево
модифікується, щоб відобразити \"is\", яке інтерпретується як крапка з
комою.

## 3.3 Підсумок

У GNAT реалізовано евристичний механізм відновлення помилок, який
спрощує реалізацію синтаксичного аналізатора: при виявленні помилки
сканер генерує відповідне повідомлення про помилку, обчислює
лексему-замінник і повертає її синтаксичному аналізатору. У більшості
випадків цей простий, але потужний механізм допомагає синтаксичному
аналізатору продовжувати роботу так, ніби у вихідній програмі немає
лексичних помилок.

У випадку простих помилок синтаксичний аналізатор маскує помилку і
генерує правильні вузли так, ніби вихідна програма є коректною. У
випадку складних помилок синтаксичний аналізатор реалізує механізм
ресинхронізації на основі обробників винятків.