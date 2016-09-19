# Контейнери і зберігання об'єктів
 
Обмежена кількість об'єктів з фіксованим часом життя характерна хіба що для відносно простих програм. В основному ваші програми будуть створювати нові об'єкти на підставі критеріїв, які стануть відомі лише під час їх роботи. До початку виконання програми ви не знаєте ні кількості, ні навіть типів потрібних вам об'єктів. Отже, використовувати іменоване посилання для кожного з можливих об'єктів не вдасться:
 
``` java
МуТуре aReference;
```
 
так як заздалегідь невідомо, скільки таких посилань реально потрібно. У більшості мов існують деякі шляхи вирішення цього вкрай нагального завдання. В *Java* передбачено кілька способів зберігання об'єктів (або, точніше, посилань на об'єкти). Вбудованим типом є масив, який ми вже розглянули. Бібліотека утиліт *Java* (***java.util.\****) також містить досить повний набір класів контейнерів (також відомих, як класи колекцій, але, оскільки ім'я ***Collection*** (колекція) використовується для позначення певної підмножини бібліотеки *Java*, я буду вживати загальний термін «контейнер»). Контейнери мають досить витончені можливості для зберігання об'єктів і роботи з ними, і з їх допомогою вдається вирішити величезну кількість завдань. 

# Параметризовані і типізовані контейнери

Одна з проблем, що існували при роботі з контейнерами до виходу *Java* *SE5*, полягала в тому, що компілятор дозволяв вставити в контейнер об'єкт невірного типу. Для прикладу розглянемо один з основних робочих контейнерів ***ArrayList***, в якому ми збираємося зберігати об'єкти ***Apple***. Поки розглядайте ***ArrayList*** як «автоматично розширюваний масив». Працювати з ним нескладно: створіть об'єкт, вставляйте об'єкти методом ***add()***, звертайтеcь до них методом ***get()***, використовуйте індексування - так само, як для масивів, але без квадратних дужок. ***ArrayList*** також містить метод ***size()***, який повертає поточну кількість елементів в масиві. У наступному прикладі в контейнері розміщуються об'єкти ***Apple*** і ***Orange***, які потім витягуються з нього. Зазвичай компілятор *Java* видає попередження, тому що в даному прикладі не використовується параметризация, проте в *Java* *SE5* існує спеціальна директива ***@SuppressWarnings*** для виключення попереджень. Директиви починаються зі знака ***@*** і можуть отримувати аргументи; в даному випадку аргумент означає, що виключаються тільки «неперевіряємі» попередження:
 
``` java
//: Holding/ApplesAndOrangesWithoutGenerics.java
// Простий приклад роботи з контейнером 
// (компілятор видає попередження). 
import java.util.*;

class Apple {
    private static long counter;
    private final long id = counter ++;
    public long id() {return id;}
}

class Orange {}

public class ApplesAndOrangesWithoutGenerics {
    @SuppressWarnings ( "Unchecked")
    public static void main (String [] args) {
        ArrayList apples = new ArrayList ();
        for (int i = 0; i <3; i ++)
            apples.add (new Apple ());
        // He перешкоджає додаванню об'єкта Orange:
        apples.add (new Orange ());
        for (int i = 0; i <apples.size (); i ++)
            ((Apple) apples.get (i)).Id ();
            // Об'єкт Orange виявляється тільки під час виконання
    }
}
```
 
Директиви *Java* *SE5* будуть розглянуті пізніше. ***Apple*** і ***Orange*** - абсолютно різні класи; вони не мають нічого спільного, крім походження від ***Object*** (нагадаю: якщо в програмі явно не вказано базовий клас, то в цій якості використовується ***Object***). Так як в ***ArrayList*** зберігаються об'єкти ***Object***, метод ***add()*** може додавати в контейнер не тільки об'єкти ***Apple***, але і ***Orange***, без помилок компіляції, або часу виконання. Але при виклику методу ***get()*** класу ***ArrayList*** ви замість об'єкта ***Apple*** отримуєте посилання на ***Object***, яку необхідно перетворити в ***Apple***. Весь вираз має бути оточений круглими дужками, щоб перетворення було виконано перед викликом методу ***id()*** класу ***Apple***. Під час виконання, при спробі перетворення об'єкта ***Orange*** в ***Apple***, відбудеться виключення. У розділі «параметризовані типи» ви дізнаєтеся, що створення класів, які використовують механізм параметризації, може бути досить складним завданням. З іншого боку, з застосуванням готових параметризованих класів проблем зазвичай не буває. Наприклад, щоб визначити об'єкт ***ArrayList***, призначений для зберігання об'єктів ***Apple***, досить використовувати замість імені ***ArrayList*** запис ***АrrayList<Apple>***. У кутових дужках перераховуються параметри типів (їх може бути кілька), що вказують на тип об'єктів, що зберігаються в даному екземплярі контейнера. Механізм параметризації запобігає занесення об'єктів невірного типу в контейнер на стадії компіляції. Розглянемо той же приклад, але з використанням параметризації:
 
``` java
//: Holding/ApplesAndOrangesWithGenerics.java
import java.util.*;

public class ApplesAndOrangesWithGenerics {
    public static void main (String [] args) {
        ArrayList <Apple> apples = new ArrayList <Apple> ();
        for (int i = 0; i <3; i ++)
            apples.add (new Apple ());
            // Помилка компіляції:
            // Apples.add (new Orange ());
        for (int i = 0; i <apples.size (); i ++)
            System.out.println (apples.get (i) .id ());
            // Використання foreach:
        for (Apple c   : Apples)
            System.out.print (c.id () + "");
    }
} /* Output: 
0 
1 
2 
0 
1 
2 
*///:~ 
```
 
На цей раз компілятор не дозволить помістити об'єкти ***Orange*** в контейнер ***apples***, тому ви отримаєте помилку на стадії компіляції (а не на стадії виконання). Також зверніть увагу на те, що вибірка даних з ***List*** не вимагає перетворення типів. Оскільки контейнер знає тип елементів що зберігаються в ньому, він автоматично виконує перетворення при виклику ***get()***. Таким чином, параметризація не тільки дозволяє компілятору перевіряти тип об'єктів, які поміщені в контейнери, а й спрощує синтаксис роботи з об'єктами в контейнері. Приклад також показує, що, якщо індекси елементів вам не потрібні, для перебору можна скористатися синтаксисом ***foreach***. Ви не зобов'язані точно дотримуватися типу об'єкта, що зазначений в якості параметра типу. Висхідне перетворення працює з параметризованими контейнерами точно так само, як і з іншими типами:
 
``` java
//: holding/GenericsAndUpcasting.java 
import java.util.*; 
 
class GrannySmith extends Apple {} 
class Gala extends Apple {} 
class Fuji extends Apple {} 
class Braeburn extends Apple {} 
 
public class GenericsAndUpcasting { 
  public static void main(String[] args) { 
    ArrayList<Apple> apples = new ArrayList<Apple>(); 
    apples.add(new GrannySmith()); 
    apples.add(new Gala()); 
    apples.add(new Fuji()); 
    apples.add(new Braeburn()); 
    for(Apple c : apples) 
      System.out.println(c); 
  } 
} /* Output: (Sample) 
GrannySmith@7d772e 
Gala@11b86e7 
Fuji@35ce36 
Braeburn@757aef 
*///:~     
```
 
Ми бачимо, що в контейнері, що розрахований на зберігання об'єктів ***Apple***, можна поміщати об'єкти типів, похідних від ***Apple***. 

Результат отриманий з використанням методу ***toString()*** об'єкта ***Object***, який виводить назву класу з беззнаковим поданням хеш коду об'єкту.
 
# Основні концепції
 
У бібліотеці контейнерів *Java* проблема зберігання об'єктів ділиться на дві концепції, виражені у вигляді базових інтерфейсів бібліотеки:
- **Колекція:** група окремих елементів, сформована за деякими правилами. Клас ***List*** (список) зберігає елементи в порядку вставки, в класі ***Set*** (безліч) не можна зберігати повторювані елементи, а клас ***Queue*** (черга) видає елементи в порядку , який визначається специфікою черги (зазвичай це порядок вставки елементів в чергу).
- **Карта:** набір пар об'єктів «ключ-значення», з можливістю вибірки по ключу. ***ArrayList*** дозволяє шукати об'єкти за порядковими номерами, тому в якомусь сенсі він пов'язує числа з об'єктами. Клас ***Map*** (карта - також зустрічаються терміни асоціативний масив і словник) дозволяє шукати об'єкти по інших об'єктах - наприклад, отримати значення об'єкта по ключу об'єкта, за аналогією з пошуком визначення по слову.

Хоча на практиці це не завжди можливо, в ідеалі весь програмний код повинен писатися в розрахунку на взаємодію з цими інтерфейсами, а точний тип вказується тільки в точці створення. Отже, об'єкт ***List*** може бути створений так:
 
``` java
List <Apple> apples = new ArrayList <Apple>();
```
 
Зверніть увагу на висхідне перетворення ***ArrayList*** до ***List***, на відміну від попередніх прикладів. Якщо пізніше ви вирішите змінити реалізацію, досить зробити це в точці створення:
 
``` java
List <Apple> apples = new LinkedList <Apple>();
```
 
Отже, в типовій ситуації ви створюєте об'єкт реального класу, підвищуєте його до відповідного інтерфейсу, а потім використовуєте інтерфейс в усьому іншому коді. Такий підхід працює не завжди, тому що деякі класи володіють додатковою функціональністю. Наприклад, ***LinkedList*** містить додаткові методи, що не входять в інтерфейс ***List***, а ***ТrееМар*** - методи, що не входять в ***Map***. Якщо такі методи використовуються в програмі, висхідне перетворення до узагальненого інтерфейсу неможливе. Інтерфейс ***Collection***
представляє концепцію послідовності як способу зберігання групи об'єктів. У наступному простому прикладі інтерфейс ***Collection*** 
(представлений контейнером ***ArrayList***) заповнюється об'єктами ***Integer***, з подальшим виводом всіх елементів отриманого контейнера:
 
``` java
//: holding/SimpleCollection.java 
import java.util.*; 
 
public class SimpleCollection { 
  public static void main(String[] args) { 
    Collection<Integer> c = new ArrayList<Integer>(); 
    for(int i = 0; i < 10; i++) 
      c.add(i); // Autoboxing 
    for(Integer i : c) 
      System.out.print(i + ", "); 
  } 
} /* Output: 
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 
*///:~ 
```
 
Оскільки в цьому прикладі використовуються тільки методи ***Collection***, підійде об'єкт будь-якого класу, похідного від ***Collection***, але ***ArrayList*** є найпростішим типом послідовності.
 
Всі колекції підтримують перебір в синтаксисі ***foreach***, як в наведеному прикладі. Пізніше в цій главі буде розглянута інша, більш гнучка концепція ітераторів. 

# Додавання груп елементів

Сімейства ***Arrays*** і ***Collections*** в ***java.util*** містять допоміжні методи для включення груп елементів в колекції. Метод ***Arrays.asList()*** отримує, або масив, або список елементів, розділених комами, і перетворює його в об'єкт ***List***. Метод ***Collections.addAll()*** отримує об'єкт ***Collection*** і або масив, або список, розділений комами, і додає елементи в ***Collection***. Приклад:
 
``` java
//: holding/AddingGroups.java 
// Додавання груп елементів в об'єкти Collection
import java.util.*; 
 
public class AddingGroups { 
  public static void main(String[] args) { 
    Collection<Integer> collection = 
      new ArrayList<Integer>(Arrays.asList(1, 2, 3, 4, 5)); 
    Integer[] moreInts = { 6, 7, 8, 9, 10 }; 
    collection.addAll(Arrays.asList(moreInts)); 
    // Runs significantly faster, but you can’t 
    // construct a Collection this way: 
    Collections.addAll(collection, 11, 12, 13, 14, 15); 
    Collections.addAll(collection, moreInts); 
    // Produces a list "backed by" an array: 
    List<Integer> list = Arrays.asList(16, 17, 18, 19, 20); 
    list.set(1, 99); // OK -- modify an element 
    // list.add(21); // Runtime error because the 
                     // underlying array cannot be resized. 
  } 
} ///:~     
```
 
Конструктор ***Collection*** може отримувати інший об'єкт ***Collection***, який використовується для його ініціалізації, тому для передачі вихідних даних можна скористатися методом ***Arrays.asList()***. Однак метод ***Collections.addAll()*** працює набагато швидше, і ви з таким же успіхом можете сконструювати ***Collection*** без елементів, а потім викликати ***Collections.addAll***, цей спосіб вважається кращим. 

Методу ***Collection.addAll()*** в аргументі може передаватися тільки інший об'єкт ***Collection***, тому він поступається в гнучкості методів ***Arrays.asList()*** і ***Collections.addAll()***, що використовують змінні списки аргументів. 

Також можна використовувати виклик ***Arrays.asList()*** безпосередньо, у вигляді ***List***, але в цьому випадку базовим поданням буде масив, що не може бути змінного розміру. Виклик ***add()***, або ***delete()*** для такого списку призведе до спроби зміни розміру масиву, а це призведе до помилки під час виконання. Недолік ***Arrays.asList()*** полягає в тому, що він намагається «вирахувати» результуючий тип ***List***, не звертаючи уваги на те, що йому присвоюється. Іноді це створює проблеми:
 
``` java
//: holding/AsListInference.java 
// Arrays.asList() makes its best guess about type. 

import java.util.*; 
 
class Snow {} 
class Powder extends Snow {} 
class Light extends Powder {} 
class Heavy extends Powder {} 
class Crusty extends Snow {} 
class Slush extends Snow {} 
 
public class AsListInference { 
  public static void main(String[] args) { 
    List<Snow> snow1 = Arrays.asList( 
      new Crusty(), new Slush(), new Powder()); 
 
    // Не компілюється: 
    // List<Snow> snow2 = Arrays.asList( 
    //   new Light(), new Heavy()); 
    // Повідомлення компілятора: 
    // found   : java.util.List<Powder> 
    // required: java.util.List<Snow> 
 
    // Collections.addAll() працює нормально: 
    List<Snow> snow3 = new ArrayList<Snow>(); 
    Collections.addAll(snow3, new Light(), new Heavy()); 
 
	// Передача інформації за допомогою уточнення
	// типу аргументу: 
    List<Snow> snow4 = Arrays.<Snow>asList( 
       new Light(), new Heavy()); 
  } 
} ///:~ 
```
 
При спробі створення ***snow2, Arrays.asList()*** створює ***List<Powder>*** замість ***List<Snow>***, тоді як ***Collections.addAll()*** працює нормально, тому що цільовий тип визначається першим аргументом. 

Як видно з створення ***snow4***, в виклик ***Arrays.asList()*** можна вставити «підказку», яка повідомляє компілятору фактичний тип об'єкта ***List***, створенного ***Arrays.asList()***. З контейнерами ***Map*** справа йде складніше, і стандартна бібліотека *​​Java* не надає іншої можливості їх автоматичної ініціалізації, окрім як по вмісту іншого об'єкта ***Map***. 

# Вивід вмісту контейнерів

Для отримання друкованого уявлення масиву необхідно використовувати метод ***Arrays.toString***, але контейнери відмінно виводяться і без сторонньої допомоги. Наступний приклад демонструє використання основних типів контейнерів:
 
``` java
//: holding/PrintingContainers.java 
// Вивід контейнерів за замовчуванням
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class PrintingContainers { 
  static Collection fill(Collection<String> collection) { 
    collection.add("rat"); 
    collection.add("cat"); 
    collection.add("dog"); 
    collection.add("dog"); 
    return collection; 
  } 
  static Map fill(Map<String,String> map) { 
    map.put("rat", "Fuzzy"); 
    map.put("cat", "Rags"); 
    map.put("dog", "Bosco"); 
    map.put("dog", "Spot"); 
    return map; 
  }  
  public static void main(String[] args) { 
    print(fill(new ArrayList<String>())); 
    print(fill(new LinkedList<String>())); 
    print(fill(new HashSet<String>())); 
    print(fill(new TreeSet<String>())); 
    print(fill(new LinkedHashSet<String>())); 
    print(fill(new HashMap<String,String>())); 
    print(fill(new TreeMap<String,String>())); 
    print(fill(new LinkedHashMap<String,String>())); 
  } 
} /* Output: 
[rat, cat, dog, dog] 
[rat, cat, dog, dog] 
[dog, cat, rat] 
Holding Your Objects  281 [cat, dog, rat] 
[rat, cat, dog] 
{dog=Spot, cat=Rags, rat=Fuzzy} 
{cat=Rags, dog=Spot, rat=Fuzzy} 
{rat=Fuzzy, cat=Rags, dog=Spot} 
*///:~
```
 
Як уже було згадано, в бібліотеці контейнерів *Java* існує дві основні категорії, що розрізняються перш за все тим, скільки в одному екземплярі контейнера «поміщається» елементів. Колекції (***Collection***) містять тільки один елемент в кожному екземплярі. До цієї категорії відносяться список (***List***), де в певній послідовності зберігається група елементів, безліч (***Set***), в яке можна додавати тільки між окремими елементами певного типу, і черга (***Queue***). У контейнерах ***Map*** (карта) зберігаються два об'єкти: ключ і пов'язане з ним значення. 

З вихідних даних програми видно, що вивід за замовчуванням (який забезпечується шляхом ***toString ()*** кожного контейнера) дає цілком пристойні результати. Вміст ***Collection*** виводиться в квадратних дужках, з поділом елементів комами. Вміст ***Map*** виводиться в фігурних дужках, ключі і значення розділяються знаком рівності (ключі зліва, значення праворуч). 

Контейнери ***ArrayList*** і ***LinkedList*** належать до сімейства ***List***, і з виводу даних видно, що елементи в них зберігаються в порядку вставки. Вони розрізняються не тільки швидкістю виконання тих чи інших операцій, але і тим, що ***LinkedList*** містить більше операцій, ніж ***ArrayList***. 

***HashSet, TreeSet*** і ***LinkedHashSet*** відносяться до сімейства ***Set***. З виводу даних видно, що у величезних кількостях ***Set*** кожен елемент зберігається тільки в одному екземплярі, а різні реалізації ***Set*** використовують різний порядок зберігання елементів. В ***HashSet*** порядок елементів визначається за досить складного алгоритму - поки досить знати, що цей алгоритм забезпечує мінімальний час вибірки елементів, але порядок проходження елементів на перший погляд виглядає хаотично. Якщо порядок зберігання для вас важливий, використовуйте контейнер ***TreeSet***, в якому об'єкти зберігаються відсортованими по зростанню в порядку порівняння, або ***LinkedHashSet*** зі зберіганням елементів в порядку додавання. 

Карта (***Map***) дозволяє шукати об'єкти по ключу, як нескладна база даних. Об'єкт, асоційований з ключем, називається значенням. (Карти також називають асоціативними масивами.) У нашому прикладі використовуються три основні різновиди ***Map: HashMap, TreeMap*** і ***LinkedHashMap***. Як і ***HashSet, HashMap*** забезпечує максимальну швидкість вибірки, а порядок зберігання його елементів не очевидний. ***TreeMap*** зберігає ключі відсортованими по зростанню, a ***LinkedHashMap*** зберігає ключі в порядку вставки, але забезпечує швидкість пошуку ***HashMap***.
 
# List
 
Контейнери ***List*** гарантують певний порядок елементів. Інтерфейс ***List*** доповнює ***Collection*** декількома методами, що забезпечують вставку і видалення елементів в середині списку. 

Існує два основні різновиди ***List***:
- **Базовий контейнер ArrayList**, оптимізований для довільного доступу до елементів, але з відносно повільними операціями вставки (видалення) елементів в середині списку.
- **Контейнер LinkedList**, оптимізований для послідовного доступу, з швидкими операціями вставки (видалення) в середині списку. Довільний доступ до елементів ***LinkedList*** виконується відносно повільно, але за кількістю можливостей він перевершує ***ArrayList***.

У наступному прикладі використовується бібліотека ***typenfo.pets*** з глави «Інформація про тип». Вона містить ієрархію класів домашніх тварин ***Pet***, а також ряд допоміжних засобів для випадкової побудови об'єктів ***Pet***. Поки досить знати, що (1) бібліотека містить клас ***Pet*** і похідні типи, і (2) статичний метод ***Pets.arrayList()*** повертає контейнер ***ArrayList***, заповнений випадково вибраними об'єктами ***Pet***.
 
``` java
//: holding/ListFeatures.java 
import typeinfo.pets.*; 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class ListFeatures { 
  public static void main(String[] args) { 
    Random rand = new Random(47); 
    List<Pet> pets = Pets.arrayList(7); 
    print("1: " + pets); 
    Hamster h = new Hamster(); 
    pets.add(h); // Automatically resizes 
    print("2: " + pets); 
    print("3: " + pets.contains(h)); 
    pets.remove(h); // Remove by object 
    Pet p = pets.get(2); 
    print("4: " +  p + " " + pets.indexOf(p)); 
    Pet cymric = new Cymric(); 
    print("5: " + pets.indexOf(cymric)); 
    print("6: " + pets.remove(cymric)); 
    // Must be the exact object: 
    print("7: " + pets.remove(p)); 
    print("8: " + pets); 
    pets.add(3, new Mouse()); // Insert at an index 
    print("9: " + pets); 
    List<Pet> sub = pets.subList(1, 4); 
    print("subList: " + sub); 
    print("10: " + pets.containsAll(sub)); 
    Collections.sort(sub); // In-place sort 
    print("sorted subList: " + sub); 
    // Order is not important in containsAll(): 
    print("11: " + pets.containsAll(sub)); 
    Collections.shuffle(sub, rand); // Mix it up 
    print("shuffled subList: " + sub); 
    print("12: " + pets.containsAll(sub)); 
    List<Pet> copy = new ArrayList<Pet>(pets); 
    sub = Arrays.asList(pets.get(1), pets.get(4)); 
    print("sub: " + sub); 
    copy.retainAll(sub); 
    print("13: " + copy); 
    copy = new ArrayList<Pet>(pets); // Get a fresh copy 
    copy.remove(2); // Remove by index 
    print("14: " + copy); 
    copy.removeAll(sub); // Only removes exact objects 
    print("15: " + copy); 
    copy.set(1, new Mouse()); // Replace an element 
    print("16: " + copy); 
    copy.addAll(2, sub); // Insert a list in the middle 
    print("17: " + copy); 
    print("18: " + pets.isEmpty()); 
    pets.clear(); // Remove all elements 
    print("19: " + pets); 
    print("20: " + pets.isEmpty()); 
    pets.addAll(Pets.arrayList(4)); 
    print("21: " + pets); 
    Object[] o = pets.toArray(); 
    print("22: " + o[3]); 
    Pet[] pa = pets.toArray(new Pet[0]); 
    print("23: " + pa[3].id()); 
  } 
} /* Output: 
1: [Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug] 
2: [Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Hamster] 
3: true 
4: Cymric 2 
5: -1 
6: false 
7: true 
8: [Rat, Manx, Mutt, Pug, Cymric, Pug] 
9: [Rat, Manx, Mutt, Mouse, Pug, Cymric, Pug] 
subList: [Manx, Mutt, Mouse] 
10: true 
sorted subList: [Manx, Mouse, Mutt] 
11: true 
shuffled subList: [Mouse, Manx, Mutt] 
12: true 
sub: [Mouse, Pug] 
13: [Mouse, Pug] 
14: [Rat, Mouse, Mutt, Pug, Cymric, Pug] 
15: [Rat, Mutt, Cymric, Pug] 
16: [Rat, Mouse, Cymric, Pug] 
17: [Rat, Mouse, Mouse, Pug, Cymric, Pug] 
18: false 
19: [] 
20: true 
21: [Manx, Cymric, Rat, EgyptianMau] 
22: EgyptianMau 
23: 14 
*///:~ 
```
 
Рядки виведення пронумеровані, щоб вам було зручніше пов'язувати результат з вихідним кодом. У першому рядку виводиться вихідний контейнер ***List*** з об'єктами ***Pets***. На відміну від масивів, ***List*** підтримує додавання і видалення елементів зі зміною розмірів списку. Результат додавання ***Hamster*** видно в рядку 2: об'єкт з'являється в кінці списку.
 
Метод ***contains()*** перевіряє, чи присутній об'єкт в списку. Щоб видалити об'єкт, передайте посилання на нього методу ***remove()***. Крім того, при наявності посилання на об'єкт можна дізнатися його індекс у списку за допомогою методу ***indexOf()***, як показано в рядку 4.
 
При перевірці входження елемента в ***List***, перевірці індексу елемента і видалення елемента з ***List*** по посиланню використовується метод ***equals()*** (з кореневого класу ***Object***). Всі об'єкти ***Pet*** вважаються унікальними, тому незважаючи на присутність двох об'єктів ***Cymric*** в списку, якщо я створю новий об'єкт ***Cymric*** і передам його ***indexOf()***, результат буде дорівнює *-1* (елемент не знайдений), а виклик ***remove()*** поверне ***false***. Для інших класів метод ***equals()*** може бути визначений інакше - наприклад, об'єкти ***String*** вважаються рівними в разі збігу вмісту.
 
У рядках 7 і 8 з ***List*** успішно видаляється заданий об'єкт. Рядок 9 і передуючий їй код демонструють вставку елемента в середину списку. 

Метод ***subList()*** дозволяє легко створити «зріз» з підмножини елементів списку; природно, при передачі його методу ***containsAll()*** більшого списку буде отримано істинний результат.  Також цікаво помітити, що порядок не важливий - ви можете побачити в вихідних рядках 11 і 12 що виклик ***Collections.sort()*** і ***Collections.shuffle()*** для ***sub*** не впливають на результат виклику ***containsAll().subList()***. Тим не меньше, зміни в повертаємому списку відображаються в оригінальному списку і навпаки.
 
Метод ***retainAll ()*** фактично виконує операцію «перетину множин», тобто збереження всіх елементів копії, які також присутні в ***sub***. І знову поведінку методу залежить від реалізації методу ***equals()***.
 
У рядку 14 представлений результат видалення елемента за індексом - це простіше, ніж видалення за посиланням на об'єкт, тому що вам не доведеться турбуватися про поведінку ***equals()***. Робота методу ***removeAll()*** також залежить від ***equals()***. Як підказує назва, метод видаляє з ***List*** всі об'єкти, що входять в ***List*** - аргумент.
 
Назва методу ***set()*** вибрано невдало, тому що воно збігається з ім'ям класу ***Set*** - можливо, краще було б назвати метод «*replace*», тому що він замінює елемент з заданим індексом (перший аргумент) другим аргументом.

У рядку виводу 17 показано, що для ***List*** існує перевантажений метод ***addAll()***, що вставляє новий список в середину вихідного списку (замість простого додавання в кінець методом ***addAll()***, успадкованим від ***Collection***).
 
У рядках 18-20 представлений результат виклику методів ***isEmpty()*** і ***clear()***. Рядки 22 і 23 демонструють, що будь-який об'єкт ***Collection*** можна перетворити в масив з використанням ***toArray()***. 

# Ітератори

У будь-якого контейнера повинен існувати механізм вставки і вибірки елементів. Зрештою, контейнер призначений саме для зберігання об'єктів. При роботі з ***List*** для вставки може використовуватися метод ***add()***, а для вибірки - метод ***get()*** (втім, існують і інші способи).
 
Якщо поглянути на ситуацію з більш високого рівня, виявляється проблема: щоб використовувати контейнер в програмі, необхідно знати його точний тип. Що, якщо ви почали використовувати в програмі контейнер ***List***, а потім виявили, що у вашому випадку буде зручніше застосувати той же код до ***Set***? Або якщо ви хочете написати універсальний код, який не залежить від типу контейнера і може застосовуватися до будь-якого контейнера?

З даної абстракцією добре узгоджується концепція ітератора (*iterator*). Ітератор - це об'єкт, що забезпечує переміщення по послідовності об'єктів з вибором кожного об'єкта цієї послідовності, при цьому програмісту-клієнту не треба знати, або піклуватися про покладений що є базою цієї послідовності. До того ж, ітератор зазвичай є так званим «легким» (*lightweight*) об'єктом: його створення має обходитися без помітних витрат ресурсів. Через це ітератори часто мають обмеження; наприклад, ***Iterator*** в *Java*
підтримує переміщення тільки в одному напрямку. Його можливості не так вже широкі, але з його допомогою можна зробити наступне:

- Запросити у контейнера ітератор викликом методу ***iterator()***. Отриманий ітератор готовий повернути початковий елемент послідовності при першому виклику свого методу ***next()***.
- Отримати наступний елемент послідовності викликом методу ***next()***.
- Перевірити, чи залишилися ще об'єкти в послідовності (метод ***hasNext()***).
- Видалити з послідовності останній елемент, повернутий ітератором, методом ***remove()***.

Щоб побачити ітератор в дії, ми знову скористаємося ієрархією ***Pets***:
 
``` java
//: holding/SimpleIteration.java 
import typeinfo.pets.*; 
import java.util.*; 
 
public class SimpleIteration { 
  public static void main(String[] args) { 
    List<Pet> pets = Pets.arrayList(12); 
    Iterator<Pet> it = pets.iterator(); 
    while(it.hasNext()) { 
      Pet p = it.next(); 
      System.out.print(p.id() + ":" + p + " "); 
    } 
    System.out.println(); 
    // A simpler approach, when possible: 
    for(Pet p : pets) 
      System.out.print(p.id() + ":" + p + " "); 
    System.out.println();   
    // An Iterator can also remove elements: 
    it = pets.iterator(); 
    for(int i = 0; i < 6; i++) { 
      it.next(); 
      it.remove(); 
    } 
    System.out.println(pets); 
  } 
} /* Output: 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 
10:EgyptianMau 11:Hamster 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 
10:EgyptianMau 11:Hamster 
[Pug, Manx, Cymric, Rat, EgyptianMau, Hamster] 
*///:~ 
```
 
Ми бачимо, що з ***Iterator*** можна не турбуватися про кількість елементів в послідовності. Перевірка здійснюється методами ***hasNext ()*** і ***next()***. Якщо ви просто перебирає елементи списку в одному напрямку, не намагаючись модифікувати його вміст, «синтаксис ***foreach***» забезпечує більш компактний запис. ***Iterator*** видаляє останній елемент, отриманий за допомогою ***next()***, тому перед викликом ***remove()*** необхідно викликати ***next()***. Тепер розглянемо задачу створення методу ***display()***, котрий залежить від типу контейнера:
 
``` java
//: holding/CrossContainerIteration.java 
import typeinfo.pets.*; 
import java.util.*; 
 
public class CrossContainerIteration { 
  public static void display(Iterator<Pet> it) { 
    while(it.hasNext()) { 
      Pet p = it.next(); 
      System.out.print(p.id() + ":" + p + " "); 
    } 
    System.out.println(); 
  }  
  public static void main(String[] args) { 
    ArrayList<Pet> pets = Pets.arrayList(8); 
    LinkedList<Pet> petsLL = new LinkedList<Pet>(pets); 
    HashSet<Pet> petsHS = new HashSet<Pet>(pets); 
    TreeSet<Pet> petsTS = new TreeSet<Pet>(pets); 
    display(pets.iterator()); 
    display(petsLL.iterator()); 
    display(petsHS.iterator()); 
    display(petsTS.iterator()); 
  } 
} /* Output: 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat 
5:Cymric 2:Cymric 7:Manx 1:Manx 3:Mutt 6:Pug 4:Pug 0:Rat 
*///:~ 
```
 
У методі ***display()*** відсутня інформація про тип послідовності, і в цьому проявляється справжня міць ітераторів: операція переміщення по послідовності відділяється від фактичної структури цієї послідовності. Іноді кажуть, що ітератори уніфікують доступ до контейнерів. 

# Listlterator

Listlterator - більш потужна різновид ***Iterator***, підтримувана тільки класами ***List***. Якщо ***Iterator*** підтримує переміщення тільки вперед, то ***ListIterator*** є двостороннім. Крім того, він може видавати індекси наступного і попереднього елементів по відношенню до поточної позиції ітератора в списку і замінювати останній відвіданий елемент методом ***set )***. Виклик ***listIterator()*** повертає ***ListIterator***, який вказує на початок ***List***, а для створення ітератора ***ListIterator***, спочатку встановленого на елемент з індексом ***n***, використовується виклик ***listIterator(n)***. Всі перераховані можливості продемонстровані в наступному прикладі:
 
