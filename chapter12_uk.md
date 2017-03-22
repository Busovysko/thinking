# Обробка помилок і виключень

Один з основоположних принципів філософії *Java* полягає в тому, що «погано написана програма не повинна запускатися. В ідеалі помилки повинні виявлятися під час компіляції, перед запуском програми. Однак не всі помилки вдається виявити в цей час. Решту проблем приходиться вирішувати під час роботи програми, за допомогою механізму, який дозволяє джерелу помилки передати необхідну інформацію про неї одержувачу - а останній справляється з виниклими труднощами. Удосконалена система відновлення після помилок входить в число найважливіших факторів, що впливають на надійність коду. Відновлення особливо важливо в мові *Java*, на якій часто пишуться програмні компоненти, що використовуються іншими сторонами. Надійна система може бути побудована тільки з надійних компонентів. Уніфікована модель передачі інформації про помилки в *Java* дозволяє компонентам передавати інформацію про виниклі проблеми в клієнтський код. Механізм виключень значно спрощує створення великих надійних програм, зменшує обсяг необхідного коду і підвищує впевненість в тому, що в додатку не буде необробленої помилки. Освоїти роботу з виключеннями нескладно, і це, одна з мовних можливостей, здатних принести негайну і значну вигоду в ваших проектах. У цьому розділі буде показано, як правильно організувати обробку виключень в програмі, а також як згенерувати власне виключення, якщо якийсь з ваших методів стикається з непередбачуваними труднощами. 

## Основні виключення

Виключенням називається проблема, через яку нормальне продовження роботи методу, або частини програми, що виконуються в даний момент, стає неможливим. Важливо відрізняти виключення від «звичайних» помилок, коли в поточному контексті є достатньо інформації для усунення проблем. У виключній ситуації обробити виключення в поточному контексті неможливо, тому що ви не маєте необхідної інформації. Залишається єдиний вихід - покинути поточний контекст і передати проблему на більш високий рівень. Саме це і відбувається при генерації виключення.
 
Найпростішим прикладом є ділення. Можливе ділення на нуль може бути виявлено перевіркою відповідної умови. Але що робити, якщо знаменник виявився нулем? Можливо, в контексті поточного завдання відомо, як слід вчинити з нульовим знаменником. Але, якщо нульовий знаменник виник несподівано, ділення в принципі неможливе, і тоді необхідно генерувати виключення, а не продовжувати виконання програми.
 
При генерації виключення відбувається відразу кілька речей. По-перше, створюється об'єкт, який являє собою виключення, - точно так само, як і будь-який інший об'єкт в *Java* (в купі, оператором ***new***). Далі поточний потік виконання (той самий, де сталася помилка) зупиняється, і посилання на об'єкт, що представляє виключення, витягується з поточного контексту. З цього моменту включається механізм обробки виключень, який починає пошук відповідного місця програми для передачі виключення. Таким місцем є обробник виключень, який намагається вирішити проблему так, щоб програма могла знову спробувати виконати проблемну операцію, або просто продовжила своє виконання.
 
Як простий приклад видачі виключення уявіть посилання на об'єкт ***t***. Можливо, отримане вами посилання не була ініціалізоване; варто перевірити цю обставину, перш ніж викликати методи з використанням цього посилання. Щоб передати інформацію про помилку на більш високий рівень, створіть об'єкт, який представляє передану інформацію, і «запустіть» його з поточного контексту. Тим самим ви згенеруєте виключення. Ось як це виглядає:
 
``` java
if(t == null) throw new NullPointerException();
```
 
Генерується виключення, яке дозволяє вам - в поточному контексті - перекласти з себе відповідальність, не замислюючись про майбутнє. Помилка, немов за помахом чарівної палички, обробляється десь в іншому місці (незабаром ми дізнаємося, де саме).
 
Один з основоположних аспектів виключень полягає в тому, що при виникненні небажаних ситуацій виконання програми не триває по звичайному шляху. Виключення дозволяють вам (в крайньому випадку) зупинити програму і повідомити що виникли проблеми, або (в ідеалі) розібратися з проблемою і повернути програму в стабільний стан. 

### Аргументи виключення

Виключення, як і будь-які об'єкти *Java*, створюються в купі оператором ***new***, який виділяє пам'ять і викликає конструктор. У всіх стандартних виключенях існує два конструктора: стандартний (за замовчуванням) і інший, з строковим аргументом, в якому можна розмістити відповідну інформацію про виключення:
 
``` java
throw new NullPointerException("t = null");
```
 
Переданий рядок потім може бути отриманий різними способами, про що буде розказано пізніше. 

Ключевое слово ***throw*** тягне за собою ряд досить цікавих дій. Виключення створюється за допомогою ***new***. Посилання на вказаний об'єкт передається команді ***throw***. Фактично цей об'єкт «повертається» методом, незважаючи на те що для об'єкта, що повертається зазвичай передбачений зовсім інший тип. Таким чином, спрощено можна говорити про обробку виключень як про альтернативний механізм повернення з виконуваного методу (втім, з цією аналогією не варто заходити надто далеко). Генерація виключень також дозволяє виходити з простих блоків видимості. В обох випадках повертається об'єкт виключення і відбувається вихід з поточного методу, або блоку. Але вся схожість із звичайним поверненням з методу на цьому закінчується, оскільки при поверненні з виключення ви потрапляєте зовсім не туди, куди потрапили б при нормальному виклику методу. (Оброблювач виключення може перебувати дуже «далеко» - на відстані декількох рівнів в стеці виклику - від методу, де виникла виняткова ситуація.)

Взагалі кажучи, можна порушити будь-який тип виключень, що походять від об'єкта ***Throwable*** (кореневий клас ієрархії виключень). Зазвичай для різних типів помилок порушуються різні типи виключень. Інформація про те що помилку яка міститься всередині об'єкта виключення, так і вказується побічно в самому типі цього об'єкта, щоб хтось на більш високому рівні зумів з'ясувати, як вчинити з виключенням. (Нерідко саме тип об'єкта виключення є єдино доступною інформацією про помилку, в той час як всередині об'єкта ніякої корисної інформації немає.) 

## Перехоплення виключень

Щоб побачити, як перехоплюються помилки, спочатку слід засвоїти поняття захищеної секції - тієї частини програми, в якій можуть відбутися виключення і за якою слідує спеціальний блок, який відповідає за обробку цих виключень.
 
### Блок try
Якщо ви «перебуваєте» всередині методу і ініціюєте виключення (або це робить інший викликаний метод), цей метод завершить роботу при виникненні виключення. Але якщо ви не хочете, щоб оператор ***throw*** завершив роботу методу, розмістіть в методі спеціальний блок для перехоплення виключення - так званий блок ***try***. Цей блок представляє собою просту область дії, якій передує ключове слово ***try***:
 
``` java
try {
	// Фрагмент, здатний порушувати виключення
}
```
 
Якби не обробка виключень, для ретельної перевірки помилок вам довелося б додати до виклику кожного методу додатковий код для перевірки помилок - навіть при багаторазовому виклику одного методу. З обробкою виключень весь код розміщується в блоці ***try***, який і перехоплює всі можливі виключення в одному місці. А це означає, що вашу програму стає значно легше писати і читати, оскільки виконуване завдання не змішується з обробкою помилок. 

### Обробники виключень

Звичайно, порушене виключення в кінцевому підсумку повинно бути десь оброблено. Цим місцем є обробник виключень, який створюється для кожного виключення, яке ви хочете перехопити. Обробники виключень розміщуються прямо за блоком ***try*** і позначаються ключовим словом ***catch***:
 
``` java
try {  
  // Частина програми, здатна порушувати виключення  
} catch(Type1 id1) {  
  // Обробка виключення Type1
} catch(Type2 id2) {  
  // Обробка виключення Type2  
} catch(Type3 id3) {  
  // Обробка виключення Type3
}  
// і т.д....
```

Кожна пропозиція ***catch*** (обробник виключення) нагадує маленький метод, який приймає один і тільки один аргумент певного типу. Ідентифікатор (***id1, id2*** і т.д.) Може використовуватися всередині обробника точно так само, як і метод розпоряджається своїми аргументами. Іноді цей ідентифікатор залишається незатребуваним, так як тип винятку дає достатньо інформації для його обробки, але тим не менше присутній він завжди. 

Обробники завжди слідують прямо за блоком ***try***. При виникненні виключення механізм обробки виключень шукає перший з обробників виключень, аргумент якого відповідає поточному типу виключення. Після цього він передає управління в блок ***catch***, і таким чином виключення вважається обробленим. Після виконання пропозиції ***catch*** пошук обробників виключення припиняється. Виконується тільки одна секція ***catch***, що відповідає типу виключення; в цьому відношенні обробка виключень відрізняється від команди ***switch***, де потрібно дописувати ***break*** після кожного ***case***, щоб запобігти виконанню всіх інших ***case***. Зауважте також, що всередині блоку ***try*** можуть викликатися різні методи, здатні породити однакові типи виключення, але обробник знадобиться всього один.
 
### Переривання в порівнянні з відновленням
 
У теорії обробки виключень є дві основні моделі. Модель переривання (яке використовується в *Java* і *C++*) передбачає, що помилка настільки серйозна, що при виникненні виключення продовжити виконання неможливо. Хоч би хто порушив виключення, сам факт його видачі означає, що виправити ситуацію «на місці» неможливо і повертати управління назад не потрібно. 

Альтернативна модель називається відновленням. Вона має на увазі, що обробник помилок зробить щось для виправлення ситуації, після чого робиться спроба повторити невдалу операцію в надії на успішний результат. 

В такому випадку виключення більше нагадує виклик методу - щоб застосувати модель відновлення в *Java*, вам доведеться піти саме цим шляхом (тобто не порушувати виключення, а викликати метод, здатний вирішити проблему). Також можна створити блок ***try*** всередині циклу ***while***, який буде знову і знову звертатися до цього блоку, поки не буде досягнутий потрібний результат. 

Історично так склалося, що програмісти, які використовують операційні системи з підтримкою відновлення, з часом переходили до моделі переривання, забуваючи іншу модель. Хоча ідея відновлення виглядає привабливо, вона не настільки корисна на практиці. Основна причина криється в зворотньому зв'язку: обробник помилки часто повинен знати, де відбулося виключення і містити спеціальний код для кожного окремого місця помилки. А це ускладнює написання та підтримку програм, особливо для великих систем, де виключення можуть бути згенеровані в багатьох різних місцях.
 
## Створення власних виключень
 
Ваш вибір не обмежується використанням вже існуючих в *Java* виключень. Ієрархія виключень *JDK* не може передбачити всі можливі помилки, тому ви маєте право створювати власні типи виключень для позначення специфічних помилок вашої програми. 

Для створення власного класу винятку вам доведеться визначити його похідним від вже існуючого типу - бажано найбільш близького до вашої ситуації (хоч це і не завжди можливо). У найпростішому випадку створюється клас з конструктором за замовчуванням:
 
``` java
//: exceptions/InheritingExceptions.java 
// Створення власного виключення
 
class SimpleException extends Exception {} 
 
public class InheritingExceptions { 
  public void f() throws SimpleException { 
    System.out.println("Throw SimpleException from f()"); 
    throw new SimpleException(); 
  } 
  public static void main(String[] args) { 
    InheritingExceptions sed = new InheritingExceptions(); 
    try { 
      sed.f(); 
    } catch(SimpleException e) { 
      System.out.println("Caught it!"); 
    } 
  } 
} /* Output: 
Throw SimpleException from f() 
Caught it! 
*///:~ 
```
 