``` java
//: holding/ListIteration.java 
import typeinfo.pets.*; 
import java.util.*; 
 
public class ListIteration { 
  public static void main(String[] args) { 
    List<Pet> pets = Pets.arrayList(8); 
    ListIterator<Pet> it = pets.listIterator(); 
    while(it.hasNext()) 
      System.out.print(it.next() + ", " + it.nextIndex() + 
        ", " + it.previousIndex() + "; "); 
    System.out.println(); 
    // Backwards: 
    while(it.hasPrevious()) 
      System.out.print(it.previous().id() + " "); 
    System.out.println(); 
    System.out.println(pets);   
    it = pets.listIterator(3); 
    while(it.hasNext()) { 
      it.next(); 
      it.set(Pets.randomPet()); 
    } 
    System.out.println(pets); 
  } 
} /* Output: 
Rat, 1, 0; Manx, 2, 1; Cymric, 3, 2; Mutt, 4, 3; Pug, 5, 4; Cymric, 6, 
5; Pug, 7, 6; Manx, 8, 7; 
7 6 5 4 3 2 1 0 
[Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Manx] 
[Rat, Manx, Cymric, Cymric, Rat, EgyptianMau, Hamster, EgyptianMau] 
*///:~ 
```
 
Метод ***Pets.randomPet()*** використовується для заміни всіх об'єктів ***Pet*** в списку, починаючи з позиції 3 і далі. 