Компілятор створює конструктор за замовчуванням, який автоматично викликає конструктор базового класу. Звичайно, в цьому випадку ви втрачаєте конструктора виду ***SimpleException(String)***, але на практиці він не дуже часто використовується. Як ви ще побачите, найбільш важливо у виключенні саме ім'я класу, так що в основному виключень, схожих на створене вище, буде достатньо. 

У прикладі результати роботи виводяться на консоль. Втім, їх також можна направити в стандартний потік помилок, що досягається використанням класу ***System.err***. Зазвичай це правильніше, ніж виводити в потік ***System.out***, який може бути перенаправлений. При виведенні результатів за допомогою ***System.err*** користувач помітить їх швидше, ніж при виведенні в ***System.out***. Також можна створити клас виключення з конструктором, які отримують аргумент ***String***:
 
``` java
//: exceptions/FullConstructors.java 
 
class MyException extends Exception { 
  public MyException() {} 
  public MyException(String msg) { super(msg); } 
} 
 
public class FullConstructors { 
  public static void f() throws MyException { 
    System.out.println("Throwing MyException from f()"); 
    throw new MyException(); 
  } 
  public static void g() throws MyException { 
    System.out.println("Throwing MyException from g()"); 
    throw new MyException("Originated in g()"); 
  } 
  public static void main(String[] args) { 
    try { 
      f(); 
    } catch(MyException e) { 
      e.printStackTrace(System.out); 
    } 
    try { 
      g(); 
    } catch(MyException e) { 
      e.printStackTrace(System.out); 
    } 
  } 
} /* Output: 
Throwing MyException from f() 
MyException 
        at FullConstructors.f(FullConstructors.java:11) 
        at FullConstructors.main(FullConstructors.java:19) 
Throwing MyException from g() 
MyException: Originated in g() 
        at FullConstructors.g(FullConstructors.java:15) 
        at FullConstructors.main(FullConstructors.java:24) 
*///:~
```

Зміни незначні - з'явилося два конструктора, що визначають спосіб створення об'єкта ***MyException***. У другому конструкторі використовується конструктор батьківського класу з аргументом ***String***, що викликається ключовим словом ***super***. У обробнику виключень викликається метод ***printStackTrace()*** класу ***Throwable*** (базового для ***Exception***). Цей метод виводить інформацію про послідовність викликів, яка привела до точки виникнення виключення. У нашому прикладі інформація направляється в ***System.out***, але виклик за замовчуванням направляє інформацію в стандартний потік помилок:
 
``` java
e.printStackTrace();
```

## Реєстрація виключень

Допоміжний простір імен ***java.util.logging*** дозволяє зареєструвати інформацію про виключення в журналі. Базові засоби реєстрації досить прості:
 
``` java
//: exceptions/LoggingExceptions.java 
// Реєстрація виключень з використанням Logger
import java.util.logging.*; 
import java.io.*; 
 
class LoggingException extends Exception { 
  private static Logger logger = 
    Logger.getLogger("LoggingException"); 
  public LoggingException() { 
    StringWriter trace = new StringWriter(); 
    printStackTrace(new PrintWriter(trace)); 
    logger.severe(trace.toString()); 
  } 
} 
 
public class LoggingExceptions { 
  public static void main(String[] args) { 
    try { 
      throw new LoggingException(); 
    } catch(LoggingException e) { 
      System.err.println("Caught " + e); 
    } 
    try { 
      throw new LoggingException(); 
Error Handling with Exceptions  319  
    } catch(LoggingException e) { 
      System.err.println("Caught " + e); 
    } 
  } 
} /* Output: (85% match) 
Aug 30, 2005 4:02:31 PM LoggingException <init> 
SEVERE: LoggingException 
        at LoggingExceptions.main(LoggingExceptions.java:19) 
 
Caught LoggingException 
Aug 30, 2005 4:02:31 PM LoggingException <init> 
SEVERE: LoggingException 
        at LoggingExceptions.main(LoggingExceptions.java:24) 
 
Caught LoggingException 
*///:~ 
```
 
Статичний метод ***Logger.getLogger()*** створює об'єкт ***Logger***, асоційований з аргументом ***String*** (зазвичай ім'я пакета і класу, до якого відносяться помилки); об'єкт передає свій вивыд в ***System.err***. Найпростіший спосіб запису інформації в ***Logger*** полягає у виклику методу, що відповідає рівню помилки; в нашому прикладі використовується метод ***severe()***. Нам хотілося б створити String для реєстрованого повідомлення з результатів трасування стека, але метод ***printStackTrace()*** за замовчуванням не створює ***String***. Для отримання ***String*** необхідно використовувати перевантажену версію ***printStackTrace()*** з аргументом ***java.io.PrintWriter*** (за докладними поясненнями звертайтеся до глави «Ввід/вивід»). Якщо передати конструктору ***PrintWriter*** об'єкт ***java.io.StringWriter***, для отримання виводу у форматі ***String*** досить викликати ***toString()***.
 
Підхід ***LoggingException*** надзвичайно зручний (вся інфраструктура реєстрації вбудована в самt виключення, і все працює автоматично без втручання з боку клієнта), однак на практиці частіше застосовується перехоплення і реєстрація «сторонніх» виключень, тому повідомлення має генеруватися в обробнику виключення:
 
``` java
//: exceptions/LoggingExceptions2.java 
// Реєстрація перехоплених виключень 
import java.util.logging.*; 
import java.io.*; 
 
public class LoggingExceptions2 { 
  private static Logger logger = 
    Logger.getLogger("LoggingExceptions2"); 
  static void logException(Exception e) { 
    StringWriter trace = new StringWriter(); 
    e.printStackTrace(new PrintWriter(trace)); 
    logger.severe(trace.toString()); 
  } 
  public static void main(String[] args) { 
    try { 
      throw new NullPointerException(); 
    } catch(NullPointerException e) { 
      logException(e); 
    } 
  } 
} /* Output: (90% match) 
Aug 30, 2005 4:07:54 PM LoggingExceptions2 logException 
SEVERE: java.lang.NullPointerException 
        at LoggingExceptions2.main(LoggingExceptions2.java:16) 
*///:~ 
```
 
На цьому процес створення власних виключень не закінчується - виключення можна забезпечити додатковими конструкторами і елементами:
 
``` java
//: exceptions/ExtraFeatures.java 
// Подальше розширення класів виключень 
import static net.mindview.util.Print.*; 
 
class MyException2 extends Exception { 
  private int x; 
  public MyException2() {} 
  public MyException2(String msg) { super(msg); } 
  public MyException2(String msg, int x) { 
    super(msg); 
    this.x = x; 
  } 
  public int val() { return x; } 
  public String getMessage() { 
    return "Detail Message: "+ x + " "+ super.getMessage(); 
  } 
} 
 
public class ExtraFeatures { 
  public static void f() throws MyException2 { 
    print("Throwing MyException2 from f()"); 
    throw new MyException2(); 
  } 
  public static void g() throws MyException2 { 
    print("Throwing MyException2 from g()"); 
    throw new MyException2("Originated in g()"); 
  } 
  public static void h() throws MyException2 { 
    print("Throwing MyException2 from h()"); 
    throw new MyException2("Originated in h()", 47); 
  } 
  public static void main(String[] args) { 
    try { 
      f(); 
    } catch(MyException2 e) { 
      e.printStackTrace(System.out); 
    } 
    try { 
      g(); 
    } catch(MyException2 e) { 
      e.printStackTrace(System.out); 
    } 
    try { 
      h(); 
    } catch(MyException2 e) { 
      e.printStackTrace(System.out); 
      System.out.println("e.val() = " + e.val()); 
    } 
  } 
} /* Output: 
Throwing MyException2 from f() 
MyException2: Detail Message: 0 null 
        at ExtraFeatures.f(ExtraFeatures.java:22) 
        at ExtraFeatures.main(ExtraFeatures.java:34) 
Throwing MyException2 from g() 
MyException2: Detail Message: 0 Originated in g() 
        at ExtraFeatures.g(ExtraFeatures.java:26) 
        at ExtraFeatures.main(ExtraFeatures.java:39) 
Throwing MyException2 from h() 
MyException2: Detail Message: 47 Originated in h() 
        at ExtraFeatures.h(ExtraFeatures.java:30) 
        at ExtraFeatures.main(ExtraFeatures.java:44) 
e.val() = 47 
*///:~ 
```
 
Було додано поле даних ***х*** разом з методом, що зчитує його значення, а також додатковий конструктор для ініціалізації ***х***. Перевизначення метод ***Throwable.getMessage()*** виводить більш змістовну інформацію про виключення. Метод ***getMessage()*** для класів виключень - аналог ***toString()*** в звичайних класах. Так як виключення є просто видом об'єкта, розширення можливостей класів виключень можна продовжити. Однак слід пам'ятати, що всі ці програмісти, які використовують ваші бібліотеки, можуть просто проігнорувати всі «прикраси» - нерідко програмісти обмежуються перевіркою типу виключення (як найчастіше буває зі стандартними виключеннями *Java*).
 
## Специфікації виключень
 