# LinkedList

***LinkedList*** теж реалізує базовий інтерфейс ***List***, як і ***ArrayList***, але виконує деякі операції (наприклад, вставку і видалення в середині списку) більш ефективно, ніж ***ArrayList***. І навпаки, операції довільного доступу виконуються ним з меншою ефективністю. Клас ***LinkedList*** також містить методи, що дозволяють використовувати його в якості стека, черги (***Queue***), або двосторонньої черги (дека). Деякі з цих методів є псевдонімами, або модифікаціями для отримання імен, більш знайомих в контексті деякого використання. Наприклад, методи ***getFirst()*** і ***element()*** ідентичні - вони повертають початок (перший елемент) списку без його видалення і генерують виключення ***NoSuchElementException*** для порожнього списку. Метод ***peek()*** являє собою невелику модифікацію цих двох методів: він повертає ***null*** для порожнього списку. Метод ***addFirst()*** вставляє елемент в початок списку. Метод ***offer()*** робить те ж, що ***add()*** і ***addLast()*** - він додає елемент в кінець списку. Метод ***removeLast()*** видаляє і повертає останній елемент списку. Наступний приклад демонструє схожі і відмінні аспекти цих методів:
 
``` java
//: holding/LinkedListFeatures.java 
import typeinfo.pets.*; 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class LinkedListFeatures { 
  public static void main(String[] args) { 
    LinkedList<Pet> pets = 
      new LinkedList<Pet>(Pets.arrayList(5)); 
    print(pets); 
    // Identical: 
    print("pets.getFirst(): " + pets.getFirst()); 
    print("pets.element(): " + pets.element()); 
    // Only differs in empty-list behavior: 
    print("pets.peek(): " + pets.peek()); 
    // Identical; remove and return the first element: 
    print("pets.remove(): " + pets.remove()); 
    print("pets.removeFirst(): " + pets.removeFirst()); 
    // Only differs in empty-list behavior: 
    print("pets.poll(): " + pets.poll()); 
    print(pets); 
    pets.addFirst(new Rat()); 
    print("After addFirst(): " + pets); 
    pets.offer(Pets.randomPet()); 
    print("After offer(): " + pets); 
    pets.add(Pets.randomPet()); 
    print("After add(): " + pets); 
    pets.addLast(new Hamster()); 
    print("After addLast(): " + pets); 
    print("pets.removeLast(): " + pets.removeLast()); 
  } 
} /* Output: 
[Rat, Manx, Cymric, Mutt, Pug] 
pets.getFirst(): Rat 
pets.element(): Rat 
pets.peek(): Rat 
pets.remove(): Rat 
pets.removeFirst(): Manx 
pets.poll(): Cymric 
[Mutt, Pug] 
After addFirst(): [Rat, Mutt, Pug] 
After offer(): [Rat, Mutt, Pug, Cymric] 
After add(): [Rat, Mutt, Pug, Cymric, Pug] 
After addLast(): [Rat, Mutt, Pug, Cymric, Pug, Hamster] 
pets.removeLast(): Hamster 
*///:~ 
```

Результат ***Pets.arrayList()*** передається конструктору ***LinkedList*** для заповнення. Придивившись до інтерфейсу ***Queue***, ви знайдете в ньому методи ***element()***, ***offer()***, ***peek()***, ***poll()*** і ***remove()***, додані в ***LinkedList*** для використання в реалізації черги (див. далі). 

# Стек

Стек часто називають контейнером, що працює за принципом «першим увійшов, останнім вийшов» (*LIFO*). Тобто елемент, останнім занесений в стек, буде першим, отриманим при вилученні з стека. У класі ***LinkedList*** є методи, що безпосередньо реалізують функціональність стека, тому ви просто використовуєте ***LinkedList***, не створюєте для стека новий клас. Втім, іноді окремий клас для контейнера-стека краще справляється із завданням:
 
``` java
//: net/mindview/util/Stack.java 
// Making a stack from a LinkedList. 
package net.mindview.util; 
import java.util.LinkedList; 
 
public class Stack<T> { 
  private LinkedList<T> storage = new LinkedList<T>(); 
  public void push(T v) { storage.addFirst(v); } 
  public T peek() { return storage.getFirst(); } 
  public T pop() { return storage.removeFirst(); } 
  public boolean empty() { return storage.isEmpty(); } 
  public String toString() { return storage.toString(); } 
} ///:~ 
```
 
Це найпростіший приклад визначення класу з використанням параметризації. Суфікс ***<Т>*** після імені класу повідомляє компілятору, що тип є параметризованим по типу ***Т*** - при використанні класу на місце ***Т*** буде підставлений фактичний тип. Фактично таке визначення означає: «Ми визначаємо клас ***Stack*** для зберігання об'єктів типу ***Т***». ***Stack*** реалізується на базі ***LinkedList***, також призначеного для зберігання типу ***Т***. Зверніть увагу: метод ***push()*** отримує об'єкт типу ***Т***, а методи ***peek()*** і ***рор()*** повертають об'єкт типу ***Т***. Метод ***peek()*** повертає верхній елемент без вилучення з стека, а метод ***рор()*** видаляє і повертає верхній елемент. Простий приклад використання нового класу ***Stack***:
 
``` java
//: holding/StackTest.java 
import net.mindview.util.*; 
 
public class StackTest { 
  public static void main(String[] args) { 
    Stack<String> stack = new Stack<String>(); 
    for(String s : "My dog has fleas".split(" ")) 
      stack.push(s); 
    while(!stack.empty()) 
      System.out.print(stack.pop() + " "); 
  } 
} /* Output: 
fleas has dog My 
*///:~ 
```
 
Якщо ви хочете використовувати клас ***Stack*** в своєму коді, вам доведеться або повністю вказати пакет, або змінити ім'я класу при створенні об'єкта; в іншому випадку, швидше за все, виникне конфлікт з класом ***Stack*** з пакета ***java.util***. Приклад використання назв пакунків при імпорту ***java.util.\**** В попередньому прикладі:
 
``` java
//: holding/StackCollision.java 
import net.mindview.util.*; 
 
public class StackCollision { 
  public static void main(String[] args) { 
    net.mindview.util.Stack<String> stack = 
      new net.mindview.util.Stack<String>(); 
    for(String s : "My dog has fleas".split(" ")) 
      stack.push(s); 
    while(!stack.empty()) 
      System.out.print(stack.pop() + " "); 
    System.out.println(); 
    java.util.Stack<String> stack2 = 
      new java.util.Stack<String>(); 
    for(String s : "My dog has fleas".split(" ")) 
      stack2.push(s); 
    while(!stack2.empty()) 
      System.out.print(stack2.pop() + " "); 
  } 
} /* Output: 
fleas has dog My 
fleas has dog My 
*///:~ 
```
 