У мові *Java* бажано повідомляти програмісту, що викликає ваш метод, про виключення, які даний метод здатний порушувати. Користувач, що викликає метод, зможе написати весь необхідний код для перехоплення можливих виключень. Звичайно, коли доступний початковий код, программіст-клієнт може перегорнути його в пошуку інструкцій ***throw***, але бібліотеки не завжди поставляються з вихідними текстами. Щоб бібліотека не перетворювалася в «чорний ящик», в *Java* додали синтаксис (обов'язковий для використання), за допомогою якого ви повідомляєте клієнту про виключення, що генеруються методом, щоб він зумів правильно обробити їх. Цей синтаксис називається специфікацією виключень (*exception specification*), входить в оголошення методу і слідує відразу за списком аргументів.
 
Специфікація виключень складається з ключового слова ***throws***, за яким перераховуються всі можливі типи виключень. Зразкове визначення методу буде виглядати так:

``` java
void f() throws TooBig, TooSmall, DivZero { //... 
}
```
 
Однак запис

``` java
void f() { // ...
}
```
 
означає, що метод не генерує виключень. (Крім виключень, похідних від ***RuntimeException***, які можуть бути порушені практично в будь-якому місці - про це ще буде сказано.)

Обійти специфікацію виключень неможливо - якщо ваш метод генерує виключення і не обробляє їх, компілятор запропонує, або обробити виключення, або включити його в специфікацію. Жорстким контролем за дотриманням правил від верху до низу *Java* гарантує правильність використання механізму виключень під час компіляції програми.
 
Втім, «обдурити» компілятор все ж можна: ви маєте право оголосити про генерацію виключення, якого насправді немає. Компілятор вірить вам на слово і змушує користувачів методу діяти так, як ніби їм і справді необхідно перехоплювати виключення. Таким чином можна «зарезервувати» виключення на майбутнє і вже потім генерувати його, не змінюючи опису готової програми. Така можливість може стати в нагоді і для створення абстрактних базових класів і інтерфейсів, в похідних класах яких може виникнути необхідність в генерації виключень. Виключення, які перевіряються і нав'язуються ще на етапі компіляції програми, називають контрольованими (*checked*). 

## Перехоплення довільних виключень

Можна створити універсальний обробник, що перехоплює будь-які типи виключення. Здійснюється це перехопленням базового класу всіх виключень ***Exception*** (існують і інші базові типи виключень, але клас ***Exception*** є базовим практично для всіх програмних виключних ситуацій):
 
``` java
catch(Exception е) {
	System.out println( "перехоплено виключення");
}
```
 
Подібна конструкція не пропустить жодного виключення, тому її слід розміщувати в самому кінці списку обробників, щоб уникнути блокування наступних за нею обробників виключень. Оскільки клас ***Exception*** є базовим для всіх класів виключень, цікавих програмісту, сам він не надасть ніякої корисної інформації про виключну ситуацію, але можна викликати методи з його базового типу ***Throwable***:

- ***String getMessage(), String getLocalizedMessage()*** повертають докладне повідомлення про помилку (або повідомлення, локалізоване для поточного контексту);
- ***String toString()*** повертає короткий опис об'єкта ***Throwable***, включаючи докладне повідомлення, якщо воно присутнє;
- ***void printStackTrace(), void printStackTrace(PrintStream), void printStackTrace(java.io.PrintWriter)*** виводять інформацію про об'єкт ***Throwable*** і трасування стека викликів для цього об'єкта. Трасування стека викликів показує послідовність виклику методів, яке приводить до точки виникнення виключення. Перший варіант відправляє інформацію в стандартний потік помилок, другий і третій - в потік за вашим вибором (в розділі «Ввід/вивід» ви зрозумієте, чому типів потоків два);
- ***Throwable fillInStackTrace()*** записує в об'єкт ***Throwable*** інформацію про поточний стан стека. Метод використовується при повторному порушенні помилок або виключень.

До того ж у вашому розпорядженні знаходяться методи типу ***Object***, базового для ***Throwable*** (і для всіх інших класів). При використанні виключень може стати в нагоді метод ***getClass()***, який повертає інформацію про клас об'єкту. Ця інформація міститься в об'єкті типу ***Class***. Наприклад, ви можете дізнатися ім'я класу разом з інформацією про пакет методом ***getName()***, або отримати тільки ім'я класу методом ***getSimpleName()***.
 
Розглянемо приклад з використанням основних методів класу ***Exception***:
 
``` java
//: exceptions/ExceptionMethods.java 
// Демонстрація методів класу Exception
import static net.mindview.util.Print.*; 
 
Error Handling with Exceptions  323  
public class ExceptionMethods { 
  public static void main(String[] args) { 
    try { 
      throw new Exception("My Exception"); 
    } catch(Exception e) { 
      print("Caught Exception"); 
      print("getMessage():" + e.getMessage()); 
      print("getLocalizedMessage():" + 
        e.getLocalizedMessage()); 
      print("toString():" + e); 
      print("printStackTrace():"); 
      e.printStackTrace(System.out); 
    } 
  } 
} /* Output: 
Caught Exception 
getMessage():My Exception 
getLocalizedMessage():My Exception 
toString():java.lang.Exception: My Exception 
printStackTrace(): 
java.lang.Exception: My Exception 
        at ExceptionMethods.main(ExceptionMethods.java:8) 
*///:~ 
```

Як бачите, методи послідовно розширюють обсяг повертаємої інформації - всякий наступний фактично є продовженням попереднього. 

### Трасування стека

Інформацію, надану методом ***printStackTrace()***, також можна отримати безпосередньо викликом ***getStackTrace()***. Метод повертає масив елементів трасування, кожен з яких представляє один кадр стека. Нульовий елемент представляє вершину стека, тобто останній викликаний метод послідовності (точка, в якій був створений і ініційований об'єкт ***Throwable***).
 
Відповідно, останній елемент масиву представляє «низ» стека, тобто перший викликаний елемент послідовності. Розглянемо простий приклад:
 
``` java
//: exceptions/WhoCalled.java 
// Програмний доступ до даних трасування стека
 
public class WhoCalled { 
  static void f() { 
    // Generate an exception to fill in the stack trace 
    try { 
      throw new Exception(); 
    } catch (Exception e) { 
      for(StackTraceElement ste : e.getStackTrace()) 
        System.out.println(ste.getMethodName()); 
    } 
  } 
  static void g() { f(); } 
  static void h() { g(); } 
  public static void main(String[] args) { 
    f(); 
    System.out.println("--------------------------------"); 
    g(); 
    System.out.println("--------------------------------"); 
    h(); 
  } 
} /* Output: 
f 
main 
-------------------------------- 
f 
g 
main 
-------------------------------- 
f 
g 
h 
main 
*///:~
```
 
### Повторне порушення виключення
 
У деяких ситуаціях потрібно заново порушити вже перехоплене виключення; найчастіше це відбувається при використанні ***Exception***
для перехоплення всіх виключень. Так як посилання на поточне виключення вже є, ви просто генеруєте виключення за цим посиланням:
 
``` java
catch(Exception е) {
	System.out.println( "повторно порушене виключення"); 
	throw e;
}
```
 
При повторному порушенні виключення передається в розпорядження обробника вищого рівня. Всі інші пропозиції ***catch***
поточного блоку ***try*** ігноруються. Вся інформація з об'єкта, що надає виключення, зберігається, і обробник вищого рівня, що перехоплює подібні виключення, зможе її отримати.
 
Якщо ви просто заново генеруєте виключення, інформація про нього, що виводиться методом ***printStackTrace()***, буде як і раніше відноситись до місця виникнення виключення, але не до місця його повторного порушення. Якщо вам знадобиться використовувати нове трасування стека, викличте метод ***fillInStaсkTrасe()***, який повертає виключення (об'єкт ***Throwable***), створене на базі старого з поміщенням туди поточної інформації про стек. Ось як це виглядає:
 
``` java
//: exceptions/Rethrowing.java 
// Демонстрація методу fillInStackTrace()
 
public class Rethrowing { 
  public static void f() throws Exception { 
    System.out.println("originating the exception in f()"); 
    throw new Exception("thrown from f()"); 
  } 
  public static void g() throws Exception { 
    try { 
      f(); 
    } catch(Exception e) { 
      System.out.println("Inside g(),e.printStackTrace()"); 
      e.printStackTrace(System.out); 
      throw e; 
    } 
  } 
  public static void h() throws Exception { 
Error Handling with Exceptions  325  
    try { 
      f(); 
    } catch(Exception e) { 
      System.out.println("Inside h(),e.printStackTrace()"); 
      e.printStackTrace(System.out); 
      throw (Exception)e.fillInStackTrace(); 
    } 
  } 
  public static void main(String[] args) { 
    try { 
      g(); 
    } catch(Exception e) { 
      System.out.println("main: printStackTrace()"); 
      e.printStackTrace(System.out); 
    } 
    try { 
      h(); 
    } catch(Exception e) { 
      System.out.println("main: printStackTrace()"); 
      e.printStackTrace(System.out); 
    } 
  } 
} /* Output: 
originating the exception in f() 
Inside g(),e.printStackTrace() 
java.lang.Exception: thrown from f() 
        at Rethrowing.f(Rethrowing.java:7) 
        at Rethrowing.g(Rethrowing.java:11) 
        at Rethrowing.main(Rethrowing.java:29) 
main: printStackTrace() 
java.lang.Exception: thrown from f() 
        at Rethrowing.f(Rethrowing.java:7) 
        at Rethrowing.g(Rethrowing.java:11) 
        at Rethrowing.main(Rethrowing.java:29) 
originating the exception in f() 
Inside h(),e.printStackTrace() 
java.lang.Exception: thrown from f() 
        at Rethrowing.f(Rethrowing.java:7) 
        at Rethrowing.h(Rethrowing.java:20) 
        at Rethrowing.main(Rethrowing.java:35) 
main: printStackTrace() 
java.lang.Exception: thrown from f() 
        at Rethrowing.h(Rethrowing.java:24) 
        at Rethrowing.main(Rethrowing.java:35) 
*///:~ 
```

Рядок з викликом ***fillInStackTrace()*** стає новою точкою генерації виключення. 

Також можна згенерувати виключення що може відрізнятися від перехопленного. У цьому випадку ефект виходить приблизно таким же, як при використанні ***fillInStackTrace()*** - інформація про місце зародження виключення втрачається, а залишається інформація, що відноситься до нової команди ***throw***.
 
``` java
//: exceptions/RethrowNew.java 
// Генерація виключення, що
// відрізняється від перехопленного

class OneException extends Exception { 
  public OneException(String s) { super(s); } 
} 
 
class TwoException extends Exception { 
326  Thinking in Java  Bruce Eckel  
  public TwoException(String s) { super(s); } 
} 
 
public class RethrowNew { 
  public static void f() throws OneException { 
    System.out.println("originating the exception in f()"); 
    throw new OneException("thrown from f()"); 
  } 
  public static void main(String[] args) { 
    try { 
      try { 
        f(); 
      } catch(OneException e) { 
        System.out.println( 
          "Caught in inner try, e.printStackTrace()"); 
        e.printStackTrace(System.out); 
        throw new TwoException("from inner try"); 
      } 
    } catch(TwoException e) { 
      System.out.println( 
        "Caught in outer try, e.printStackTrace()"); 
      e.printStackTrace(System.out); 
    } 
  } 
} /* Output: 
originating the exception in f() 
Caught in inner try, e.printStackTrace() 
OneException: thrown from f() 
        at RethrowNew.f(RethrowNew.java:15) 
        at RethrowNew.main(RethrowNew.java:20) 
Caught in outer try, e.printStackTrace() 
TwoException: from inner try 
        at RethrowNew.main(RethrowNew.java:25) 
*///:~ 
```
 
Про останнє виключення відомо тільки те, що воно надійшло з внутрішнього блоку ***try***, але не з методу ***f()***. Вам ніколи не доведеться піклуватися про видалення попередніх виключень, і виключень взагалі. Всі вони є об'єктами, створеними в загальній купі оператором new, і збирач сміття знищує їх автоматично.
 
### Ланцюжки виключень
 
Найчастіше необхідно перехопити одне виключення і порушити наступне, не втративши при цьому інформації про перше виключення - це називається ланцюжком виключень (*exception chaining*). До випуску пакета *JDK* 1.4 програмістам доводилося самостійно писати код, який зберігає інформацію про попередній виключення, проте тепер конструкторам всіх підкласів ***Throwable*** може передаватися об'єкт-причина (*cause*). Передбачається, що причиною є початкове виключення і передача його в новий об'єкт забезпечує трасування стека аж до самого його початку, хоча при цьому створюється і збуджується нове виключення. Цікаво відзначити, що єдиними підкласами класу ***Throwable***, які беруть об'єкт-причину як аргумент конструктора, є три основних класи виключень: ***Error*** (використовується віртуальною машиною (*JVM*) для повідомлень про системні помилки), ***Exception*** і ***RuntimeException***. Для організації ланцюжків з інших типів виключень доведеться використовувати метод ***initCause()***, а не конструктор. Наступний приклад демонструє динамічне додавання полів в об'єкт ***DynamicFields*** під час роботи програми:
 
``` java
//: exceptions/DynamicFields.java 
// Динамічне додавання полів в клас. 
// Приклад використання ланцюжка виключень 
import static net.mindview.util.Print.*; 
 
class DynamicFieldsException extends Exception {} 
 
public class DynamicFields { 
  private Object[][] fields; 
  public DynamicFields(int initialSize) { 
    fields = new Object[initialSize][2]; 
    for(int i = 0; i < initialSize; i++) 
      fields[i] = new Object[] { null, null }; 
  } 
  public String toString() { 
    StringBuilder result = new StringBuilder(); 
    for(Object[] obj : fields) { 
      result.append(obj[0]); 
      result.append(": "); 
      result.append(obj[1]); 
      result.append("\n"); 
    } 
    return result.toString(); 
  } 
  private int hasField(String id) { 
    for(int i = 0; i < fields.length; i++) 
      if(id.equals(fields[i][0])) 
        return i; 
    return -1; 
  } 
  private int 
  getFieldNumber(String id) throws NoSuchFieldException { 
    int fieldNum = hasField(id); 
    if(fieldNum == -1) 
      throw new NoSuchFieldException(); 
    return fieldNum; 
  } 
  private int makeField(String id) { 
    for(int i = 0; i < fields.length; i++) 
      if(fields[i][0] == null) { 
        fields[i][0] = id; 
        return i; 
      } 
    // No empty fields. Add one: 
    Object[][] tmp = new Object[fields.length + 1][2]; 
    for(int i = 0; i < fields.length; i++) 
      tmp[i] = fields[i]; 
    for(int i = fields.length; i < tmp.length; i++) 
      tmp[i] = new Object[] { null, null }; 
    fields = tmp; 
    // Recursive call with expanded fields: 
    return makeField(id); 
  } 
  public Object 
  getField(String id) throws NoSuchFieldException { 
    return fields[getFieldNumber(id)][1]; 
  } 
  public Object setField(String id, Object value) 
  throws DynamicFieldsException { 
    if(value == null) { 
      // Most exceptions don’t have a "cause" constructor. 
      // In these cases you must use initCause(), 
      // available in all Throwable subclasses. 
      DynamicFieldsException dfe = 
        new DynamicFieldsException(); 
      dfe.initCause(new NullPointerException()); 
      throw dfe; 
    } 
    int fieldNumber = hasField(id); 
    if(fieldNumber == -1) 
      fieldNumber = makeField(id); 
    Object result = null; 
    try { 
      result = getField(id); // Get old value 
    } catch(NoSuchFieldException e) { 
      // Use constructor that takes "cause": 
      throw new RuntimeException(e); 
    } 
    fields[fieldNumber][1] = value; 
    return result; 
  } 
  public static void main(String[] args) { 
    DynamicFields df = new DynamicFields(3); 
    print(df); 
    try { 
      df.setField("d", "A value for d"); 
      df.setField("number", 47); 
      df.setField("number2", 48); 
      print(df); 
      df.setField("d", "A new value for d"); 
      df.setField("number3", 11); 
      print("df: " + df); 
      print("df.getField(\"d\") : " + df.getField("d")); 
      Object field = df.setField("d", null); // Exception 
    } catch(NoSuchFieldException e) { 
      e.printStackTrace(System.out); 
    } catch(DynamicFieldsException e) { 
      e.printStackTrace(System.out); 
    } 
  } 
} /* Output: 
null: null 
null: null 
null: null 
 
d: A value for d 
number: 47 
number2: 48 
 
df: d: A new value for d 
number: 47 
number2: 48 
number3: 11 
 
df.getField("d") : A new value for d 
DynamicFieldsException 
        at DynamicFields.setField(DynamicFields.java:64) 
        at DynamicFields.main(DynamicFields.java:94) 
Caused by: java.lang.NullPointerException 
        at DynamicFields.setField(DynamicFields.java:66) 
        ... 1 more 
*///:~ 
```
 
Кожен об'єкт ***DynamicFields*** містить масив пар ***Object-Object***. Перший об'єкт містить ідентифікатор поля (***String***), а другий об'єкт - значення поля, яке може бути будь-якого типу, крім упакованих примітивів. При створенні об'єкта необхідно оцінити приблизну кількість полів. Метод ***setField()***, або знаходить вже існуюче поле з заданим ім'ям, або створює нове поле і зберігає значення. Коли простір для полів закінчується, метод нарощує його, створюючи масив розміром на одиницю більше і копіюючи в нього старі елементи. При спробі розміщення порожнього посилання ***null*** метод ініціює виключення ***DynamicFieldsException***, створюючи об'єкт потрібного типу і передаючи методу ***initCause()*** в якості причини виключення ***NullPointerException*** . Для значення, що повертається метод ***setField()*** використовує старе значення поля, отримуючи його методом ***getField()***, яке може згенерувати виключення ***NoSuchFieldException***. Якщо метод ***getField()*** викликає програміст-клієнт, то він відповідальний за обробку можливого виключення ***NoSuchFieldException***, однак, якщо останнє виникає в методі ***setField()***, це є помилкою програми;
відповідно, отримане виключення перетворюється в виключення ***RuntimeException*** на конструкторі, що приймає аргумент-причину.
 
Для створення результату ***toString()*** використовує об'єкт ***StringBuilder***. Цей клас буде детально розглянуто при описі роботи з рядками.
 
## Стандартні виключення *Java*
 
Клас *Java* ***Throwable*** описує всі об'єкти, які можуть порушуватися як виключення. Існує два основні різновиди об'єктів ***Throwable*** (тобто гілки успадкування). Тип ***Error*** представляє системні помилки і помилки часу компіляції, які зазвичай не перехоплюються (крім кількох особливих випадків). Тип ***Exception*** може бути згенерований з будь-якого методу стандартної бібліотеки класів *Java*, або призначеного для користувача методу в разі неполадок при виконанні програми. Таким чином, для програмістів інтерес представляє перш за все тип ***Exception***. 

Кращий спосіб отримати уявлення про виключення - переглянути документацію *JDK*. Варто зробити це хоча б раз, щоб отримати уявлення про різні види виключень, але незабаром ви переконаєтеся в тому, що найбільш важливою відмінністю між різними виключення є їх імена. До того ж кількість виключень у *Java* постійно зростає, і навряд чи має сенс описувати їх в книзі. Будь-яка програмна бібліотека від стороннього виробника, швидше за все, також буде мати власний набір виключень. Тут важливіше зрозуміти принцип роботи і робити з виключеннями відповідно.

Основний принцип полягає в тому, що ім'я виключення повністю пояснює суть виниклої проблеми. Не всі виключення визначені в пакеті ***java.lang***, деякі з них створені для підтримки інших бібліотек, таких як ***util, net*** і ***io***, як можна бачити з повних імен їх класів, або з базових класів. Наприклад, всі виключення, пов'язані з вводом/виводом (*I/O*), успадковані від ***java.io.IOException***.
 
### Особливий випадок: ***RuntimeException***

Згадаймо перший приклад в цьому розділі:
 
``` java
if(t == null) 
	throw new NullPointerException();
```
 
Тільки уявіть, як жахливо було б перевіряти таким чином кожне посилання, передане вашому методу. На щастя, робити це не потрібно - така перевірка автоматично виконується під час виконання *Java*-програми, і при спробі використання ***null*** посилань автоматично генерується ***NullPointerException***. Таким чином, використана в прикладі конструкція надлишкова.
 
Є ціла група виключень, що належать до цієї категорії. Вони завжди генеруються в *Java* автоматично, і вам не доведеться включати їх в специфікацію виключень. Всі вони успадковані від одного базового класу ***RuntimeException***, що дає нам ідеальний приклад наслідування:
сімейство класів, що мають спільні характеристики і поведінку. Вам також не доведеться створювати специфікацію виключень, що вказує на порушення методом ***RuntimeException*** (або будь-якого успадкованого від нього виключення), так як ці виключення відносяться до неконтрольованих (*unchecked*). Такі виключення означають помилки в програмі, і фактично вам ніколи не доведеться перехоплювати їх - це робиться автоматично. Перевірка ***RuntimeException*** привела б до зайвого захаращення програми. І хоча вам звичайно не потрібно перехоплювати ***RuntimeException***, можливо, ви вважаєте за потрібне порушувати деякі з них в своїх власних бібліотеках програм.
 
Що ж відбувається, коли подібні виключення, не перехоплюються? Так як компілятор не змушує перераховувати такі виключення в специфікаціях, можна припустити, що виключення ***RuntimeException*** проникне прямо в метод ***main()***, і не буде перехоплено. Щоб побачити все в дії, випробуйте наступний приклад:
 
``` java
//: exceptions/NeverCaught.java 
// Ігнорування RuntimeExceptions. 
// {ThrowsException} 
 
public class NeverCaught { 
  static void f() { 
    throw new RuntimeException("From f()"); 
  } 
  static void g() { 
    f(); 
  } 
  public static void main(String[] args) { 
    g(); 
  } 
} ///:~ 
```
 
Можна відразу зауважити, що ***RuntimeException*** (і все від нього успадковане) є спеціальним випадком, так як компілятор не вимагає для нього специфікації виключення. Вихідні дані виводяться в ***System.err***:

```
Exception in thread "main" Java.lang.RuntimeException: From f() 
        at NeverCaught.f(NeverCaught.Java:7) 
        at NeverCaught.g(NeverCaught.Java:10) 
        at NeverCaught.main(NeverCaught.Java:13) 
```

Ми підійшли до відповіді на поставлене запитання: якщо ***RuntimeException*** добирається до ***main()*** без перехоплення, то робота програми завершується з викликом методу ***printStackTrace()***.
 
Пам'ятайте, що тільки виключення типу ***RuntimeException*** (і похідних класів) можуть ігноруватися під час написання тексту програми, в той час як інші дії компілятор здійснює в обов'язковому порядку. Це пояснюється тим, що ***RuntimeException*** є наслідком помилки програміста, наприклад:

1. Помилка, яку неможливо передбачити (наприклад, отримання ***null*** - посилання в вашому методі, переданої зовні);
2. Помилка, яку ви як програміст повинні були перевірити у вашій програмі (такій як ***ArraylndexOutOfBoundsException***, з перевіркою розміру масиву). Помилки першого виду часто стають причиною помилок другого виду.

У подібних ситуаціях виключення надають неоціненну допомогу в відладочному процесі. Назвати механізм виключень *Java* вузькоспеціалізованим інструментом було б невірно. Так, він допомагає впоратися з прикрими помилками на стадії виконання програми, які неможливо передбачити заздалегідь, але при цьому даний механізм допомагає виявляти багато помилок програмування, що виходять за межі можливостей компілятора. 

## Завершення за допомогою finally

Часто зустрічається ситуація, коли деяка частина програми повинна виконуватися незалежно від того, було чи ні згенеровано виключення  всередині блоку try. Зазвичай це має відношення до операцій, не пов'язаних із звільненням пам'яті (так як це входить в обов'язки збирача сміття). Для досягнення бажаної мети необхідно розмістити блок ***finally*** після всіх обробників виключень. Таким чином, повна конструкція обробки виключення виглядає так:
 
``` java
try {
	// Захищена секція: ризиковані операції,
	// Які можуть породити виключення А, В. або C
} Catch(A al) {
	// Оброблювач для ситуації А
} Catch(B bl) {
	// Оброблювач для ситуації В
} Catch(C cl) {
	// Оброблювач для ситуації C
} Finally {
	// Дії, вироблені в будь-якому випадку
}
```
 
Щоб продемонструвати, що блок ***finally*** виконується завжди, розглянемо наступну програму:
 
``` java
//: Exceptions/FinallyWorks.java
// Блок finally виконується завжди 

class ThreeException extends Exception {} 
 
public class FinallyWorks { 
  static int count = 0; 
  public static void main(String[] args) { 
    while(true) { 
      try { 
        // Post-increment is zero first time: 
        if(count++ == 0) 
          throw new ThreeException(); 
        System.out.println("No exception"); 
      } catch(ThreeException e) { 
        System.out.println("ThreeException"); 
      } finally { 
        System.out.println("In finally clause"); 
        if(count == 2) break; // out of "while" 
      } 
    } 
  } 
} /* Output: 
ThreeException 
In finally clause 
No exception 
In finally clause 
*///:~       	
```
 
Результат роботи програми показує, що незалежно від того, чи було згенеровано виключення, пропозиція ***finally*** виконується завжди. 

Даний приклад також підказує, як впоратися з тим фактом, що *Java* не дозволяє повернутися до місця виникнення виключення, про що говорилося раніше. Якщо розташувати блок ***try*** в циклі, можна також визначити умову, на підставі якої буде вирішено, чи повинна програма продовжуватиметься. Також можна додати статичний лічильник, або інший механізм для перевірки декількох різних рішень, перш ніж відмовитися від спроб відновлення. Це один із способів забезпечення підвищеної відмовостійкості програм.
 
### Для чого потрібен блок finally?
 
У мовах без збору сміття і без автоматичного виклику деструкторів блок ***finally*** гарантує звільнення ресурсів і пам'яті незалежно від того, що сталося в блоці ***try***. В *Java* існує збирач сміття, тому після визволення пам'яті проблем не буває. Також немає необхідності викликати деструктори, їх просто немає. Коли ж потрібно використовувати ***finally*** в *Java*?

Блок ***finally*** необхідний тоді, коли в початковий стан вам необхідно повернути щось інше, ніж пам'ять. Це може бути, наприклад, відкритий файл, або підключення до мережі, частина зображення на екрані, або навіть якийсь фізичний перемикач, на кшталт змодельованого в наступному прикладі:
 
``` java
//: exceptions/Switch.java 
import static net.mindview.util.Print.*; 
 
public class Switch { 
  private boolean state = false; 
  public boolean read() { return state; } 
  public void on() { state = true; print(this); } 
  public void off() { state = false; print(this); } 
  public String toString() { return state ? "on" : "off"; } 
} ///:~ 
 
//: exceptions/OnOffException1.java 
public class OnOffException1 extends Exception {} ///:~ 

//: exceptions/OnOffException2.java 
public class OnOffException2 extends Exception {} ///:~ 
 
//: exceptions/OnOffSwitch.java 
// Навіщо використовувати finally?
 
public class OnOffSwitch { 
  private static Switch sw = new Switch(); 
  public static void f() 
  throws OnOffException1,OnOffException2 {} 
  public static void main(String[] args) { 
    try { 
      sw.on(); 
      // Code that can throw exceptions... 
      f(); 
      sw.off(); 
    } catch(OnOffException1 e) { 
      System.out.println("OnOffException1"); 
      sw.off(); 
    } catch(OnOffException2 e) { 
      System.out.println("OnOffException2"); 
      sw.off(); 
    } 
  } 
} /* Output: 
on 
off 
*///:~ 
```
 
Наша мета - переконатися в тому, що перемикач був вимкнений по завершенні методу ***main()***, тому в кінці блоку ***try*** і в кінці кожного обробника виключення поміщається виклик ***sw.off()***. Однак у програмі може виникнути неперехвачене виключення, і тоді виклик ***sw.off()*** буде пропущений. Однак завдяки ***finally*** завершальний код можна помістити в одному певному місці:
 
``` java
//: exceptions/WithFinally.java 
// Finally гарантує виконання завершального коду. 
 
public class WithFinally { 
  static Switch sw = new Switch(); 
  public static void main(String[] args) { 
    try { 
      sw.on(); 
      // Code that can throw exceptions... 
      OnOffSwitch.f(); 
    } catch(OnOffException1 e) { 
      System.out.println("OnOffException1"); 
    } catch(OnOffException2 e) { 
      System.out.println("OnOffException2"); 
    } finally { 
      sw.off(); 
    } 
  } 
} /* Output: 
on 
off 
*///:~
```
 
Тут виклик методу ***sw.off()*** просто переміщений в те місце, де він гарантовано буде виконаний.
 
Навіть якщо виключення не перехоплюється в поточному наборі умов ***catch***, блок ***finally*** відпрацює перед тим, як механізм обробки виключень продовжить пошук обробника на більш високому рівні:
 
``` java
//: exceptions/AlwaysFinally.java 
// Finally is always executed. 
import static net.mindview.util.Print.*; 
 
class FourException extends Exception {} 
 
public class AlwaysFinally { 
  public static void main(String[] args) { 
    print("Entering first try block"); 
    try { 
      print("Entering second try block"); 
      try { 
        throw new FourException(); 
      } finally { 
        print("finally in 2nd try block"); 
      } 
    } catch(FourException e) { 
      System.out.println( 
        "Caught FourException in 1st try block"); 
    } finally { 
      System.out.println("finally in 1st try block"); 
    } 
  } 
} /* Output: 
Entering first try block 
Entering second try block 
finally in 2nd try block 
Caught FourException in 1st try block 
finally in 1st try block 
*///:~ 
```
 
Блок ***finally*** також виконується при використанні команд ***break*** і ***continue***. Зауважте, що комбінація ***finally*** в поєднанні з ***break*** і ***continue*** з мітками знімає в *Java* всяку необхідність в операторі ***goto***.
 
### Використання finally з return
 
Оскільки секція ***finally*** виконується завжди, важливі завершальні дії будуть виконуватись навіть при поверненні з декількох точок методу:
 
``` java
//: exceptions/MultipleReturns.java 
import static net.mindview.util.Print.*; 
 
public class MultipleReturns { 
  public static void f(int i) { 
    print("Initialization that requires cleanup"); 
    try { 
      print("Point 1"); 
      if(i == 1) return; 
      print("Point 2"); 
      if(i == 2) return; 
      print("Point 3"); 
      if(i == 3) return; 
      print("End"); 
      return; 
    } finally { 
      print("Performing cleanup"); 
    } 
  } 
  public static void main(String[] args) { 
    for(int i = 1; i <= 4; i++) 
      f(i); 
  } 
} /* Output: 
Initialization that requires cleanup 
Point 1 
Performing cleanup 
Initialization that requires cleanup 
Point 1 
Point 2 
Performing cleanup 
Initialization that requires cleanup 
Point 1 
Point 2 
Point 3 
Performing cleanup 
Initialization that requires cleanup 
Point 1 
Point 2 
Point 3 
End 
Performing cleanup 
*///:~ 
```
 
З вихідних даних видно, що виконання ***finally*** не залежить від того, в якій точці захищеної секції була виконана команда ***return***. 

### Проблема втрачених виключень

На жаль, реалізація механізму виключень в *Java* не обійшлася без вад. Хоча виключення сигналізує про аварійну ситуацію в програмі і ніколи не повинно ігноруватися, воно може бути втрачено. Це відбувається при використанні ***finally*** в конструкції певного виду:
 
``` java
//: exceptions/LostMessage.java 
// Як губляться виключення. 
 
class VeryImportantException extends Exception { 
  public String toString() { 
    return "A very important exception!"; 
  } 
} 
 
class HoHumException extends Exception { 
  public String toString() { 
    return "A trivial exception"; 
  } 
} 
 
public class LostMessage { 
  void f() throws VeryImportantException { 
    throw new VeryImportantException(); 
  } 
  void dispose() throws HoHumException { 
    throw new HoHumException(); 
  } 
  public static void main(String[] args) { 
    try { 
      LostMessage lm = new LostMessage(); 
      try { 
        lm.f(); 
      } finally { 
        lm.dispose(); 
      } 
    } catch(Exception e) { 
      System.out.println(e); 
    } 
  } 
} /* Output: 
A trivial exception 
*///:~ 
```
 
У виводі немає ніяких ознак ***VeryImportantException***, воно було просто заміщено виключенням ***HoHumException*** в розділі ***finally***. Це дуже серйозний недолік, так як втрата виключення може статися в набагато більш прихованій і важко діагностуємій ситуації, на відміну від тієї, що показана в прикладі. Наприклад, в *C++* подібна ситуація (порушення другого виключення без обробки першого) розглядається як груба помилка програміста. Можливо, в нових версіях *Java* ця проблема буде вирішена (втім, будь-який метод, здатний порушувати виключення - такий, як ***dispose()*** в наведеному прикладі - зазвичай полягає в конструкції ***try-catch***).
 
Ще простіше втратити виключення простим поверненням з finally:
 
``` java
//: exceptions/ExceptionSilencer.java 
 
public class ExceptionSilencer { 
  public static void main(String[] args) { 
    try { 
      throw new RuntimeException(); 
    } finally { 
      // Команда 'return' в блоці finally 
      // перериває обробку виключення return
      return; 
    } 
  } 
} ///:~ 
```
 
Запустивши цю програму, ви побачите, що вона нічого не виводить - незважаючи на виключення.
 
## Обмеження при використанні виключень
 
У перевизначенних методах можна порушувати лише ті виключення, які були описані в методі базового класу. Це корисне обмеження означає, що програма, яка працює з базовим класом, автоматично зможе працювати і з об'єктом, що походить від базового (звичайно, це фундаментальний принцип ООП), включаючи і виключення. Наступний приклад демонструє види обмежень (під час компіляції), накладені на виключення:
 
``` java
//: exceptions/StormyInning.java 
// Перевизначенні методи можуть порушувати тільки
// ті виключення, які описані в базовому класі,
// або виключення, успадковані від виключень
// базового класу.
 
class BaseballException extends Exception {} 
class Foul extends BaseballException {} 
class Strike extends BaseballException {} 
 
abstract class Inning { 
  public Inning() throws BaseballException {} 
  public void event() throws BaseballException { 
    // Doesn’t actually have to throw anything 
  } 
  public abstract void atBat() throws Strike, Foul; 
  public void walk() {} // Throws no checked exceptions 
} 
 
class StormException extends Exception {} 
class RainedOut extends StormException {} 
class PopFoul extends Foul {} 
 
interface Storm { 
  public void event() throws RainedOut; 
  public void rainHard() throws RainedOut; 
} 
 
public class StormyInning extends Inning implements Storm { 
  // OK to add new exceptions for constructors, but you 
  // must deal with the base constructor exceptions: 
  public StormyInning() 
    throws RainedOut, BaseballException {} 
  public StormyInning(String s) 
    throws Foul, BaseballException {} 
  // Regular methods must conform to base class: 
//! void walk() throws PopFoul {} //Compile error 
  // Interface CANNOT add exceptions to existing 
  // methods from the base class: 
//! public void event() throws RainedOut {} 
  // If the method doesn’t already exist in the 
  // base class, the exception is OK: 
  public void rainHard() throws RainedOut {} 
  // You can choose to not throw any exceptions, 
    // even if the base version does: 
  public void event() {} 
  // Overridden methods can throw inherited exceptions: 
  public void atBat() throws PopFoul {} 
  public static void main(String[] args) { 
    try { 
      StormyInning si = new StormyInning(); 
      si.atBat(); 
    } catch(PopFoul e) { 
      System.out.println("Pop foul"); 
    } catch(RainedOut e) { 
      System.out.println("Rained out"); 
    } catch(BaseballException e) { 
      System.out.println("Generic baseball exception"); 
    } 
    // Strike not thrown in derived version. 
    try { 
      // What happens if you upcast? 
      Inning i = new StormyInning(); 
      i.atBat(); 
      // You must catch the exceptions from the 
      // base-class version of the method: 
    } catch(Strike e) { 
      System.out.println("Strike"); 
    } catch(Foul e) { 
      System.out.println("Foul"); 
    } catch(RainedOut e) { 
      System.out.println("Rained out"); 
    } catch(BaseballException e) { 
      System.out.println("Generic baseball exception"); 
    } 
  } 
} ///:~ 
```

У класі ***Inning*** і конструктор, і метод ***event()*** оголошують, що порушуватимуть виключення, але в дійсності цього не роблять. Це допустимо, оскільки подібний підхід змушує користувача перехоплювати всі види виключень, які потім можуть бути додані в перевизначеній версії методу ***event()***. Даний принцип поширюється і на абстрактні методи, що і показано для методу ***atBat()***. 

Інтерфейс ***Storm*** цікавий тим, що містить один метод (***event()***), вже визначений у класі ***Inning***, і один унікальний. Обидва методи збуджують новий тип виключення ***RainedOut***. Коли клас ***StormyInning*** розширює ***Inning*** і реалізує інтерфейс ***Storm***, з'ясовується, що метод ***event()*** з ***Storm*** НЕ здатний змінити тип виключення для методу ***event()***
класу ***Inning***. Знову-таки це цілком розумно, тому що інакше ви б ніколи не знали, чи перехоплюєте потрібне виключення в разі роботи з базовим класом. Звичайно, коли метод, описаний в інтерфейсі, відсутній в базовому класі (як ***rainHard()***), ніяких проблем з порушенням виключень немає. 

Метод ***StormyInning.walk()*** НЕ компілюється через те, що він генерує виключення, тоді як ***Inning.walk()*** такого не робить. Якби це дозволялося, ви могли б написати код, що викликає метод ***Inning.walk()*** і не перехоплює ніяких виключень, а потім при підстановці об'єкта класу, похідного від Inning, виникли б виключення, що порушують роботу програми. Таким чином, примусово забезпечуючи відповідність специфікацій виключень в похідних і базових версіях методів, *Java* домагається взаємозамінності об'єктів. 

Перевизначений метод ***event()*** показує, що метод похідного класу може взагалі не порушувати виключень, навіть якщо це робиться в базовій версії. Знову-таки це нормально, тому що не впливає на вже написаний код - мається на увазі, що метод базового класу збуджує виключення. Аналогічна логіка застосовна для методу ***atBat()***, що порушує виключення ***PopFoul***, похідне від ***Foul***, яке порушується базовою версією ***atBat()***. Отже, якщо ви пишете код, який працює з ***Inning*** і викликає ***atBat()***, то він повинен перехоплювати виключення ***Foul***. Так як ***PopFoul*** успадковує від ***Foul***, обробник виключення для ***Foul*** перехопить і ***PopFoul***. 

Останній цікавий момент зустрічається в методі ***main()***. Ми бачимо, що при роботі саме з об'єктом ***StormyInning*** компілятор змушує перехоплювати тільки ті Виключення, які характерні для цього класу, але при висхідному перетворенні до базового типу компілятор змушує перехоплювати виключення з базового класу. Всі ці обмеження значно підвищують ясність і надійність коду обробки виключень. 

Хоча компілятор змушує описувати виключення при успадкуванні, специфікація виключень не є частиною оголошення (сигнатури) методу, яке складається тільки з імені методу і типів аргументів. Відповідно, не можна перевизначати методи тільки за специфікаціями виключень. До того ж, навіть якщо специфікація виключення присутня в методі базового класу, це зовсім не гарантує його існування в методі похідного класу. Дана практика сильно відрізняється від правил успадкування, за якими метод базового класу обов'язково присутній і в похідному класі. Іншими словами, «інтерфейс специфікації виключень» для певного методу може звузитися в процесі успадкування і перевизначення, але ніяк не розширитися - і це пряма протилежність інтерфейсу класу під час успадкування. 

## Конструктори

При програмуванні обробки виключень завжди запитуйте себе: «Якщо станеться виключення, чи буде все коректно завершено?» Найчастіше все йде більш-менш безпечно, але з конструкторами виникає проблема. Конструктор призводить об'єкт в певний початковий стан, але може почати виконувати будь-яку дію - таке як відкриття файлу - яка не буде правильно завершено, поки користувач не звільнить об'єкт, викликавши спеціальний завершальний метод. Якщо виключення станеться в конструкторі, ці фінальні дії можуть бути виконані помилково. А це означає, що при написанні конструкторів необхідно бути особливо уважним.
 
Здавалося б, блок ***finally*** вирішує всі проблеми. Але в дійсності все складніше - адже ***finally*** виконується завжди, і навіть тоді, коли завершальний код не повинен активізуватися до виклику якогось методу. Якщо збій в конструкторі відбудеться десь в середині, може виявитися, що частина об'єкта, що звільняється в ***finally***, ще не була створена.
 
У наступному прикладі створюється клас, названий ***InputFile***, який відкриває файл і дозволяє читати з нього по одному рядку. Він використовує класи ***FileReader*** і ***BufferedReader*** зі стандартної бібліотеки вводу/виводу *Java*, яка буде вивчена далі, але ці класи досить прості, і у вас не виникне особливих складнощів при роботі з ними :
 
``` java
//: exceptions/InputFile.java 
// Специфіка виключень в конструкторах 
import java.io.*; 

public class InputFile { 
  private BufferedReader in; 
  public InputFile(String fname) throws Exception { 
    try { 
      in = new BufferedReader(new FileReader(fname)); 
      // Other code that might throw exceptions 
    } catch(FileNotFoundException e) { 
      System.out.println("Could not open " + fname); 
      // Wasn’t open, so don’t close it 
      throw e; 
    } catch(Exception e) { 
      // All other exceptions must close it 
      try { 
        in.close(); 
      } catch(IOException e2) { 
        System.out.println("in.close() unsuccessful"); 
      } 
      throw e; // Rethrow 
    } finally { 
      // Don’t close it here!!! 
    } 
  } 
  public String getLine() { 
    String s; 
    try { 
      s = in.readLine(); 
    } catch(IOException e) { 
      throw new RuntimeException("readLine() failed"); 
    } 
    return s; 
  } 
  public void dispose() { 
    try { 
      in.close(); 
      System.out.println("dispose() successful"); 
    } catch(IOException e2) { 
      throw new RuntimeException("in.close() failed"); 
    } 
  } 
} ///:~ 
```
 
Конструктор ***InputFile*** отримує в якості аргументу рядок (***String***) з ім'ям файлу, що ви хочете відкрити. Усередині блоку ***try*** він створює об'єкт ***FileReader*** для цього файлу. Клас ***FileReader*** не дуже корисний сам по собі, тому ми вбудовуємо його в створений ***BufferedReader***, з яким і працюємо, - одна з переваг ***InputFile*** полягає в тому, що він об'єднує ці дві дії.
 
Якщо при виклику конструктора ***FileReader*** відбудеться збій, збуджується виключення ***FileNotFoundException***. В цьому випадку закривати файл не потрібно, так як він і не відкривався. Всі інші блоки ***catch*** зобов'язані закрити файл, так як він вже був відкритий під час входу в них. (Звичайно, все було б складніше в разі, якби кілька методів могли порушувати ***FileNotFoundException***. У таких ситуаціях зазвичай потрібно кілька блоків ***try***.) Метод ***close()*** теж може порушити виключення, яке також перевіряється і перехоплюється - незважаючи на те, що виклик знаходиться в іншому блоці ***catch*** - з точки зору компілятора *Java* це всього лише ще одна пара фігурних дужок. Після виконання всіх необхідних локальних дій виключення збуджується заново; адже викликаємий метод не повинен вважати, що об'єкт був успішно створений.
 
У цьому прикладі блок ***finally*** безумовно не підходить для закриття файлу, оскільки в такому варіанті закриття відбувалося б кожен раз по завершенні роботи конструктора. Ми хочемо, щоб файл залишався відкритим протягом усього життєвого циклу ***InputFile***.
 
Метод ***getLine()*** повертає об'єкт ***String*** з наступним рядком з файлу. Він викликає метод ***readLine()***, який здатний порушувати виключення, але вони перехоплюються; таким чином, сам ***getLine()*** виключень не збуджує. При проектуванні обробки виключень ви вибираєте між повною обробкою виключення на певному рівні, його часткової обробкою і передачею далі того ж (або іншого) виключення і, нарешті, простою передачею далі. Там, де це можливо, передача виключення значно спрощує програмування. У даній ситуації метод ***getLine()*** перетворює виключення в ***RuntimeException***, щоб вказати на помилку в програмі.

Метод ***dispose()*** повинен викликатися користувачем при завершенні роботи з об'єктом ***InputFile***. Він звільняє системні ресурси (такі, як відкриті файли), закріплені за об'єктами ***BufferedReader*** і (або) ***FileReader***. Робити це слід тільки тоді, коли робота з об'єктом ***InputFile*** дійсно буде завершена. Здавалося б, подібні дії зручно розмістити в методі ***finalize()***, але, як згадувалося в главі 5, виклик цього методу не гарантований (і навіть якщо ви знаєте, що він буде викликаний, то невідомо, коли). Це один з недоліків *Java*: всі завершальні дії, крім звільнення пам'яті, не робляються автоматично, так що вам доведеться інформувати користувача про те, що він відповідальний за їх виконання.
 
Найбезпечніший спосіб використання класу, який здатний видати виключення при конструюванні і вимагає завершальних дій, заснований на використанні вкладених блоків ***try***:
 
``` java
//: exceptions/Cleanup.java 
// Гарантоване звільнення ресурсів.
 
public class Cleanup { 
  public static void main(String[] args) { 
    try { 
      InputFile in = new InputFile("Cleanup.java"); 
      try { 
        String s; 
        int i = 1; 
        while((s = in.getLine()) != null) 
          ; // Perform line-by-line processing here... 
      } catch(Exception e) { 
        System.out.println("Caught Exception in main"); 
        e.printStackTrace(System.out); 
      } finally { 
        in.dispose(); 
      } 
    } catch(Exception e) { 
      System.out.println("InputFile construction failed"); 
    } 
  } 
} /* Output: 
dispose() successful 
*///:~ 
```
 
Придивіться до логіки того, що відбувається: конструювання об'єкта ***InputFile*** фактично укладено в власний блок ***try***. Якщо спроба завершується невдачею, ми входимо в зовнішнє секцію ***catch*** і метод ***dispose()*** не викликається. Але, якщо конструювання пройшло успішно, ми хочемо забезпечити гарантоване завершення, тому відразу ж після конструювання створюється новий блок ***try***. Блок ***finally***, що виконує завершення, зв'язується з внутрішнім блоком ***try***; таким чином, блок ***finally*** не виконується при невдалому конструюванні і завжди виконується, якщо конструювання пройшло вдало. Ця універсальна ідіома застосовується і в тих ситуаціях, коли конструктор не видає виключень. Основний принцип: відразу ж після створення об'єкта, що вимагає завершення, починається конструкція ***try-finally***:
 
``` java
//: exceptions/CleanupIdiom.java
// За кожним звільняються об'єктом слід try-finally
 
class NeedsCleanup { // Construction can’t fail 
  private static long counter = 1; 
  private final long id = counter++; 
  public void dispose() { 
    System.out.println("NeedsCleanup " + id + " disposed"); 
  } 
} 
 
class ConstructionException extends Exception {} 
 
class NeedsCleanup2 extends NeedsCleanup { 
  // Construction can fail: 
  public NeedsCleanup2() throws ConstructionException {} 
} 
 
public class CleanupIdiom { 
  public static void main(String[] args) { 
    // Section 1: 
    NeedsCleanup nc1 = new NeedsCleanup(); 
    try { 
      // ... 
    } finally { 
      nc1.dispose(); 
    } 
 
    // Section 2: 
    // If construction cannot fail you can group objects: 
    NeedsCleanup nc2 = new NeedsCleanup(); 
    NeedsCleanup nc3 = new NeedsCleanup(); 
    try { 
      // ... 
    } finally { 
      nc3.dispose(); // Reverse order of construction 
      nc2.dispose(); 
    } 
 
    // Section 3: 
    // If construction can fail you must guard each one: 
    try { 
      NeedsCleanup2 nc4 = new NeedsCleanup2(); 
      try { 
        NeedsCleanup2 nc5 = new NeedsCleanup2(); 
        try { 
          // ... 
        } finally { 
          nc5.dispose(); 
        } 
      } catch(ConstructionException e) { // nc5 constructor 
        System.out.println(e); 
      } finally { 
        nc4.dispose(); 
      } 
    } catch(ConstructionException e) { // nc4 constructor 
      System.out.println(e); 
    } 
  } 
} /* Output: 
NeedsCleanup 1 disposed 
NeedsCleanup 3 disposed 
NeedsCleanup 2 disposed 
NeedsCleanup 5 disposed 
NeedsCleanup 4 disposed 
*///:~
```
 
Секція 1 методу ***main()*** вельми прямолінійна: за створенням завершаємого об'єкта слідує ***try-finally***. Якщо конструювання не може завершитися невдачею, ***catch*** не потрібна. У секції 2 ми бачимо, що конструктори, які не можуть завершитися невдачею, можуть групуватися як для конструювання, так і для завершення.
 
Секція 3 показує, як чинити з об'єктами, при конструюванні яких можливі збої і які потребують завершення. Тут програма ускладнюється, тому що кожне конструювання має обгортатись своїм власним ***try-catch*** і за ним повинна слідувати конструкція ***try-finally***, що забезпечує завершення.
 
Незручності обробки виключення в подібних випадках - вагомий аргумент на користь створення конструкторів, виконання яких свідомо обходиться без збоїв (хоча це і не завжди можливо). 

## Ідентифікація виключень

Механізм обробки виключень шукає в списку «найближчий» відповідний обробник в порядку їх слідування. Коли відповідність виявляється, виключення вважається знайденим і подальшого пошуку не відбувається. Ідентифікація виключень не вимагає обов'язкової відповідності між виключенням і обробником. Об'єкт породженого класу підійде і для обробника, спочатку написаного для базового класу:
 
``` java
//: exceptions/Human.java
// Перехоплення ієрархії виключень.

class Annoyance extends Exception {} 
class Sneeze extends Annoyance {} 
 
public class Human { 
  public static void main(String[] args) { 
    // Catch the exact type: 
    try { 
      throw new Sneeze(); 
    } catch(Sneeze s) { 
      System.out.println("Caught Sneeze"); 
    } catch(Annoyance a) { 
      System.out.println("Caught Annoyance");
    } 
    // Catch the base type: 
    try { 
      throw new Sneeze(); 
    } catch(Annoyance a) { 
      System.out.println("Caught Annoyance"); 
    } 
  } 
} /* Output: 
Caught Sneeze 
Caught Annoyance 
*///:~       
```
 
Виключення ***Sneeze*** буде перехоплено в першому блоці ***catch***, який йому відповідає - звичайно, це буде перший блок. Але, якщо видалити перший блок ***catch***, залишивши тільки перевірку ***Annoyance***, програма все одно буде працювати, тому що вона перехоплює базовий клас ***Sneeze***. Іншими словами, блок ***catch (Annoyance а)*** зловить ***Annoyance***, або будь-який інший клас, успадкований від нього. Якщо ви додасте нові похідні виключення в свій метод, програма користувача цього методу не потребує змін, так як клієнт перехоплює виключення базового класу. Якщо ви спробуєте «замаскувати» виключення похідного класу, помістивши спочатку блок ***catch*** базового класу:
 
``` java
try { 
  throw new Sneeze(); 
} catch(Annoyance a) { 
// ... 
} catch(Sneeze s) { 
// ... 
} 
```
 
компілятор видасть повідомлення про помилку, так як він бачить, що блок ***catch*** для виключення ***Sneeze*** ніколи не виконається.

## Альтернативні рішення

Система обробки виключень це «чорний хід», що дозволяє програмі порушити нормальну послідовність виконання команд. «Чорний хід» відкривається при виникненні «виключних ситуацій», коли звичайна робота далі неможлива, або небажана. Виключення становлять собою умови, з якими поточний метод впоратися не в змозі. Причина, по якій виникають системи обробки виключень, криється в тому, що програмісти не бажали мати справи з громіздкою перевіркою всіх можливих умов виникнення помилок кожної функції. В результаті помилки ними просто ігнорувалися. Варто зазначити, що питання зручності програміста при обробці помилок стояло на першому місці для розробників *Java*.
 
Основне правило при використанні виключень говорить: «Не обробляйте виключення, якщо ви не знаєте, що з ним робити». По суті, відділення коду, відповідального за обробку помилок, від місця, де помилка виникає, є однією з головних цілей обробки виключень. Це дозволяє вам сконцентруватися на тому, що ви хочете зробити в одному фрагменті коду, і на тому, як ви збираєтеся вчинити з помилками в зовсім іншому місці програми. В результаті основний код не перемежовується з логікою обробки помилок, що спрощує його супровід і розуміння. Виключення також скорочують обсяг коду, так як один обробник може обслуговувати кілька потенційних джерел помилок.
 
Контрольовані виключення трохи ускладнюють ситуацію, оскільки вони змушують додавати обробку виключеннь там, де ви не завжди ще готові впоратися з помилкою. В результаті виникає проблема «проковтання виключень»:
 
``` java
try { 
// ... робимо щось корисне
} catch(ObligatoryException e) {} //  Проковтнули!   
```
 
Програмісти (і я в тому числі, в першому виданні книги), не довго думаючи, робили найпростіші речі і «ковтали» виключення - найчастіше ненавмисно, але, як тільки справа була зроблена, компілятор був задоволений, тому поки ви не згадували про необхідність переглянути і виправити код, не згадували і про виключення. Виключення відбувається, але безповоротно втрачається. Через те що компілятор змушує вас писати код для обробки виключень прямо на місці, це здається найпростішим рішенням, хоча насправді нічого гіршого і придумати не можна.
 
Жахнувшись тим, що я так вчинив, у другому виданні книги я «виправив» проблему, виводячи в обробнику трасування стека виключення (і зараз це можна бачити - в потрібних місцях - в деяких прикладах цього розділу). Хоча це і корисно при відстеженні поведінки виключень, трасування фактично означає, що ви так і не знаєте, що ж робити з виключенням в даному фрагменті коду. У цьому розділі ми розглянемо деякі тонкощі і ускладнення, що породжуються контрольованими виключеннями, і варіанти роботи з останніми.
 
Незважаючи на гадану простоту, проблема не тільки дуже складна, але і до того ж неоднозначна. Існують тверді прихильники обох точок зору, які вважають, що правильна відповідь очевидна і просто кидається в очі. Ймовірно, одна з точок зору заснована на безсумнівній перевазі переходу від слабо типизированної мови (наприклад, *C* до виходу стандарту ANSI) до мови з суворою статичною перевіркою типів (тобто з перевіркою під час компіляції), подібною до *C++*, або *Java*. Переваги такого переходу настільки очевидні, що сувора статична перевірка типів здається панацеєю від усіх бід. Я сподіваюся поставити під сумнів ту невелику частину моєї еволюції, що відрізняється абсолютною вірою в строгу статичну перевірку типів: без сумніву, велику частину часу вона приносить користь, але існує неформальна межа, за якою така перевірка стає перешкодою на вашому шляху (одна з моїх улюблених цитат така: «Всі моделі невірні, але деякі корисні»).
 
### Передісторія

Обробка виключень зародилася в таких системах, як *PL/1* і *Mesa*, а потім мігрувала в *CLU, Smalltalk, Modula-3, Ada, Eiffel, С++, Python, Java* і в ті що з'явилися після *Java*, мови *Ruby* і *С#*. Конструкції *Java* схожі з конструкціями *C++*, крім тих аспектів, в яких рішення *C++* призводили до проблем.
 
Обробка виключень була додана в *C++* на досить пізньому етапі стандартизації. Модель виключень в *C++* в основному була запозичена з *CLU*. Втім, в той час існували і інші мови з підтримкою обробки виключень: *Ada, Smalltalk* (в обох були виключення, але були відсутні їх специфікації) і *Modula-З* (в якому існували і виключення, і їх специфікації).
 
Дотримуючись підходу *CLU* при розробці виключень *C++*, Страуструп вважав, що основною метою є скорочення обсягу коду відновлення після помилки. Ймовірно, він бачив чимало програмістів, які не писали код обробки помилок на *C*, оскільки обсяг цього коду був страхітливим, а розміщення виглядало нелогічно. В результаті все відбувалося в стилі *C*: помилки в коді ігнорувалися, а з проблемами справлялися за допомогою відладчика. Щоб виключення реально запрацювали, *C* -програмісти повинні були писати «зайвий» код, без якого вони зазвичай обходилися. Таким чином, обсяг нового коду не повинен бути надмірним. Важливо пам'ятати про ці цілі, кажучи про ефективність контрольованих виключень в *Java*.
 
*C++* додав до ідеї *CLU* додаткову можливість: специфікації виключень, тобто включення в сигнатуру методу інформації про виключення, що виникають при виклику. Насправді специфікація виключення несе подвійний сенс. Вона означає: «Я порушую це виключення у коді, а ви його обробляєте». Але вона також може означати: «Я ігнорую виключення, яке може виникнути в моєму коді; забезпечте його обробку». При висвітленні механізмів виключень ми концентрувалися на «забезпеченні обробки», але тут мені хотілося б ближче розглянути той факт, що найчастіше виключення ігноруються, і саме цей факт може бути відображений в специфікації виключення.
 
В *C++* специфікація виключення не входить в інформацію про тип функції. Єдина перевірка, що здійснюється під час компіляції, відноситься до узгодженого використання виключень: наприклад, якщо функція або метод збуджує виключення, то перевантажена, або перевизначена версія повинна порушувати ті ж самі виключення. Однак, на відміну від *Java*, компілятор не перевіряє, чи дійсно функція або метод збуджують цей виключення, або повноту специфікації (тобто описує вона все виключення, можливі для цього методу). Якщо збуджує виключення, що не входить в специфікацію, програма на *C++* викликає функцію *unexpected()* зі стандартної бібліотеки.
 
Цікаво відзначити, що через використання шаблонів (*templates*) специфікації виключень відсутні в стандартній бібліотеці *C++*. В *Java* існують обмеження на використання параметризованих типів зі специфікаціями виключень.
 
### Перспективи

По-перше, мова *Java*, по суті, стала першопрохідцем у використанні контрольованих виключень (безсумнівно через специфікацій виключень *C++* і того факту, що програмісти на *C++* не приділяли їм занадто багато уваги). Це був експеримент, повторити який з тих пір поки не наважилась ще жодна мова.
 
По-друге, контрольовані виключення однозначно хороші при розгляді вступних прикладів і в невеликих програмах. Виявляється, що важковловимі проблеми починають проявлятися при розростанні програми. Звичайно, програми не розростаються тут же і відразу, але вони мають тенденцію зростати непомітно. І коли мови, не призначені для великих проектів, використовуються для невеликих, але зростаючих проектів, ми в певний момент з подивом виявляємо, що ситуація змінилася з керованою на скрутну в управлінні. Саме це, як я вважаю, може статися, коли перевірок типів занадто багато, і особливо щодо контрольованих виключень.
 
Одним з важливих досягнень *Java* стала уніфікація моделі передачі інформації про помилки, так як про всі помилки повідомляється за допомогою виключень. В *C++* цього не було, через забезпечення сумісності з *C* і можливості задіяти стару модель простого ігнорування помилок. Коли *Java* змінила модель *C++* так, що повідомляти про помилки стало можливо тільки за допомогою виключень, необхідність в додаткових заходах примусу у вигляді контрольованих виключень скоротилася.
 
У минулому я твердо вважав, що для розробки надійних програм необхідні і контрольовані виключення, і сувора статична перевірка типів. Однак досвід, отриманий особисто, і з сторони, з мовами, більш динамічними, ніж статичними, привів мене до думки, що насправді головні переваги обумовлені наступними аспектами:

1. Уніфікація моделі повідомлення про помилки за допомогою виключень (незалежно від того, чи змушує компілятор програміста їх обробляти).
2. Перевірка типів, не прив'язана до того, коли вона проводиться - на стадії компіляції, або під час роботи програми.
 
До того ж зниження обмежень часу компіляції вельми позитивно відбивається на продуктивності програміста. З іншого боку, для компенсації надмірної жорсткості статичної перевірки типів необхідні рефлексія і параметризація, як ви переконаєтеся в деяких прикладах книги. 

Дехто запевняв мене, що все сказане є блюзнірством, що безнадійно зіпсує мою репутацію, призведе до загибелі цивілізації і провалу великої частки програмних проектів. Віра в те, що виявлення помилок на стадії компіляції врятує ваш проект, вельми сильна, але набагато важливіше усвідомлювати обмеження того, на що здатний комп'ютер. Варто пам'ятати: 

*«Хороша мова програмування допомагає програмістам писати хороші програми. Жодна з мов програмування не може заборонити своїм користувачам писати погані програми».*

У будь-якому випадку видалення з *Java* контрольованих виключень дуже малоймовірно. Це занадто радикальна зміна мови, і захисники їх в Sun дуже сильні. Історія Sun невіддільна від політики абсолютної сумісності - фактично будь-яке програмне забезпечення Sun
працює на будь-якому обладнанні Sun, яким би старим воно не було. Але, якщо ви відчуваєте, що контрольовані виключення стають для вас перешкодою (особливо якщо вас змушують обробляти виключення, а ви не знаєте, як з ним вчинити), існує кілька варіантів.
 
## Передача виключень на консоль
 
У нескладних програмах, як у багатьох прикладах даної книги, найпростішим рішенням є передача виключення за межі методу ***main()***, на консоль. Наприклад, при відкритті файлу для читання (подробиці ви незабаром дізнаєтеся) необхідно відкрити і закрити потік ***FilelnputStream***, який збуджує виключення. У невеликій програмі можна поступити наступним чином (подібний підхід характерний для багатьох прикладів книги):
 
``` java
//: exceptions/MainException.java 
import java.io.*; 
 
public class MainException { 
  // Передаємо всіх виключення на консоль: 
  public static void main(String[] args) throws Exception { 
    // Відкриваємо файл: 
    FileInputStream file = 
      new FileInputStream("MainException.java"); 
    // Використовуємо файл ... 
    // Закриваємо файл: 
    file.close(); 
  } 
} ///:~ 
```
 
Зауважте, що ***main()*** - такий же метод, як і всі інші; він теж може мати специфікацію виключень, і тут типом виключення є ***Exception***, базовий клас всіх контрольованих виключень. Передаючи його на консоль, ви звільняєтесь від необхідності написання ***try-catch*** в тілі методу ***main()***.
 
### Перетворення контрольованих виключень в неконтрольовані
 
Розглянутий вище підхід хороший при написанні методу ***main()***, але в більш загальних ситуаціях не дуже корисний. Справжня проблема виникає при написанні тіла самого простого способу, коли при виклику іншого методу ви чітко усвідомлюєте: «Поняття не маю, що робити з виключенням далі, але ковтати його не хочеться, так само як і друкувати банальне повідомлення». Проблема вирішується за допомогою ланцюжків виключень. Кероване виключення просто «загортається» у клас ***RuntimeException*** приблизно так:
 
``` java
try { 
  // ... робимо щось корисне
} catch(IDontKnowWhatToDoWithThisCheckedException e) { 
  throw new RuntimeException(e); 
} 
```
 
Рішення ідеально підходить для тих випадків, коли ви хочете «придушити» контрольоване виключення: ви не «ковтаєте» його, вам не доводиться описувати його в своїй специфікації виключень, і завдяки ланцюжку виключень ви не втрачаєте інформацію про початкове виключення.
 
Описана методика дозволяє ігнорувати виключення і пустити його «спливати» вгору по стеку виклику без необхідності писати блоки ***try-catch*** і (або) специфікації виключення. Втім, при цьому ви все одно можете перехопити і обробити конкретне виключення, використовуючи метод ***getCause()***, як показано нижче:
 
``` java
//: exceptions/TurnOffChecking.java 
// "Придушення" контрольованих виключень. 
import java.io.*; 
import static net.mindview.util.Print.*; 
 
class WrapCheckedException { 
  void throwRuntimeException(int type) { 
    try { 
      switch(type) { 
        case 0: throw new FileNotFoundException(); 
        case 1: throw new IOException(); 
        case 2: throw new RuntimeException("Where am I?"); 
        default: return; 
      } 
    } catch(Exception e) { // Перетворюємо в неконтрольоване:
      throw new RuntimeException(e); 
    } 
  } 
} 
 
class SomeOtherException extends Exception {} 
 
public class TurnOffChecking { 
  public static void main(String[] args) { 
    WrapCheckedException wce = new WrapCheckedException(); 
    // Ви можете викликати throwRuntimeException() без блоку try
    // і дозволити виключенню RuntimeException покинути метод:
    wce.throwRuntimeException(3); 
    // Або перехопити виключення:
    for(int i = 0; i < 4; i++) 
      try { 
        if(i < 3) 
          wce.throwRuntimeException(i); 
        else 
          throw new SomeOtherException(); 
      } catch(SomeOtherException e) { 
          print("SomeOtherException: " + e); 
      } catch(RuntimeException re) { 
        try { 
          throw re.getCause(); 
        } catch(FileNotFoundException e) { 
          print("FileNotFoundException: " + e); 
        } catch(IOException e) { 
          print("IOException: " + e); 
        } catch(Throwable e) { 
          print("Throwable: " + e); 
        } 
      } 
  } 
} /* Output: 
FileNotFoundException: java.io.FileNotFoundException 
IOException: java.io.IOException 
Throwable: java.lang.RuntimeException: Where am I? 
SomeOtherException: SomeOtherException 
*///:~ 
```
 
Метод ***WrapCheckedException.throwRuntimeException()*** містить код, що генерує різні типи виключень. Вони перехоплюються і «загортаються» в об'єкти ***RuntimeException***, стаючи таким чином «причиною» цих виключень. 

В класі ***TurnOffChecking*** неважко помітити, що викликати метод ***throwRuntimeException()*** можна і без блоку ***try***, оскільки він не збуджує ніяких контрольованих виключень. Але коли ви будете готові перехопити виключення, у вас буде можливість перехопити будь-яку з них - досить помістити свій код в блок ***try***. Починаєте ви з перехоплення виключень, які, як ви знаєте, можуть явно виникнути в коді блоку ***try***, - в нашому випадку в першу чергу перехоплюється ***SomeOtherException***. В кінці ви перехоплюєте ***RuntimeException*** і заново збуджуєте виключення, що є його причиною (отримуючи останнім методом ***getCause()***, «загорнуте»
виключення). Так витягуються початкові виключення, оброблювані в своїх блоках ***catch***. 

Методика «загортання» керованих виключень в об'єкти ***RuntimeException*** зустрічається в деяких прикладах книги. Інше можливе рішення - створення власного класу, похідного від ***RuntimeException***. Перехоплювати таке виключення не обов'язково, але, якщо ви захочете, така можливість існує. 

## Основні правила обробки виключень

Використовуйте виключення для того, щоб:

1. обробити помилку на поточному рівні (уникайте перехоплювати виключення, якщо ви не знаєте, як з ними вчинити);
2. виправити проблему і знову викликати метод, який порушив виключення;
3. вжити всіх необхідних заходів і продовжити виконання без повторного виклику методу;
4. спробувати знайти альтернативний результат замість того, який мав би зробити викликаний метод;
5. зробити все можливе в поточному контексті і заново порушити це ж виключення, перенаправивши його на більш високий рівень;
6. зробити все, що можна в поточному контексті, і порушити нове виключення, перенаправивши його на більш високий рівень;
7. завершити роботу програми;
8. спростити програму (якщо використовувана вами схема обробки виключень робить все тільки складніше, значить, вона нікуди не годиться);
9. зробити вашу бібліотеку і програму безпечнішою (спочатку це допоможе в налагодженні програми, а в подальшому окупиться її надійністю). 

## Резюме

Виключення є невід'ємною частиною програмування на *Java*; існує якийсь бар'єр, який неможливо подолати без уміння працювати з ними. З цієї причини виключення були представлені саме в цій частині книги - багатьма бібліотеками (скажімо, бібліотекою вводу/виводу) просто неможливо нормально користуватися без обробки виключень. 

Одна з переваг обробки виключень полягає в тому, що вона дозволяє зосередитися на розв'язуванні проблеми, а потім обробити всі помилки в описаному коді в іншому місці. Хоча виключення зазвичай описуються як засіб передачі інформації і відновлення після помилок на стадії виконання, я сильно сумніваюся, що «відновлення» часто реалізується на практиці. За моєю оцінкою, це відбувається не більше ніж в 10% випадків, і навіть тоді в основному все зводиться до розкручування стека до свідомо стабільного стану замість реального виконання дій з відновлення. На мій погляд, цінність виключень в основному обумовлена ​​саме передачею інформації. *Java* фактично наполягає, що програма повинна повідомляти про всі помилки у вигляді виключень, і саме ця обставина забезпечує *Java* велику перевагу перед мовами подібними на *C++*, де програма може повідомляти про помилки різними способами (а то і не повідомляти зовсім) .
 