В ***java.util*** немає загального інтерфейсу ***Stack*** - ймовірно, через те, що ім'я було задіяне в вихідній, невдало спроектованій версії ***java.util.Stack*** для *Java* 1.0. Хоча клас ***java.util.Stack*** існує, ***LinkedList*** забезпечує більш якісну реалізацію стека, і рішення ***net.mindview.util.Stack*** найбільш прийнятне.
 
# Set (безліч)
 
В ***Set*** кожне значення може зберігатися тільки в одному екземплярі. Спроби додати новий екземпляр еквівалентного об'єкта блокуються. ***Set*** часто використовується для перевірки приналежності, щоб ви могли легко перевірити, чи належить об'єкт заданій множині. Отже, найважливішою операцією ***Set*** є операція пошуку, тому на практиці зазвичай вибирається реалізація ***HashSet***, оптимізована для швидкого пошуку.
 
***Set*** має такий же інтерфейс, що і ***Collection***. По суті, ***Set*** і є ***Collection***, але має дещо іншу поведінку (до речі, ідеальний приклад використання успадкування та поліморфізму: вираження різних концепцій поведінки). Приклад використання ***HashSet*** з об'єктами ***Integer***:
 
``` java
//: holding/SetOfInteger.java 
import java.util.*; 
 
public class SetOfInteger { 
  public static void main(String[] args) { 
    Random rand = new Random(47); 
    Set<Integer> intset = new HashSet<Integer>(); 
    for(int i = 0; i < 10000; i++) 
      intset.add(rand.nextInt(30)); 
    System.out.println(intset); 
  } 
} /* Output: 
[15, 8, 23, 16, 7, 22, 9, 21, 6, 1, 29, 14, 24, 4, 19, 26, 11, 18, 3, 
12, 27, 17, 2, 13, 28, 20, 25, 10, 5, 0] 
*///:~ 
```
 
В ***Set*** включаються десять тисяч випадкових чисел від *0* до *29*; природно, числа повинні багаторазово повторюватися. Але при цьому ми бачимо, що в результаті кожне число присутній тільки в одному екземплярі.
 
Також зверніть увагу на непередбачуваний порядок проходження чисел у виводі. Це пояснюється тим, що ***HashSet*** використовує хешування для прискорення вибірки. Порядок, підтримуваний ***HashSet***, відрізняється від порядку ***TreeSet***, або ***LinkedHashSet***, оскільки кожна реалізація впорядковує елементи по-своєму. Якщо ви хочете, щоб результат був впорядкований, скористайтеся ***TreeSet*** замість ***HashSet***:
 
``` java
//: holding/SortedSetOfInteger.java 
import java.util.*; 
 
public class SortedSetOfInteger { 
  public static void main(String[] args) { 
    Random rand = new Random(47); 
    SortedSet<Integer> intset = new TreeSet<Integer>(); 
    for(int i = 0; i < 10000; i++) 
      intset.add(rand.nextInt(30)); 
    System.out.println(intset); 
  } 
} /* Output: 
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 
20, 21, 22, 23, 24, 25, 26, 27, 28, 29] 
*///:~ 
```
 
Однією з найбільш поширених операцій з множинами є перевірка приналежності методом ***contains()***, але існують і інші операції, які нагадають вам діаграми Венна зі шкільного курсу:
 
``` java
//: holding/SetOperations.java 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class SetOperations { 
   public static void main(String[] args) { 
    Set<String> set1 = new HashSet<String>(); 
    Collections.addAll(set1, 
      "A B C D E F G H I J K L".split(" ")); 
    set1.add("M"); 
    print("H: " + set1.contains("H")); 
    print("N: " + set1.contains("N")); 
    Set<String> set2 = new HashSet<String>(); 
    Collections.addAll(set2, "H I J K L".split(" ")); 
    print("set2 in set1: " + set1.containsAll(set2)); 
    set1.remove("H"); 
    print("set1: " + set1); 
    print("set2 in set1: " + set1.containsAll(set2)); 
    set1.removeAll(set2); 
    print("set2 removed from set1: " + set1); 
    Collections.addAll(set1, "X Y Z".split(" ")); 
    print("‘X Y Z’ added to set1: " + set1); 
  } 
} /* Output: 
H: true 
N: false 
set2 in set1: true 
set1: [D, K, C, B, L, G, I, M, A, F, J, E] 
set2 in set1: false 
set2 removed from set1: [D, C, B, G, M, A, F, E] 
‘X Y Z’ added to set1: [Z, D, C, B, G, M, A, F, Y, X, E] 
*///:~ 
```
 
Імена методів говорять за себе. Інформацію про інші методи ***Set*** можна знайти в документації *JDK*.

Producing a list of unique elements can be quite useful. For example, suppose you’d like to 
Генерація списку унікальних елементів може бути досить корисним. Наприклад, припустимо ви б хотіли вивести список всіх слів в файлі SetOperations.java, що наведений вище. Використовуючи утиліту net.mindview.TextFile, що буде описана пізніше, ви можете відкрити і читати файл в ***Set***:

```java
//: holding/UniqueWords.java 
import java.util.*; 
import net.mindview.util.*; 
 
public class UniqueWords { 
  public static void main(String[] args) { 
    Set<String> words = new TreeSet<String>( 
      new TextFile("SetOperations.java", "\\W+")); 
    System.out.println(words); 
  } 
} /* Output: 
[A, B, C, Collections, D, E, F, G, H, HashSet, I, J, K, L, M, N, Output, 
Print, Set, SetOperations, String, X, Y, Z, add, addAll, added, args, 
class, contains, containsAll, false, from, holding, import, in, java, 
main, mindview, net, new, print, public, remove, removeAll, removed, 
set1, set2, split, static, to, true, util, void] 
*///:~ 
```

***TextFile*** успадкований від List<String>. Конструктор ***TextFile*** відкриває файл і розбиває на слова згідно регулярному виразу "\\W+", який означає "один, або більше літер" (регулярні вирази пояснені в розділі Рядки (***Strings***). Результат передається конструктору ***TreeSet***, який додає вміст ***List*** до себе. Так як це тип ***TreeSet***, результат відсортований. В цьому випадку список відсортований лексографічно, отже великі і малі букви знаходяться в різних групах. Якщо ви хочете відсортувати по алфавіту, ви можете передати компаратор String.CASE_INSENSITIVE_ORDER (компаратор являється об'єктом, що встановлює порядок) до конструктура TreeSet: 

``` java
//: holding/UniqueWordsAlphabetic.java 
// Producing an alphabetic listing. 
import java.util.*; 
import net.mindview.util.*; 
 
public class UniqueWordsAlphabetic { 
  public static void main(String[] args) { 
    Set<String> words = 
      new TreeSet<String>(String.CASE_INSENSITIVE_ORDER); 
    words.addAll( 
      new TextFile("SetOperations.java", "\\W+")); 
    System.out.println(words); 
  } 
} /* Output: 
[A, add, addAll, added, args, B, C, class, Collections, contains, 
containsAll, D, E, F, false, from, G, H, HashSet, holding, I, import, 
in, J, java, K, L, M, main, mindview, N, net, new, Output, Print, 
public, remove, removeAll, removed, Set, set1, set2, SetOperations, 
split, static, String, to, true, util, void, X, Y, Z] 
*///:~ 
```

Компаратори (***comparator***) будуть роз'яснені в деталях в розділі присв'ченному масивам (***Arrays***).
 
# Map
 
Можливість відображення одних об'єктів на інші (асоціація) надзвичайно корисна при вирішенні широкого класу задач програмування. Як приклад розглянемо програму, яка аналізує якість розподілу класу *Java* ***Random***. В ідеалі клас ***Random*** повинен видавати абсолютно рівномірний розподіл чисел, але щоб переконатися в цьому, необхідно згенерувати велику кількість випадкових чисел і підрахувати їх кількість в різних інтервалах. ***Maps*** спрощують цю задачу: ключем в даному випадку є число, сгенероване за допомогою ***Random***, а значенням - кількість його входжень:
 
``` java
//: holding/Statistics.java
// Простий приклад використання HashMap
import java.util.*; 
 
public class Statistics { 
  public static void main(String[] args) { 
    Random rand = new Random(47); 
    Map<Integer,Integer> m = 
      new HashMap<Integer,Integer>(); 
    for(int i = 0; i < 10000; i++) { 
      // Produce a number between 0 and 20: 
      int r = rand.nextInt(20); 
      Integer freq = m.get(r); 
      m.put(r, freq == null ? 1 : freq + 1); 
    } 
    System.out.println(m); 
  } 
} /* Output:
{15=497, 4=481, 19=464, 8=468, 11=531, 16=533, 18=478, 3=508, 7=471, 
12=521, 17=509, 2=489, 13=506, 9=549, 6=519, 1=502, 14=477, 10=513, 
5=503, 0=481} 
*///:~ 
```
 
В ***main()*** механізм автоматичної упаковки перетворює випадково згенероване ціле число на посилання на Integer, яка може використовуватися з ***HashMap*** (контейнери не можуть використовуватися для зберігання примітивів).
 
Метод ***get()*** повертає ***null***, якщо елемент відсутній в контейнері (тобто якщо число було згенеровано вперше). В іншому випадку метод ***get()*** повертає значення ***Integer***, пов'язане з ключем, і останній збільшується на *1* (автоматична упаковка знову спрощує обчислення, але в дійсності при цьому виконуються перетворення до ***Integer*** і назад).
 
Наступний приклад демонструє пошук об'єктів ***Pet*** по строковому опису ***String***. Він також показує, як перевірити присутність деякого ключа, або значення в ***Map*** методами ***containsKey()*** і ***containsValue()***:
 
``` java
//: holding/PetMap.java 
import typeinfo.pets.*; 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class PetMap { 
  public static void main(String[] args) { 
    Map<String,Pet> petMap = new HashMap<String,Pet>(); 
    petMap.put("My Cat", new Cat("Molly")); 
    petMap.put("My Dog", new Dog("Ginger")); 
    petMap.put("My Hamster", new Hamster("Bosco")); 
    print(petMap); 
    Pet dog = petMap.get("My Dog"); 
    print(dog); 
    print(petMap.containsKey("My Dog")); 
    print(petMap.containsValue(dog)); 
  } 
} /* Output: 
{My Cat=Cat Molly, My Hamster=Hamster Bosco, My Dog=Dog Ginger} 
Dog Ginger 
true 
true 
*///:~ 
```
 
***Map***, за аналогією з масивами і ***Collection***, легко розширюються до декількох вимірювань; досить створити ***Map***
зі значеннями типу ***Map*** (причому значеннями цих ***Map*** можуть бути інші контейнери, і навіть інші ***Map***). Контейнери легко комбінуються один з одним, що дозволяє швидко створювати складні структури даних. Наприклад, якщо нам буде потрібно зберегти інформацію про власників відразу декількох домашніх тварин, для цього буде достатньо створити контейнер ***Map<Person, List<Pet>>***:
 
``` java
//: holding/MapOfList.java 
package holding; 
import typeinfo.pets.*; 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class MapOfList { 
  public static Map<Person, List<? extends Pet>> 
    petPeople = new HashMap<Person, List<? extends Pet>>(); 
  static { 
    petPeople.put(new Person("Dawn"), 
      Arrays.asList(new Cymric("Molly"),new Mutt("Spot"))); 
    petPeople.put(new Person("Kate"), 
      Arrays.asList(new Cat("Shackleton"), 
        new Cat("Elsie May"), new Dog("Margrett")));
    petPeople.put(new Person("Marilyn"), 
      Arrays.asList( 
       new Pug("Louie aka Louis Snorkelstein Dupree"), 
       new Cat("Stanford aka Stinky el Negro"), 
       new Cat("Pinkola")));   
    petPeople.put(new Person("Luke"), 
      Arrays.asList(new Rat("Fuzzy"), new Rat("Fizzy"))); 
    petPeople.put(new Person("Isaac"), 
      Arrays.asList(new Rat("Freckly"))); 
  } 
  public static void main(String[] args) { 
    print("People: " + petPeople.keySet()); 
    print("Pets: " + petPeople.values()); 
    for(Person person : petPeople.keySet()) { 
      print(person + " has:"); 
      for(Pet pet : petPeople.get(person)) 
        print("    " + pet); 
    } 
  } 
} /* Output:   
People: [Person Luke, Person Marilyn, Person Isaac, Person Dawn, Person 
Kate] 
Pets: [[Rat Fuzzy, Rat Fizzy], [Pug Louie aka Louis Snorkelstein Dupree, 
Cat Stanford aka Stinky el Negro, Cat Pinkola], [Rat Freckly], [Cymric 
Molly, Mutt Spot], [Cat Shackleton, Cat Elsie May, Dog Margrett]] 
Person Luke has: 
    Rat Fuzzy 
    Rat Fizzy 
Person Marilyn has: 
    Pug Louie aka Louis Snorkelstein Dupree 
    Cat Stanford aka Stinky el Negro 
    Cat Pinkola 
Person Isaac has: 
    Rat Freckly 
Person Dawn has: 
    Cymric Molly 
    Mutt Spot 
Person Kate has: 
    Cat Shackleton 
    Cat Elsie May 
    Dog Margrett 
*///:~     
```
 
***Map*** може повернути безліч (***Set***) з ключами, колекцію (***Collection***) значень, або безліч (***Set***) всіх пар «ключ-значення». Метод ***keySet()*** створює безліч всіх ключів, яке потім використовується в синтаксисі ***foreach*** для перебору ***Map***. 

# Черга (Queue)

Черга (***queue***) зазвичай являє собою контейнер, що працює за принципом «першим увійшов, першим вийшов» (*FIFO*). Інакше кажучи, елементи заносяться в чергу з одного «кінця» і витягуються з іншого в порядку їх надходження. Черги часто застосовуються для реалізації надійної передачі об'єктів між різними областями програми. Клас ***LinkedList*** містить методи, що підтримують поведінку черги, і реалізує інтерфейс ***Queue***, тому ***LinkedList*** може використовуватися в якості реалізації ***Queue***. У наступному прикладі ***LinkedList*** підвищується висхідним перетворенням до ***Queue***:
 
``` java
//: holding/QueueDemo.java 
// Висхідне перетворення LinkedList в Queue
import java.util.*; 
 
public class QueueDemo { 
  public static void printQ(Queue queue) { 
    while(queue.peek() != null) 
      System.out.print(queue.remove() + " "); 
    System.out.println(); 
  } 
  public static void main(String[] args) { 
    Queue<Integer> queue = new LinkedList<Integer>(); 
    Random rand = new Random(47); 
    for(int i = 0; i < 10; i++)
      queue.offer(rand.nextInt(i + 10)); 
    printQ(queue); 
    Queue<Character> qc = new LinkedList<Character>(); 
    for(char c : "Brontosaurus".toCharArray()) 
      qc.offer(c); 
    printQ(qc); 
  } 
} /* Output: 
8 1 1 1 5 14 3 1 0 1 
B r o n t o s a u r u s 
*///:~     	
```
 
Метод ***offer()***, один з методів ***Queue***, вставляє елемент в кінець черги, а якщо вставка неможлива - повертає ***false***. Методи ***peek()*** і ***element()*** повертають початковий елемент без його видалення з черги, але ***peek()*** для порожної черги повертає ***null***, a ***element()*** генерує виключення ***NoSuchElementException***. Методи ***poll()*** і ***remove()*** видаляють і повертають початковий елемент черги, але ***poll()*** для порожної черги повертає ***null***, a ***remove()*** генерує ***NoSuchElementException***. Автоматична упаковка перетворює результат ***int*** виклику ***nextInt()*** в об'єкт ***Integer***, необхідний для ***queue***, a ***char c*** - в об'єкт ***Character***, необхідний для ***qc***. Інтерфейс ***Queue*** звужує доступ до методів ***LinkedList*** так, що доступними залишаються тільки відповідні методи і у користувача залишається менше можливостей для виклику методів ***LinkedList*** (звичайно, ***Queue*** можна перетворити назад в ***LinkedList***, але це створює додаткові труднощі).
 
# Пріоритетна черга (PriorityQueue)
 
Принцип FIFO описує найбільш типову організацію черги. Саме організація черги визначає, який елемент буде наступним для заданого стану черги. Правило FIFO означає, що наступним елементом буде той, який найдовше перебуває в черзі.
 
У пріоритетній черзі наступним елементом вважається елемент, що має найвищий пріоритет. Наприклад, в аеропорту пасажира, літак якого скоро полетить, можуть пропустити без черги. У системах обробки повідомлень деякі повідомлення можуть бути важливішою за інші і повинні оброблятися якнайшвидше, незалежно від моменту їх надходження. Параметризований клас ***PriorityQueue*** був доданий в *Java* *SE5*
як механізм автоматичної реалізації цієї поведінки.
 
При поміщенні об'єкта в ***PriorityQueue*** викликом ***offer()*** об'єкт сортується в черзі. За замовчуванням використовується природний порядок поміщення об'єктів в чергу, однак ви можете змінити його, створивши власний компаратор (***Comparator***). ***PriorityQueue*** гарантує, що при виклику ***peek(), poll()***, або ***remove ()*** ви отримаєте елемент з найвищим пріоритетом.
 
Створення пріоритетної черги для вбудованих типів - ***Integer, String, Character*** і т. д. - є тривіальною справою. У наступному прикладі використовуються ті ж значення, що і в попередньому, але ***PriorityQueue*** видає їх в іншому порядку:
 
``` java
//: holding/PriorityQueueDemo.java 
import java.util.*; 
 
public class PriorityQueueDemo { 
  public static void main(String[] args) { 
    PriorityQueue<Integer> priorityQueue = 
      new PriorityQueue<Integer>(); 
    Random rand = new Random(47); 
    for(int i = 0; i < 10; i++) 
      priorityQueue.offer(rand.nextInt(i + 10)); 
    QueueDemo.printQ(priorityQueue); 
 
    List<Integer> ints = Arrays.asList(25, 22, 20, 
      18, 14, 9, 3, 1, 1, 2, 3, 9, 14, 18, 21, 23, 25); 
    priorityQueue = new PriorityQueue<Integer>(ints); 
    QueueDemo.printQ(priorityQueue); 
    priorityQueue = new PriorityQueue<Integer>( 
        ints.size(), Collections.reverseOrder()); 
    priorityQueue.addAll(ints); 
    QueueDemo.printQ(priorityQueue); 
 
    String fact = "EDUCATION SHOULD ESCHEW OBFUSCATION"; 
    List<String> strings = Arrays.asList(fact.split("")); 
    PriorityQueue<String> stringPQ = 
      new PriorityQueue<String>(strings); 
    QueueDemo.printQ(stringPQ); 
    stringPQ = new PriorityQueue<String>( 
      strings.size(), Collections.reverseOrder()); 
    stringPQ.addAll(strings); 
    QueueDemo.printQ(stringPQ); 
 
    Set<Character> charSet = new HashSet<Character>(); 
    for(char c : fact.toCharArray()) 
      charSet.add(c); // Autoboxing 
    PriorityQueue<Character> characterPQ = 
      new PriorityQueue<Character>(charSet); 
    QueueDemo.printQ(characterPQ); 
  } 
} /* Output: 
0 1 1 1 1 1 3 5 8 14 
1 1 2 3 3 9 9 14 14 18 18 20 21 22 23 25 25 
25 25 23 22 21 20 18 18 14 14 9 9 3 3 2 1 1 
       A A B C C C D D E E E F H H I I L N N O O O O S S S T T U U U W 
W U U U T T S S S O O O O N N L I I H H F E E E D D C C C B A A 
  A B C D E F H I L N O S T U W 
*///:~ 
```
 
Ми бачимо, що дублікати дозволені, а менші значення мають більш високий пріоритет. Щоб показати, як змінити порядок елементів за допомогою власного об'єкта ***Comparator***, при третьому виклику конструктора ***PriorityQueue<Integer>*** і другому -
***PriorityQueue<String>*** використовується ***Comparator*** зі зворотнім сортуванням, отриманий викликом ***Collections.reverseOrder ()*** (одне з нововведень *Java* *SE5*). 

В останній частині додається ***HashSet*** для уникнення дублікатів ***Character*** - просто для того, щоб приклад був трохи більш цікавим. ***Integer, String і Character*** спочатку працюють з ***PriorityQueue***, тому що вони мають «вбудоване» природне упорядкування. Якщо ви хочете використовувати власний клас з ***PriorityQueue***, включіть додаткову реалізацію природного упорядкування, або власний об'єкт ***Comparator***. 

# Collection і Iterator

***Collection*** - кореневої інтерфейс, що описує загальну функціональність всіх послідовних контейнерів. Його можна розглядати як «вторинний інтерфейс», що з'явився внаслідок подібності між іншими інтерфейсами. Крім того, клас ***java.util.AbstractCollection*** надає реалізацію ***Collection*** за замовчуванням, тому ви можете створити новий підтип ***AbstractCollection*** без надмірного дублювання коду.
 
Один з аргументів на користь інтерфейсів полягає в тому, що вони дозволяють створювати більш універсальний код. Код, написаний для інтерфейсу, а не для його реалізації, може бути застосований до більш широкого кола об'єктів. Таким чином, якщо я пишу метод, якому при виклику передається ***Collection***, цей метод буде працювати з будь-яким типом, які реалізують ***Collection***, - отже, якщо новий клас реалізує ***Collection***, він буде сумісний з моїм методом. Однак цікаво помітити, що стандартна бібліотека *C++* не має загального базового класу для своїх контейнерів - вся спільність контейнерів забезпечується ітераторами. Здавалося б, в *Java* буде логічно взяти приклад з *C++* і виражать схожість між контейнерами за допомогою ітераторів, а не ***Collection***. Проте ці два підходи взаємопов'язані, тому що реалізація ***Collection*** також означає підтримку методу ***iterator()***:
 
``` java
//: holding/InterfaceVsIterator.java 
import typeinfo.pets.*; 
import java.util.*; 
 
public class InterfaceVsIterator { 
  public static void display(Iterator<Pet> it) { 
    while(it.hasNext()) { 
      Pet p = it.next(); 
      System.out.print(p.id() + ":" + p + " "); 
    } 
    System.out.println(); 
  } 
  public static void display(Collection<Pet> pets) { 
    for(Pet p : pets) 
      System.out.print(p.id() + ":" + p + " "); 
    System.out.println(); 
  }  
  public static void main(String[] args) { 
    List<Pet> petList = Pets.arrayList(8); 
    Set<Pet> petSet = new HashSet<Pet>(petList); 
    Map<String,Pet> petMap = 
      new LinkedHashMap<String,Pet>(); 
    String[] names = ("Ralph, Eric, Robin, Lacey, " + 
      "Britney, Sam, Spot, Fluffy").split(", "); 
    for(int i = 0; i < names.length; i++) 
      petMap.put(names[i], petList.get(i)); 
    display(petList); 
    display(petSet); 
    display(petList.iterator()); 
    display(petSet.iterator()); 
    System.out.println(petMap); 
    System.out.println(petMap.keySet()); 
    display(petMap.values()); 
    display(petMap.values().iterator()); 
  }  
} /* Output: 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat 
{Ralph=Rat, Eric=Manx, Robin=Cymric, Lacey=Mutt, Britney=Pug, 
Sam=Cymric, Spot=Pug, Fluffy=Manx} 
[Ralph, Eric, Robin, Lacey, Britney, Sam, Spot, Fluffy] 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
*///:~       
```
 
Обидві версії ***display()*** працюють як з об'єктами ***Map***, так і з підтипами ***Collection***; при цьому як ***Collection***, так і ***Iterator*** ізолюють методи ***display()*** від знання конкретної реалізації використовуваного контейнера. В даному випадку обидва рішення приблизно рівноцінні. Використання ***Iterator*** стає кращим при реалізації стороннього класу, для якого реалізація інтерфейсу ***Collection*** утруднена, або небажана. Наприклад, якщо ми створюємо реалізацію ***Collection*** успадкуванням від класу, що містить об'єкти ***Pet***, нам доведеться реалізувати всі методи ***Collection***, навіть якщо вони не будуть використовуватися в методі ***display()***. Хоча проблема легко вирішується успадкуванням від ***AbstractCollection***, вам все одно доведеться реалізувати ***iterator()*** разом з ***size()***, щоб надати методи, які не реалізовані ***AbstractCollection***, але використовувані іншими методами ***AbstractCollection***:
 
``` java
//: holding/CollectionSequence.java 
import typeinfo.pets.*; 
import java.util.*; 
 
public class CollectionSequence 
extends AbstractCollection<Pet> { 
  private Pet[] pets = Pets.createArray(8); 
  public int size() { return pets.length; } 
  public Iterator<Pet> iterator() { 
    return new Iterator<Pet>() { 
      private int index = 0; 
      public boolean hasNext() { 
        return index < pets.length; 
      } 
      public Pet next() { return pets[index++]; } 
      public void remove() { // Not implemented 
        throw new UnsupportedOperationException(); 
      } 
    }; 
  }  
  public static void main(String[] args) { 
    CollectionSequence c = new CollectionSequence(); 
    InterfaceVsIterator.display(c); 
    InterfaceVsIterator.display(c.iterator()); 
  } 
} /* Output: 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
*///:~       
```
 
Метод ***remove()*** є необов'язковою операцією. У нашому прикладі реалізовувати його не потрібно, і в разі виклику він генерує виключення. З наведеного прикладу видно, що при реалізації ***Collection*** ви реалізуєте ***iterator()***, а проста окрема реалізація ***iterator()*** вимагає трохи менших зусиль, ніж успадкування від ***AbstractCollection***. Але, якщо клас вже успадковується від іншого класу, успадкування ще й від ***AbstractCollection*** неможливо. В цьому випадку для реалізації ***Collection*** доведеться реалізувати всі методи інтерфейсу, і тоді набагато простіше обмежитися успадкуванням і додати можливість створення ітератора:
 
``` java
//: holding/NonCollectionSequence.java 
import typeinfo.pets.*; 
import java.util.*; 
 
class PetSequence { 
  protected Pet[] pets = Pets.createArray(8); 
} 
 
public class NonCollectionSequence extends PetSequence { 
  public Iterator<Pet> iterator() { 
    return new Iterator<Pet>() { 
      private int index = 0; 
      public boolean hasNext() { 
        return index < pets.length; 
      } 
      public Pet next() { return pets[index++]; } 
      public void remove() { // Not implemented 
        throw new UnsupportedOperationException(); 
      } 
    }; 
  } 
  public static void main(String[] args) { 
    NonCollectionSequence nc = new NonCollectionSequence(); 
    InterfaceVsIterator.display(nc.iterator()); 
  } 
} /* Output: 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 
*///:~ 
```
 
Створення ***Iterator*** забезпечує мінімальну логічну прив'язку між послідовністю і методом, що використовує цю послідовність, а також накладає набагато менше обмежень на клас послідовності, який реалізує ***Collection***.
 
# Синтаксис foreach і ітератори
 
До теперішнього моменту «синтаксис ***foreach***» використовувався в основному з масивами, але він також буде працювати з будь-яким об'єктом ***Collection***. Деякі приклади вже зустрічалися нам при роботі з ***ArrayList***, але можна навести і більш загальне підтвердження:
 
``` java
//: holding/ForEachCollections.java 
// Синтаксис foreach працює з будь-якими колекціями. 
import java.util.*; 
 
public class ForEachCollections { 
  public static void main(String[] args) { 
    Collection<String> cs = new LinkedList<String>(); 
    Collections.addAll(cs, 
      "Take the long way home".split(" ")); 
    for(String s : cs) 
      System.out.print("‘" + s + "‘ "); 
  } 
} /* Output: 
‘Take’ ‘the’ ‘long’ ‘way’ ‘home’ 
*///:~ 
```
 
Оскільки ***cs*** є має тип ***Collection***, цей приклад показує, що підтримка ***foreach*** є характеристикою всіх об'єктів ***Collection***. Робота цієї конструкції пояснюється тим, що в *Java* *SE5* з'явився новий інтерфейс ***Iterable***, який містить метод ***iterator()*** для створення ***Iterator***, і саме інтерфейс ***Iterable*** використовується при переборі послідовності в синтаксисі ***foreach***. Отже, створивши будь-який клас, який реалізує ***Iterable***, ви зможете використовувати його в синтаксисі ***foreach***:
 
``` java
//: holding/IterableClass.java 
// Anything Iterable works with foreach. 
import java.util.*; 
 
public class IterableClass implements Iterable<String> { 
  protected String[] words = ("And that is how " + 
    "we know the Earth to be banana-shaped.").split(" "); 
  public Iterator<String> iterator() { 
    return new Iterator<String>() { 
      private int index = 0; 
      public boolean hasNext() { 
        return index < words.length; 
      } 
      public String next() { return words[index++]; } 
      public void remove() { // Не реалізовано 
        throw new UnsupportedOperationException(); 
      } 
    }; 
  }  
  public static void main(String[] args) { 
    for(String s : new IterableClass()) 
      System.out.print(s + " "); 
  } 
} /* Output: 
And that is how we know the Earth to be banana-shaped. 
*///:~ 
```
 
Метод ***iterator()*** повертає екземпляр анонімної внутрішньої реалізації ***Iterator <String>***, доставляє кожне слово в масив. В ***main()*** ми бачимо, що ***IterableClass*** дійсно працює в синтаксисі ***foreach***. В *Java* *SE5* багато класи реалізують ***Iterable***, перш за все всі класи ***Collection*** (але не ***Map***). Наприклад, наступний код виводить всі змінні оточення (*environment*) операційної системи:
 
``` java
//: holding/EnvironmentVariables.java 
import java.util.*; 
 
public class EnvironmentVariables { 
  public static void main(String[] args) { 
    for(Map.Entry entry: System.getenv().entrySet()) { 
      System.out.println(entry.getKey() + ": " + 
        entry.getValue()); 
    } 
  } 
} /* (Execute to see output) *///:~ 
```
 
***System.getenv()*** повертає ***Map, entrySet()*** створює ***Set*** з елементами ***Map.Entry***, a ***Set*** підтримує ***Iterable*** і тому може використовуватися в циклі ***foreach***. Синтаксис ***foreach*** працює з масивами і всім, що підтримує ***Iterable***, але це не означає, що масив автоматично підтримує ***Iterable***:
 
``` java
//: holding/ArrayIsNotIterable.java 
import java.util.*; 
 
public class ArrayIsNotIterable { 
  static <T> void test(Iterable<T> ib) { 
    for(T t : ib) 
      System.out.print(t + " "); 
  } 
  public static void main(String[] args) { 
    test(Arrays.asList(1, 2, 3)); 
    String[] strings = { "A", "B", "C" }; 
	// Масив працює в foreach. але не є Iterable:
	//! test (strings);
	// Його необхідно явно перетворити до Iterable:
    test(Arrays.asList(strings)); 
  } 
} /* Output: 
1 2 3 A B C 
*///:~ 
```
 
Спроба передачі масиву в аргументі ***Iterable*** завершується невдачею. Автоматичне перетворення в ***Iterable*** не проводиться; його необхідно виконувати вручну. 

## Ідіома «метод-адаптер»

Що робити, якщо у вас є існуючий клас, який реалізує ***Iterable***, і ви хочете додати нові способи використання цього класу в синтаксисі ***foreach***? Припустимо, ви хочете мати можливість вибору між перебором списку слів в прямому, або зворотному напрямку. Якщо просто скористатися успадкуванням від класу і перевизначити метод ***iterator***, то існуючий метод буде замінений і ніякого вибору не буде. Одне з рішень цієї проблеми грунтується на використанні ідіоми, яку я називаю «методом-адаптером». Термін «адаптер»
походить від однойменного патерну: ви повинні надати інтерфейс, необхідний для роботи синтаксису ***foreach***. Якщо у вас є один інтерфейс, а потрібен інший, проблема вирішується написанням адаптера. В даному випадку потрібно додати до стандартного «прямого»  зворотній ітератор, так що перевизначення виключено. Замість цього ми додамо метод, який створює об'єкт ***Iterable***, який може використовуватися в синтаксисі ***foreach***. Як буде показано далі, це дозволить нам надати кілька варіантів використання ***foreach***:
 
``` java
//: holding/AdapterMethodIdiom.java 
// Ідіома "метод-адаптер" дозволяє використовувати foreach
// з додатковими різновидами Iterables. 
import java.util.*; 
 
class ReversibleArrayList<T> extends ArrayList<T> { 
  public ReversibleArrayList(Collection<T> c) { super(c); } 
  public Iterable<T> reversed() { 
    return new Iterable<T>() { 
      public Iterator<T> iterator() { 
        return new Iterator<T>() { 
          int current = size() - 1; 
          public boolean hasNext() { return current > -1; } 
          public T next() { return get(current--); } 
          public void remove() { // Not implemented 
            throw new UnsupportedOperationException(); 
          } 
        }; 
      } 
    }; 
  } 
}   
 
public class AdapterMethodIdiom { 
  public static void main(String[] args) { 
    ReversibleArrayList<String> ral = 
      new ReversibleArrayList<String>( 
        Arrays.asList("To be or not to be".split(" "))); 
    // Grabs the ordinary iterator via iterator(): 
    for(String s : ral) 
      System.out.print(s + " "); 
    System.out.println(); 
    // Hand it the Iterable of your choice 
    for(String s : ral.reversed()) 
      System.out.print(s + " "); 
  } 
} /* Output: 
To be or not to be 
be to not or be To 
*///:~
```
 
Якщо просто помістити об'єкт ***ral*** в синтаксис ***foreach***, ми отримаємо (стандартний) «прямий» ітератор. Але якщо викликати для об'єкта ***reversed()***, поведінка зміниться. Використавши цей прийом, можна додати в приклад ***IterableClass.java*** два методи-адаптери:
 
``` java
//: holding/MultiIterableClass.java 
// Adding several Adapter Methods. 
import java.util.*; 
 
public class MultiIterableClass extends IterableClass { 
  public Iterable<String> reversed() { 
    return new Iterable<String>() { 
      public Iterator<String> iterator() { 
        return new Iterator<String>() { 
          int current = words.length - 1; 
          public boolean hasNext() { return current > -1; } 
          public String next() { return words[current--]; } 
          public void remove() { // Not implemented 
            throw new UnsupportedOperationException(); 
          } 
        }; 
      } 
    }; 
  }  
  public Iterable<String> randomized() { 
    return new Iterable<String>() { 
      public Iterator<String> iterator() { 
        List<String> shuffled = 
          new ArrayList<String>(Arrays.asList(words)); 
        Collections.shuffle(shuffled, new Random(47)); 
        return shuffled.iterator(); 
      } 
    }; 
  }  
  public static void main(String[] args) { 
    MultiIterableClass mic = new MultiIterableClass(); 
    for(String s : mic.reversed()) 
      System.out.print(s + " "); 
    System.out.println(); 
    for(String s : mic.randomized()) 
      System.out.print(s + " "); 
    System.out.println(); 
    for(String s : mic) 
      System.out.print(s + " "); 
  } 
} /* Output: 
banana-shaped. be to Earth the know we how is that And 
is banana-shaped. Earth that how the be And we know to 
And that is how we know the Earth to be banana-shaped. 
*///:~
```
 
З вихідних даних видно, що метод ***Collections.shuffle*** не змінює вихідний масив, а тільки переставляє посилання в ***shuffled***. Так відбувається тільки тому, що метод ***randomized()*** створює для результату ***Arrays.asList()*** «обгортку» у вигляді ***ArrayList***. Якби операція виконувалася безпосередньо з об'єктом ***List***, отриманим від ***Arrays.asList()***, то це призвело б до зміни нижчого масиву:
 
``` java
//: holding/ModifyingArraysAsList.java 
import java.util.*; 
 
public class ModifyingArraysAsList { 
  public static void main(String[] args) { 
    Random rand = new Random(47); 
    Integer[] ia = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 }; 
    List<Integer> list1 = 
      new ArrayList<Integer>(Arrays.asList(ia)); 
    System.out.println("Before shuffling: " + list1); 
    Collections.shuffle(list1, rand); 
    System.out.println("After shuffling: " + list1); 
    System.out.println("array: " + Arrays.toString(ia)); 
 
    List<Integer> list2 = Arrays.asList(ia); 
    System.out.println("Before shuffling: " + list2); 
    Collections.shuffle(list2, rand); 
    System.out.println("After shuffling: " + list2); 
    System.out.println("array: " + Arrays.toString(ia)); 
  } 
} /* Output: 
Before shuffling: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
After shuffling: [4, 6, 3, 1, 8, 7, 2, 5, 10, 9] 
array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
Before shuffling: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
After shuffling: [9, 1, 6, 3, 7, 2, 5, 10, 4, 8] 
array: [9, 1, 6, 3, 7, 2, 5, 10, 4, 8] 
*///:~ 
```
 
У першому випадку вивід ***Arrays.asList()*** передається конструктору ***ArrayList()***, а останній створює об'єкт ***ArrayList***, який посилається на елементи ***ia***. Перестановка цих посилань не змінює масиву. Але, якщо ми використовуємо результат ***Arrays.asList(ia)*** безпосередньо, перестановка змінить порядок ***ia***. Важливо враховувати, що ***Arrays.asList()*** створює об'єкт ***List***, який використовує нижчележачий масив в якості своєї фізичної реалізації. Якщо з цим об'єктом ***List*** виконуються будь-які операції, але ви не хочете зміни вихідного масиву, створіть копію в іншому контейнері. 

# Резюме

В *Java* існує кілька способів зберігання об'єктів:

- У масивах об'єктам призначаються числові індекси. Масив містить об'єкти заздалегідь відомого типу, тому перетворення типу при вибірці об'єкта не потрібно. Масив може бути багатовимірним і може використовуватися для зберігання примітивних типів. Проте змінити розмір створеного масиву неможливо.
- В ***Collection*** зберігаються окремі елементи, а в ***Map*** - пари асоційованих елементів. Механізм параметризації дозволяє задати тип об'єктів, що зберігаються в контейнері, тому помістити в контейнер об'єкт невірного типу неможливо, і елементи не потребують перетворення типу при вибірці. І ***Collection***, і ***Map*** автоматично змінюються в розмірах при додаванні нових елементів. У контейнерах не можуть зберігатися примітиви, але механізм автоматичної упаковки автоматично створює об'єктні «обгортки», які зберігаються в контейнері.
- У контейнері ***List***, як і в масиві, об'єктам призначаються числові індекси - таким чином, масиви і ***List*** є впорядкованими контейнерами.
- Використовуйте ***ArrayList*** при частому використанні довільного доступу до елементів, або ***LinkedList*** при частому виконанні операцій вставки і видалення в середині списку.
- Поведінка черг і стеків забезпечується контейнером ***LinkedList***.
- Контейнер ***Map*** пов'язує з об'єктами не цілочисельний індекс, а інший об'єкт. Контейнери ***HashMap*** оптимізовані для швидкого доступу, а контейнер ***TreeMap*** зберігає ключі в відсортованому порядку, але поступається за швидкістю ***HashMap***. У контейнері ***LinkedHashMap*** елементи зберігаються в порядку вставки, але хешування забезпечує швидкий доступ.
- У контейнері ***Set*** кожен об'єкт може зберігатися тільки в одному екземплярі. Контейнер ***HashSet*** забезпечує максимальну швидкість пошуку, а в ***TreeSet*** елементи зберігаються в відсортованому порядку. У контейнері ***LinkedHashSet*** елементи зберігаються в порядку вставки.
- Використовувати старі класи ***Vector, Hashtable*** і ***Stack*** в новому коді не потрібно.

Контейнери *Java* - необхідний інструмент, яким ви будете постійно користуватися у своїй повсякденній роботі; завдяки ним ваш код стане простішим, потужним і ефективним. Можливо, на освоєння деяких аспектів контейнерів потрібен час, але ви швидко звикнете до класів цієї бібліотеки і почнете використовувати їх.

 