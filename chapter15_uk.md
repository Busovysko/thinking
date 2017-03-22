# Масиви

Наприкінці глави "Ініціалізація і очистка" було показано, як визначити і ініціалізувати масив. Програміст створює і ініціалізує масиви, витягує з них елементи по цілочисельним індексам, а розмір масиву залишається незмінним. Як правило, при роботі з масивами цього цілком достатньо, але іноді доводиться виконувати більш складні операції, а також оцінювати ефективність масиву в порівнянні з іншими контейнерами. В цьому розділі масиви розглядаються на більш глибокому рівні.

## Чому масиви такі особливі?

В *Java* існує чимало різних способів зберігання об'єктів, чому ж масиви займають особливе місце?

Масиви відрізняються від інших контейнерів за трьома показниками: ефективність, типізація і можливість зберігання примітивів. Масиви *Java* є найефективнішим засобом зберігання і довільного доступу до послідовності посилань на об'єкти. Масив являє собою просту лінійну послідовність, завдяки чому звернення до елементів здійснюються надзвичайно швидко. За швидкість доводиться розплачуватися тим, що розмір об'єкта масиву фіксується і не може змінюватися протягом його життєвого циклу. Як говорилося в главі "Контейнери і зберігання об'єктів", контейнер ***ArrayList*** здатний автоматично виділяти додатковий простір, виділяючи новий блок пам'яті і переміщаючи в нього всі посилання з старого. Хоча зазвичай ***ArrayList*** віддається перевага перед масивами, за гнучкість доводиться розплачуватися, і ***ArrayList*** значно поступається за ефективністю звичайному масиву.

І масиви, і контейнери захищені від можливих зловживань. При виході за межі масиву, або контейнера відбувається виключення ***RuntimeException***, що свідчить про помилку програміста.

До появи параметризації іншим контейнерним класам доводилося працювати з об'єктами так, як якщо б вони не мали певного типу. Інакше кажучи, об'єкти розглядалися як такі, що належать до типу ***Object***, кореневого для всієї ієрархії класів в *Java*. Масиви зручніші «старих» контейнерів тим, що масив створюється для зберігання конкретного типу. Перевірка типів на стадії компіляції не дозволить використовувати невірний тип, або невірно інтерпретувати тип який видубувається. Звичайно, *Java* так чи інакше заборонить відправити невідповідне повідомлення об'єкту на стадії компіляції, або виконання, так що жоден із способів не є більш ризикованим порівняно з іншим. Просто буде зручніше, якщо компілятор вкаже на існуючу проблему, і знижується ймовірність того, що користувач програми отримає несподіване виключення.

Масив може містити примітивні типи, а «старі» контейнери - ні. З іншого боку, параметризовані контейнери можуть перевіряти тип збережених об'єктів, а завдяки автоматичній упаковці вони працюють так, як якби підтримували зберігання примітивів, оскільки перетворення виконується автоматично. У наступному прикладі масиви порівнюються з параметризовані контейнерами:

``` java
//: Arrays/ContainerComparison.java
import java.util.*;
import static net.mindview.util.Print.*;

class BerylliumSphere {
  private static long counter;
  private final long id = counter ++;
  public String toString () {return "Sphere" + id; }
}

public class ContainerComparison {
  public static void main (String [] args) {
    BerylliumSphere [] spheres = new BerylliumSphere [10];
    for (int i = 0; i <5; i ++)
      spheres [i] = new BerylliumSphere ();
    print (Arrays.toString (spheres));
    print (spheres [4]);

    List <BerylliumSphere> sphereList =
      new ArrayList <BerylliumSphere> ();
    for (int i = 0; i <5; i ++)
      sphereList.add (new BerylliumSphere ());
    print (sphereList);
    print (sphereList.get (4));

    int [] integers = {0, 1, 2, 3, 4, 5};
    print (Arrays.toString (integers));
    print (integers [4]);

    List <Integer> intList = new ArrayList <Integer> (
      Arrays.asList (0, 1, 2, 3, 4, 5));
    intList.add (97);
    print (intList);
    print (intList.get (4));
  }
} /* Output:
[Sphere 0, Sphere 1, Sphere 2, Sphere 3, Sphere 4, null, null, null,
null, null]
Sphere 4
[Sphere 5, Sphere 6, Sphere 7, Sphere 8, Sphere 9]
Sphere 9
[0, 1, 2, 3, 4, 5]
4
[0, 1, 2, 3, 4, 5, 97]
4
*///:~
```

Обидва способи зберігання об'єктів забезпечують перевірку типів, а єдинf очевиднf відмінність полягає в тому, що масиви використовують для звернення до елементів конструкцію ***[]***, a ***List*** - методи на кшталт ***add()***, або ***get()***. Розробники мови навмисно зробили масиви і ***ArrayList*** настільки схожими, щоб програмісту було концептуально простіше переключатися між ними. Але, як було показано в главі "Контейнери і зберігання об'єктів", контейнери мають більш широкі можливості, ніж масиви. З появою механізму автоматичної упаковки контейнери за зручністю роботи з примітивами майже не поступаються масивам. Єдиною реальною перевагою масивів залишається їх ефективність. Втім, при вирішенні більш загальних проблем може виявитися, що можливості масивів занадто обмежені, і тоді доводиться користуватися контейнерними класами.

## Масив як об'єкт

З яким би типом масиву ви не працювали, ідентифікатор масиву в дійсності представляє собою посилання на об'єкт, створений у динамічної пам'яті. Цей об'єкт містить посилання на інші об'єкти і створюється або неявно (в синтаксисі ініціалізації масиву), або явно конструкцією ***new***. Однією з частин об'єкта масиву (а по суті, єдиним доступним полем) є доступна лише для читання змінна ***length***, яка вказує, скільки елементів може зберігатися в об'єкті масиву. Весь доступ до об'єкта масиву обмежується синтаксисом *** []***. Наступний приклад демонструє різні способи ініціалізації масивів і присвоювання посилань на масиви. Він також наочно показує, що масиви об'єктів і масиви примітивних типів практично ідентичні. Єдина відмінність полягає в тому, що масиви об'єктів містять посилання, а масиви примітивів містять самі примітивні значення.

``` java
//: arrays/ArrayOptions.java
// Ініціалізація і повторне присвоювання масивів
import java.util.*;
import static net.mindview.util.Print.*;

public class ArrayOptions {
  public static void main(String[] args) {
    // Масиви об'єктів:
    BerylliumSphere[] a; // Локальна неініціалізованих змінна
    BerylliumSphere[] b = new BerylliumSphere[5];
    // Посилання в масиві автоматично ініціалізовано як null:
    print("b: " + Arrays.toString(b));
    BerylliumSphere[] c = new BerylliumSphere[4];
    for(int i = 0; i < c.length; i++)
      if(c[i] == null) // Перевірка посилання на null
        c[i] = new BerylliumSphere();
    // Агрегатна ініціалізація:
    BerylliumSphere[] d = { new BerylliumSphere(),
      new BerylliumSphere(), new BerylliumSphere()
    };
    // Динамічна агрегатна ініціалізація:
    a = new BerylliumSphere[]{
      new BerylliumSphere(), new BerylliumSphere(),
    };
    // (Завершальна кома не обов'язкова в обох випадках)
    print("a.length = " + a.length);
    print("b.length = " + b.length);
    print("c.length = " + c.length);
    print("d.length = " + d.length);
    a = d;
    print("a.length = " + a.length);

    // Масиви примітивів:
    int[] e; // Посилання null
    int[] f = new int[5];
    // Примітиви в масиві автоматично ініціалізовано нулями:
    print("f: " + Arrays.toString(f));
    int[] g = new int[4];
    for(int i = 0; i < g.length; i++)
      g[i] = i*i;
    int[] h = { 11, 47, 93 };
    // Помилка компіляції змінна e НЕ ініціалізована:
    //!print("e.length = " + e.length);
    print("f.length = " + f.length);
    print("g.length = " + g.length);
    print("h.length = " + h.length);
    e = h;
    print("e.length = " + e.length);
    e = new int[]{ 1, 2 };
    print("e.length = " + e.length);
  }
} /* Output:
b: [null, null, null, null, null]
a.length = 2
b.length = 5
c.length = 4
d.length = 3
a.length = 3
Arrays  537  
f: [0, 0, 0, 0, 0]
f.length = 5
g.length = 4
h.length = 3
e.length = 3
e.length = 2
*///:~
```

Масив ***а*** - неініціалізована локальна змінна, і компілятор не дозволяє що-небудь робити з нею доти, поки вона не буде відповідним чином ініціалізована. Масив ***b*** ініціалізується масивом посилань на об'єкти ***BerylliumSpere***, хоча в масиві немає жодного такого об'єкту. Незважаючи на це, ми можемо запросити розмір масиву, тому що ***b*** вказує на дійсний об'єкт. У цьому проявляється певний недолік масивів: поле ***length*** повідомляє, скільки елементів може бути вміщено в масив, тобто розмір об'єкта масиву, а не кількість об'єктів що зберігаються в ньому. Проте при створенні об'єкта масиву всі посилання автоматично ініціалізовано значенням ***null***, і щоб дізнатися, чи пов'язаний певний елемент масиву з об'єктом, досить перевірити посилання на рівність ***null***. Аналогічно, масиви примітивних типів автоматично ініціалізовано нулями для числових типів: ***(char) 0*** для ***char*** і ***false*** для ***boolean***.

Масив ***c*** демонструє створення масиву з подальшим привласненням об'єктів ***BerylliumSphere*** для всіх елементів масиву. Масив ***d*** демонструє синтаксис «агрегатної ініціалізації», при якому об'єкт масиву створюється (з ключовим словом ***new***, як масив ***с***) і ініціалізується об'єктами ***BerylliumSphere***, причому все це відбувається в одній команді.

Наступну конструкцію ініціалізації масиву можна назвати «динамічно агрегатною ініціалізацією». Агрегатна ініціалізація, використовувана ***d***, повинна використовуватися в точці визначення ***d***, але при другому синтаксисі об'єкт масиву може створюватися і використовуватися в будь-якій точці. Припустимо, методу ***hide()*** передається масив об'єктів ***BerylliumSphere***. Його виклик може виглядати так:

``` java
 hide (d);
```

проте масив, який передається в аргументі, також можна створити динамічно:

``` java
 hide (new BerylliumSphere [] {new BeryllіumSphere (), new BerylliumSphere ()});
```

У багатьох ситуаціях такий синтаксис виявляється більш зручним.

Вираз

``` java
 a = d;
```

показує, як взяти посилання, пов'язане з одним об'єктом масиву, і привласнити його іншому об'єкту масиву, як це робиться з будь-яким іншим типом посилання на об'єкт. В результаті ***a*** і ***d*** вказують на один об'єкт масиву в купі.

Друга частина ***ArrayOptions.java*** показує, що примітивні масиви працюють точно так само, як масиви об'єктів, за винятком того, що примітивні значення зберігаються в них безпосередньо.

## Повернення масиву

Припустимо, ви пишете метод, який повинен повертати не окреме значення, а цілий набір значень. У таких мовах, як *C* і *C++*, це зробити нелегко, тому що повертається з методу не масив, а тільки покажчик на масив. При цьому виникають проблеми, оскільки складності з керуванням життєвим циклом масиву можуть привести до витоку пам'яті. В *Java* ви просто повертаєте масив. Вам не потрібно турбуватися про нього - масив буде існувати до тих пір, поки він вам потрібен, а коли потреба в ньому відпаде, масив буде знищений збирачем сміття. Як приклад розглянемо повернення масиву ***String***:

``` java
//: arrays/IceCream.java
// Повернення масивів з методів import
import java.util.*;

public class IceCream {
  private static Random rand = new Random(47);
  static final String[] FLAVORS = {
    "Chocolate", "Strawberry", "Vanilla Fudge Swirl",
    "Mint Chip", "Mocha Almond Fudge", "Rum Raisin",
    "Praline Cream", "Mud Pie"
  };
  public static String[] flavorSet(int n) {
    if(n > FLAVORS.length)
      throw new IllegalArgumentException("Set too big");
    String[] results = new String[n];
    boolean[] picked = new boolean[FLAVORS.length];
    for(int i = 0; i < n; i++) {
      int t;
      do
        t = rand.nextInt(FLAVORS.length);
      while(picked[t]);
      results[i] = FLAVORS[t];
      picked[t] = true;
    }
    return results;
  }
  public static void main(String[] args) {
    for(int i = 0; i < 7; i++)
      System.out.println(Arrays.toString(flavorSet(3)));
  }
} /* Output:
[Rum Raisin, Mint Chip, Mocha Almond Fudge]
[Chocolate, Strawberry, Mocha Almond Fudge]
[Strawberry, Mint Chip, Mocha Almond Fudge]
[Rum Raisin, Vanilla Fudge Swirl, Mud Pie]
[Vanilla Fudge Swirl, Chocolate, Mocha Almond Fudge]
[Praline Cream, Strawberry, Mocha Almond Fudge]
[Mocha Almond Fudge, Strawberry, Mint Chip]
*///:~
```

Метод ***flavorSet()*** створює масив ***results*** з елементами ***String***. Розмір масиву дорівнює ***n***; він визначається аргументом, що передаються при виклику методу. Далі метод випадковим чином вибирає елементи з масиву ***FLAVORS*** і поміщає їх в масив ***results***, що повертається методом. Масив повертається так само, як будь-який інший об'єкт, - за посиланням. При цьому не важливо, чи був масив створений методом ***flavorSet()***, чи він був створений в іншому місці. Масив залишиться з вами весь час, поки він буде потрібен, а потім збирач сміття подбає про його знищення. З вихідних даних видно, що метод ***flavorSet()*** дійсно вибирає випадкову підмножину елементів при кожному виклику.

## Багатовимірні масиви

Створення багатовимірних масивів в *Java* не викликає особливих складнощів. Для багатовимірних масивів примітивних типів кожен вектор обрамлений в фігурні дужки:

``` java
//: arrays/MultidimensionalPrimitiveArray.java
// Creating multidimensional arrays.
import java.util.*;

public class MultidimensionalPrimitiveArray {
  public static void main(String[] args) {
    int[][] a = {
      { 1, 2, 3, },
      { 4, 5, 6, },
    };
    System.out.println(Arrays.deepToString(a));
  }
} /* Output:
[[1, 2, 3], [4, 5, 6]]
*///:~
```

Кожна вкладена пара фігурних дужок описує нову розмірність масиву. У цьому прикладі використовується метод *Java* *SE5* ***Arrays.deepToString()***. Як видно з вихідних даних, він перетворює багатовимірні масиви в ***String***. Масив також може створюватися ключовим словом ***new***. Приклад створення тривимірного масиву виразом ***new***:

``` java
//: arrays/ThreeDWithNew.java
import java.util.*;

public class ThreeDWithNew {
  public static void main(String[] args) {
    // 3-D array with fixed length:
    int[][][] a = new int[2][2][4];
    System.out.println(Arrays.deepToString(a));
  }
} /* Output:
[[[0, 0, 0, 0], [0, 0, 0, 0]], [[0, 0, 0, 0], [0, 0, 0, 0]]]
*///:~
```

Як бачите, якщо масиву примітивних типів не задані явні значення, він автоматично ініціалізується значеннями за замовчуванням. Масиви об'єктів ініціалізуються посиланнями ***null***. Вектори масивів, що утворюють матрицю, можуть мати різну довжину (це називається ступінчастим масивом):

``` java
//: arrays/RaggedArray.java
import java.util.*;

public class RaggedArray {
  public static void main(String[] args) {
    Random rand = new Random(47);
    // Тривимірний масив з векторами змінної довжини:
    int[][][] a = new int[rand.nextInt(7)][][];
    for(int i = 0; i < a.length; i++) {
      a[i] = new int[rand.nextInt(5)][];
      for(int j = 0; j < a[i].length; j++)
        a[i][j] = new int[rand.nextInt(5)];
    }
    System.out.println(Arrays.deepToString(a));
  }
} /* Output:
[[], [[0], [0], [0, 0, 0, 0]], [[], [0, 0], [0, 0]], [[0, 0, 0], [0],
[0, 0, 0, 0]], [[0, 0, 0], [0, 0, 0], [0], []], [[0], [], [0]]]
*///:~
```

Перша конструкція ***new*** створює масив, у якого перший елемент має випадкову довжину, а решта залишаються невизначеними. Друга конструкція ***new*** в циклі ***for***
заповнює елементи, але залишає третій індекс невизначеним аж до виконання третього ***new***.

Масиви з не-примітивними елементами заповнюються аналогічним чином. Приклад об'єднання декількох виразів ***new*** в фігурних дужках:

``` java
//: arrays/MultidimensionalObjectArrays.java
import java.util.*;

public class MultidimensionalObjectArrays {
  public static void main(String[] args) {
    BerylliumSphere[][] spheres = {
      { new BerylliumSphere(), new BerylliumSphere() },
      { new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere() },
      { new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere() },
    };
    System.out.println(Arrays.deepToString(spheres));
  }
} /* Output:
[[Sphere 0, Sphere 1], [Sphere 2, Sphere 3, Sphere 4, Sphere 5], [Sphere
6, Sphere 7, Sphere 8, Sphere 9, Sphere 10, Sphere 11, Sphere 12, Sphere
13]]
*///:~
```

Масив ***spheres*** також є ступінчастим, тобто довжини вкладених списків об'єктів розрізняються. Механізм автоматичного пакування працює з ініціалізатором масивів:

``` java
//: arrays/AutoboxingArrays.java
import java.util.*;

public class AutoboxingArrays {
  public static void main(String[] args) {
    Integer[][] a = { // Autoboxing:
      { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 },
      { 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 },
      { 51, 52, 53, 54, 55, 56, 57, 58, 59, 60 },
      { 71, 72, 73, 74, 75, 76, 77, 78, 79, 80 },
    };
    System.out.println(Arrays.deepToString(a));
  }
} /* Output:
[[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], [21, 22, 23, 24, 25, 26, 27, 28, 29,
30], [51, 52, 53, 54, 55, 56, 57, 58, 59, 60], [71, 72, 73, 74, 75, 76,
77, 78, 79, 80]]
*///:~
```

А ось як відбувається поелементна побудова масиву не-примітивних об'єктів:

``` java
//: arrays/AssemblingMultidimensionalArrays.java
// Створення багатовимірних масивів
import java.util.*;

public class AssemblingMultidimensionalArrays {
  public static void main(String[] args) {
    Integer[][] a;
    a = new Integer[3][];
    for(int i = 0; i < a.length; i++) {
      a[i] = new Integer[3];
      for(int j = 0; j < a[i].length; j++)
        a[i][j] = i * j; // Autoboxing
    }
    System.out.println(Arrays.deepToString(a));
  }
} /* Output:
[[0, 0, 0], [0, 1, 2], [0, 2, 4]]
*///:~
```
Вираз ***i \* j*** присутній тільки для того, щоб помістити менш тривіальне значення в ***Integer***. Метод ***Arrays.deepToString()*** працює як з масивами примітивних типів, так і з масивами об'єктів:

``` java
//: arrays/MultiDimWrapperArray.java
// Multidimensional arrays of "wrapper" objects.
import java.util.*;

public class MultiDimWrapperArray {
  public static void main(String[] args) {
    Integer[][] a1 = { // Autoboxing
      { 1, 2, 3, },
      { 4, 5, 6, },
    };
    Double[][][] a2 = { // Autoboxing
      { { 1.1, 2.2 }, { 3.3, 4.4 } },
      { { 5.5, 6.6 }, { 7.7, 8.8 } },
      { { 9.9, 1.2 }, { 2.3, 3.4 } },
    };
    String[][] a3 = {
      { "The", "Quick", "Sly", "Fox" },
      { "Jumped", "Over" },
      { "The", "Lazy", "Brown", "Dog", "and", "friend" },
    };
    System.out.println("a1: " + Arrays.deepToString(a1));
    System.out.println("a2: " + Arrays.deepToString(a2));
    System.out.println("a3: " + Arrays.deepToString(a3));
  }
} /* Output:
a1: [[1, 2, 3], [4, 5, 6]]
a2: [[[1.1, 2.2], [3.3, 4.4]], [[5.5, 6.6], [7.7, 8.8]], [[9.9, 1.2],
[2.3, 3.4]]]
a3: [[The, Quick, Sly, Fox], [Jumped, Over], [The, Lazy, Brown, Dog,
and, friend]]
*///:~     
```

І знову в масивах ***Integer*** і ***Double*** механізм автоматичної упаковки *Java* *SE5* створює необхідні об'єкти- «обгортки».

## Масиви і параметризація

У загальному випадку масиви і параметризація погано поєднуються один з одним. Наприклад, масиви не можуть ініціалізовуватись параметризованими типами:

``` java
Peel<Banana>[] peels = new Peel <Banana> [10];  // He дозволено
```

Стирання видаляє інформацію про параметр типу, а масив повинен знати точний тип зберігаються в ньому об'єктів для забезпечення безпеки типів. Втім, параметризовати можна сам тип масиву:

``` java
//: arrays/ParameterizedArrayType.java

class ClassParameter<T> {
  public T[] f(T[] arg) { return arg; }
}

class MethodParameter {
  public static <T> T[] f(T[] arg) { return arg; }
}

public class ParameterizedArrayType {
  public static void main(String[] args) {
    Integer[] ints = { 1, 2, 3, 4, 5 };
    Double[] doubles = { 1.1, 2.2, 3.3, 4.4, 5.5 };
    Integer[] ints2 =
      new ClassParameter<Integer>().f(ints);
    Double[] doubles2 =
      new ClassParameter<Double>().f(doubles);
    ints2 = MethodParameter.f(ints);
    doubles2 = MethodParameter.f(doubles);
  }
} ///:~
```

Зверніть увагу, як зручно використовувати параметризований метод замість параметризованого класу: вам не доведеться створювати чергову «версію» класу з параметром для кожного типу, до якого він застосовується, і його можна зробити ***static***. Звичайно, параметризований клас не завжди можна замінити параметризованим методом, але таке рішення може виявитися кращим.

Як з'ясовується, не зовсім правильно говорити, що ви не можете створювати масиви параметризованих типів. Дійсно, компілятор не дозволить створити екземпляр масиву параметризованого типу, але ви можете створити посилання на такий масив. приклад:

``` java
 List <String> [] Is;
```

Така конструкція проходить перевірку без найменших заперечень з боку компілятора. І хоча ви не можете створити об'єкт масиву з параметризацією, можна створити об'єкт непараметризованного типу і перетворити його:

``` java
//: arrays/ArrayOfGenerics.java
// Можливість створення масивів параметрезованих типів.
import java.util.*;

public class ArrayOfGenerics {
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
    List<String>[] ls;
    List[] la = new List[10];     
    ls = (List<String>[])la; // Попередження при неперевіреному перетворенні
    ls[0] = new ArrayList<String>();
    // Приводить до помилки на стадії компіляції:
    //! ls[1] = new ArrayList<Integer>();

    // Проблема: List <String> є підтипом Object
    Object[] objects = ls; // Тому присвоювання можливо
    // Компілює і виконується без помилок і попереджень:
    objects[1] = new ArrayList<Integer>();

    // Але якщо ваші потреби досить елементарні.
	// Створити масив параметрезованих типів можна, хоча
	// І з попередженням про "неперевіреному" перетворенні:
    List<BerylliumSphere>[] spheres =
      (List<BerylliumSphere>[])new List[10];
    for(int i = 0; i < spheres.length; i++)
      spheres[i] = new ArrayList<BerylliumSphere>();
  }
} ///:~   
```

Ми бачимо, що при при отриманні посилання на  ***List<String>[]*** виконується деяка перевірка на стадії компіляції. Проблема в тому, що масиви коваріантного, тому ***List<String>[]*** також є  ***Object[]***, тому вашому масиву можна привласнити  ***ArrayList<Integer>*** без видачі помилок на стадії компіляції або виконання. Якщо ви впевнені в тому, що висхідне перетворення виконуватися не буде, а ваші потреби відносно прості, можна створити масив параметризованих типів, що забезпечує просту перевірку типів на стадії компіляції. Проте параметризований контейнер практично завжди виявляється більш вдалим рішенням, ніж масив параметрезованих типів.

## Створення тестових даних

При експериментах з масивами (і програмами взагалі) корисно мати можливість простого заповнення масивів тестовими даними. Інструментарій, описаний в цьому розділі, заповнює масив об'єктними значеннями.

### Arrays.fill()

Клас ***Arrays*** зі стандартної бібліотеки *Java* містить вельми тривіальний метод ***fill()***: він всього лише дублює одне значення в кожному елементі масиву, а в разі об'єктів копіює одне посилання в кожен елемент. Приклад:

``` java
//: arrays/FillingArrays.java
// Використання Arrays.fill()
import java.util.*;
import static net.mindview.util.Print.*;

public class FillingArrays {
  public static void main(String[] args) {
    int size = 6;
    boolean[] a1 = new boolean[size];
    byte[] a2 = new byte[size];
    char[] a3 = new char[size];
    short[] a4 = new short[size];
    int[] a5 = new int[size];
    long[] a6 = new long[size];
    float[] a7 = new float[size];
    double[] a8 = new double[size];
    String[] a9 = new String[size];
    Arrays.fill(a1, true);
    print("a1 = " + Arrays.toString(a1));
    Arrays.fill(a2, (byte)11);
    print("a2 = " + Arrays.toString(a2));
    Arrays.fill(a3, ‘x’);
    print("a3 = " + Arrays.toString(a3));
    Arrays.fill(a4, (short)17);
    print("a4 = " + Arrays.toString(a4));
    Arrays.fill(a5, 19);
    print("a5 = " + Arrays.toString(a5));
    Arrays.fill(a6, 23);
    print("a6 = " + Arrays.toString(a6));
    Arrays.fill(a7, 29);
    print("a7 = " + Arrays.toString(a7));
    Arrays.fill(a8, 47);
    print("a8 = " + Arrays.toString(a8));
    Arrays.fill(a9, "Hello");
    print("a9 = " + Arrays.toString(a9));
    // Manipulating ranges:
    Arrays.fill(a9, 3, 5, "World");
    print("a9 = " + Arrays.toString(a9));
  }
} /* Output:
a1 = [true, true, true, true, true, true]
a2 = [11, 11, 11, 11, 11, 11]
a3 = [x, x, x, x, x, x]
a4 = [17, 17, 17, 17, 17, 17]
a5 = [19, 19, 19, 19, 19, 19]
a6 = [23, 23, 23, 23, 23, 23]
a7 = [29.0, 29.0, 29.0, 29.0, 29.0, 29.0]
a8 = [47.0, 47.0, 47.0, 47.0, 47.0, 47.0]
a9 = [Hello, Hello, Hello, Hello, Hello, Hello]
a9 = [Hello, Hello, Hello, World, World, Hello]
*///:~
```

Метод заповнює або весь масив, або, як показують дві останні команди, діапазон його елементів. Але, оскільки викликати ***Arrays.fill()*** можна тільки для одного значення даних, отримані результати не дуже корисні.

## Генератори даних

Щоб створювати менш тривіальні масиви даних з більш гнучкими можливостями, ми скористаємося концепцією ***генераторів***, представленої в розділі "Параметризація". Генератор здатний видавати будь-які дані за вашим вибором (нагадаю, що він є прикладом патерну «стратегія» - різні генератори представляють різні стратегії).

У цьому розділі будуть представлені деякі готові генератори, але ви також зможете легко визначити власний генератор для своїх потреб.

Для початку розглянемо найпростіший набір рахункових генераторів для всіх примітивних типів і ***String***. Класи генераторів вкладені в клас ***CountingGenerator***, щоб вони могли використовувати ті ж імена що й типів об'єктів що генеруються. Наприклад, генератор, що створює об'єкти ***Integer***, буде створюватися виразом ***new
CountingGenerator.Integer()***:

``` java
//: net/mindview/util/CountingGenerator.java
// Simple generator implementations.
package net.mindview.util;

public class CountingGenerator {
  public static class
  Boolean implements Generator<java.lang.Boolean> {
    private boolean value = false;
    public java.lang.Boolean next() {
      value = !value; // Почергове перемикання
      return value;
    }
  }
  public static class
  Byte implements Generator<java.lang.Byte> {
    private byte value = 0;
    public java.lang.Byte next() { return value++; }
  }
  static char[] chars = ("abcdefghijklmnopqrstuvwxyz" +
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ").toCharArray();
  public static class
  Character implements Generator<java.lang.Character> {
    int index = -1;
    public java.lang.Character next() {
      index = (index + 1) % chars.length;
      return chars[index];
    }
  }
  public static class
  String implements Generator<java.lang.String> {
    private int length = 7;
    Generator<java.lang.Character> cg = new Character();
    public String() {}
    public String(int length) { this.length = length; }
    public java.lang.String next() {
      char[] buf = new char[length];
      for(int i = 0; i < length; i++)
        buf[i] = cg.next();
      return new java.lang.String(buf);
    }
  }
  public static class
  Short implements Generator<java.lang.Short> {
    private short value = 0;
    public java.lang.Short next() { return value++; }
  }
  public static class
  Integer implements Generator<java.lang.Integer> {
    private int value = 0;
    public java.lang.Integer next() { return value++; }
  }
  public static class
  Long implements Generator<java.lang.Long> {
    private long value = 0;
    public java.lang.Long next() { return value++; }
  }
  public static class
  Float implements Generator<java.lang.Float> {
    private float value = 0;
    public java.lang.Float next() {
      float result = value;
      value += 1.0;
      return result;
    }
  }
  public static class
  Double implements Generator<java.lang.Double> {
    private double value = 0.0;
    public java.lang.Double next() {
      double result = value;
      value += 1.0;
      return result;
    }
  }
} ///:~
```

Кожен клас реалізує деяке поняття «лічби». У разі ***CountingGenerator.Character*** це повторення символів верхнього і нижнього регістра. Клас ***CountingGenerator.String***використовує***CountingGenerator.Character*** для заповнення масиву символів, який потім перетворюється в ***String***. Розмір масиву визначається аргументом конструктора. Зверніть увагу на те, що ***CountingGenerator.String*** використовує базову конструкцію *** Generator <java.lang. Character>*** замість конкретного посилання на ***CountingGenerator.Character.***

Наступна тестова програма використовує рефлексію з ідіомою вкладених генераторів, що дозволяє застосовувати її для будь-якого набору генераторів, побудованого за вказаним зразком:

``` java
//: arrays/GeneratorsTest.java
import net.mindview.util.*;

public class GeneratorsTest {
  public static int size = 10;
  public static void test(Class<?> surroundingClass) {
    for(Class<?> type : surroundingClass.getClasses()) {
      System.out.print(type.getSimpleName() + ": ");
      try {
        Generator<?> g = (Generator<?>)type.newInstance();
        for(int i = 0; i < size; i++)
          System.out.printf(g.next() + " ");
        System.out.println();
      } catch(Exception e) {
        throw new RuntimeException(e);
      }
    }
  }
  public static void main(String[] args) {
    test(CountingGenerator.class);
  }
} /* Output:
Double: 0.0 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0
Float: 0.0 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0
Long: 0 1 2 3 4 5 6 7 8 9
Integer: 0 1 2 3 4 5 6 7 8 9
Short: 0 1 2 3 4 5 6 7 8 9
String: abcdefg hijklmn opqrstu vwxyzAB CDEFGHI JKLMNOP QRSTUVW XYZabcd
efghijk lmnopqr
Character: a b c d e f g h i j
Byte: 0 1 2 3 4 5 6 7 8 9
Boolean: true false true false true false true false true false
*///:~
```

Передбачається, що тестований клас містить серію вкладених об'єктів ***Generator***, кожен з яких має конструктор за замовчуванням (тобто без аргументів). Рефлексійний метод ***getClasses()*** видає інформацію про всі вкладені класи. Далі метод ***test()*** створює екземпляр кожного генератора і виводить результати, отримані при десятикратному виклику ***next()***.

Наступний набір генераторів грунтується на випадкових числах. Так як конструктор ***Random*** ініціалізується константою, результати будуть повторюватися при кожному запуску програми:

``` java
//: Net / mindview / util / RandomGenerator.java
// Генератори, що видають випадкові значення. package net.mindview.util;
//: net/mindview/util/RandomGenerator.java
// Generators that produce random values.
package net.mindview.util;
import java.util.*;

public class RandomGenerator {
  private static Random r = new Random(47);
  public static class
  Boolean implements Generator<java.lang.Boolean> {
    public java.lang.Boolean next() {
      return r.nextBoolean();
    }
  }
  public static class
  Byte implements Generator<java.lang.Byte> {
    public java.lang.Byte next() {
      return (byte)r.nextInt();
    }
  }
  public static class
  Character implements Generator<java.lang.Character> {
    public java.lang.Character next() {
      return CountingGenerator.chars[
        r.nextInt(CountingGenerator.chars.length)];
    }
  }
  public static class
  String extends CountingGenerator.String {
    // Plug in the random Character generator:
    { cg = new Character(); } // Instance initializer
    public String() {}
    public String(int length) { super(length); }
  }
  public static class
  Short implements Generator<java.lang.Short> {
    public java.lang.Short next() {
      return (short)r.nextInt();
    }
  }
  public static class
  Integer implements Generator<java.lang.Integer> {
    private int mod = 10000;
    public Integer() {}
    public Integer(int modulo) { mod = modulo; }
    public java.lang.Integer next() {
      return r.nextInt(mod);
    }
  }
  public static class
  Long implements Generator<java.lang.Long> {
    private int mod = 10000;
    public Long() {}
    public Long(int modulo) { mod = modulo; }
    public java.lang.Long next() {
      return new java.lang.Long(r.nextInt(mod));
    }
  }
  public static class
  Float implements Generator<java.lang.Float> {
    public java.lang.Float next() {
      // Trim all but the first two decimal places:
      int trimmed = Math.round(r.nextFloat() * 100);
      return ((float)trimmed) / 100;
    }
  }
  public static class
  Double implements Generator<java.lang.Double> {
    public java.lang.Double next() {
      long trimmed = Math.round(r.nextDouble() * 100);
      return ((double)trimmed) / 100;
    }
  }
} ///:~         
```

Як бачите, ***RandomGenerator.String*** успадковує від ***CountingGenerator.String***, просто підключаючи новий генератор ***Character***.

Щоб генеруємі числа були не надто великі, ***RandomGenerator.Integer*** за замовчуванням бере залишок від ділення на 10 000, але перевантажений конструктор дозволяє вибрати менше значення. Аналогічний підхід використовується і для ***RandomGenerator.Long***. Для генераторів ***Float***і***Double*** цифри в дробової частини усікаються. Для тестування ***RandomGenerator*** можна скористатися вже готовим класом ***GeneratorsTest***:

``` java
//: arrays/RandomGeneratorsTest.java
import net.mindview.util.*;
public class RandomGeneratorsTest {
  public static void main(String[] args) {
    GeneratorsTest.test(RandomGenerator.class);
  }
} /* Output:
Double: 0.73 0.53 0.16 0.19 0.52 0.27 0.26 0.05 0.8 0.76
Float: 0.53 0.16 0.53 0.4 0.49 0.25 0.8 0.11 0.02 0.8
Long: 7674 8804 8950 7826 4322 896 8033 2984 2344 5810
Integer: 8303 3141 7138 6012 9966 8689 7185 6992 5746 3976
Short: 3358 20592 284 26791 12834 -8092 13656 29324 -1423 5327
String: bkInaMe sbtWHkj UrUkZPg wsqPzDy CyRFJQA HxxHvHq XumcXZJ oogoYWM
NvqeuTp nXsgqia
Character: x x E A J J m z M s
Byte: -60 -17 55 -14 -5 115 39 -37 79 115
Boolean: false true false false true true true true true true
*///:~
```

Щоб змінити кількість генеруємих значень, скористайтеся ***public*** - полем ***GeneratorsTest.size***.

## Створення масивів з використанням генераторів

Для створення масивів на основі ***Generator*** нам будуть потрібні два допоміжних класи. Перший використовує довільний ***Generator*** для отримання масиву типів, похідних від ***Object***. Для вирішення проблеми з примітивами другий клас отримує довільний масив з об'ектамі- «обгортками» і будує для нього відповідний масив примітивів.

``` java
public static class Double implements Generator <java.lang.Double> {
  public java.lang.Double next () {
    long trimmed = Math.round (r.nextDouble () * 100);
    return ((double) trimmed) / 100;
  }
}
```

Перший допоміжний клас може працювати в двох режимах, представлених перевантаженим статичним методом ***аrrау()***. Перша версія методу отримує існуючий масив і заповнює його з використанням ***Generator***. Друга версія отримує об'єкт ***Class.Generator*** і кількість елементів, і створює новий масив, який також заповнюється з використанням ***Generator***. Пам'ятайте, що при цьому створюються тільки масиви субтипов ***Object***, але не масиви примітивних типів:

``` java
//: Net/mindview/util/Generated.java
package net.mindview.util;
import java.util.*;

public class Generated {
  // Заповнення існуючого массіваy:
  public static <T> T [] array (T [] a, Generator <T> gen) {
    return new CollectionData <T> (gen, a.length) .toArray (a);
  }
  // Створення нового масиву:
  @SuppressWarnings ( "Unchecked")
  public static <T> T [] array (Class <T> type, Generator <T> gen, int size) {
    T [] a = (T []) java.lang.reflect.Array.newInstance (type, size);
    return new CollectionData <T> (gen, size) .toArray (a);
  }
}
```

Клас ***CollectionData*** створює об'єкт ***Collection***, заповнений елементами, які були створені генератором ***gen***. Кількість елементів визначається другим аргументом конструктора. Всі субтипи ***Collection*** містять метод ***toArray()***, що заповнює масив-аргумент елементами з ***Collection***. Другий метод використовує рефлексію для динамічного створення нового масиву відповідного типу і розміру. Потім створений масив заповнюється таким же способом, як в першому методі.

Щоб протестувати ***Generated***, ми скористаємося одним з класів ***CountingGenerator***, описаних в попередньому розділі:

``` java
//: arrays/TestGenerated.java
import java.util.*;
import net.mindview.util.*;

public class TestGenerated {
  public static void main(String[] args) {
    Integer[] a = { 9, 8, 7, 6 };
    System.out.println(Arrays.toString(a));
    a = Generated.array(a,new CountingGenerator.Integer());
    System.out.println(Arrays.toString(a));
    Integer[] b = Generated.array(Integer.class,
        new CountingGenerator.Integer(), 15);
    System.out.println(Arrays.toString(b));
  }
} /* Output:
[9, 8, 7, 6]
[0, 1, 2, 3]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
*///:~
```

Хоча масив а ініціалізується, ці дані перезаписувати при виклику ***Generated.array()***. Ініціалізація ***b*** показує, як створити заповнений масив «з нуля».

Параметризація не працює з примітивами, тому для заповнення примітивних масивів будуть використовуватися генератори. Для вирішення цієї проблеми ми створимо перетворювач, який отримує довільний масив об'єктних «обгорток» і перетворює його в масив відповідних примітивних типів. Без нього нам довелося б створювати спеціалізовані генератори для всіх примітивів.

``` java
    return result; 
  } 
  public static int[] primitive(Integer[] in) { 
    int[] result = new int[in.length]; 
    for(int i = 0; i < in.length; i++) 
      result[i] = in[i]; 
    return result; 
  } 
  public static long[] primitive(Long[] in) { 
    long[] result = new long[in.length]; 
    for(int i = 0; i < in.length; i++) 
      result[i] = in[i]; 
    return result; 
  } 
  public static float[] primitive(Float[] in) { 
    float[] result = new float[in.length]; 
    for(int i = 0; i < in.length; i++) 
      result[i] = in[i]; 
    return result; 
  } 
  public static double[] primitive(Double[] in) { 
    double[] result = new double[in.length]; 
    for(int i = 0; i < in.length; i++) 
      result[i] = in[i]; 
    return result; 
  } 
} ///:~ 
```

Кожна версія ***primitive()*** створює примітивний масив правильної довжини, а потім копіює елементи з масиву ***in***. Зверніть увагу на виконання автоматичної розпакування в вираженні

``` java
result [i] = in [i];
```

Приклад використання ConvertTo з обома версіями Generated.array ():

``` java
//: arrays/PrimitiveConversionDemonstration.java 
import java.util.*; 
import net.mindview.util.*; 
 
public class PrimitiveConversionDemonstration { 
  public static void main(String[] args) { 
    Integer[] a = Generated.array(Integer.class, 
        new CountingGenerator.Integer(), 15); 
    int[] b = ConvertTo.primitive(a); 
    System.out.println(Arrays.toString(b)); 
    boolean[] c = ConvertTo.primitive( 
      Generated.array(Boolean.class, 
        new CountingGenerator.Boolean(), 7)); 
    System.out.println(Arrays.toString(c)); 
  } 
} /* Output: 
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14] 
[true, false, true, false, true, false, true] 
*///:~ 
```

Нарешті, наступна програма тестує інструментарій створення масивів з класами ***RandomGenerator***:

``` java
//: arrays/TestArrayGeneration.java 
// Test the tools that use generators to fill arrays. 
import java.util.*; 
import net.mindview.util.*; 
import static net.mindview.util.Print.*; 
 
public class TestArrayGeneration { 
  public static void main(String[] args) { 
    int size = 6; 
    boolean[] a1 = ConvertTo.primitive(Generated.array( 
      Boolean.class, new RandomGenerator.Boolean(), size)); 
    print("a1 = " + Arrays.toString(a1)); 
    byte[] a2 = ConvertTo.primitive(Generated.array( 
      Byte.class, new RandomGenerator.Byte(), size)); 
    print("a2 = " + Arrays.toString(a2)); 
    char[] a3 = ConvertTo.primitive(Generated.array( 
      Character.class, 
      new RandomGenerator.Character(), size)); 
    print("a3 = " + Arrays.toString(a3)); 
    short[] a4 = ConvertTo.primitive(Generated.array( 
      Short.class, new RandomGenerator.Short(), size)); 
    print("a4 = " + Arrays.toString(a4)); 
    int[] a5 = ConvertTo.primitive(Generated.array( 
      Integer.class, new RandomGenerator.Integer(), size)); 
    print("a5 = " + Arrays.toString(a5)); 
    long[] a6 = ConvertTo.primitive(Generated.array( 
      Long.class, new RandomGenerator.Long(), size)); 
    print("a6 = " + Arrays.toString(a6)); 
    float[] a7 = ConvertTo.primitive(Generated.array( 
      Float.class, new RandomGenerator.Float(), size)); 
    print("a7 = " + Arrays.toString(a7)); 
    double[] a8 = ConvertTo.primitive(Generated.array( 
      Double.class, new RandomGenerator.Double(), size)); 
    print("a8 = " + Arrays.toString(a8)); 
  } 
} /* Output: 
a1 = [true, false, true, false, false, true] 
a2 = [104, -79, -76, 126, 33, -64] 
a3 = [Z, n, T, c, Q, r] 
a4 = [-13408, 22612, 15401, 15161, -28466, -12603] 
a5 = [7704, 7383, 7706, 575, 8410, 6342] 
a6 = [7674, 8804, 8950, 7826, 4322, 896] 
a7 = [0.01, 0.2, 0.4, 0.79, 0.27, 0.45] 
a8 = [0.16, 0.87, 0.7, 0.66, 0.87, 0.59] 
*///:~ 
```

Як бачите, всі версії ***ConvertTo.primitive()*** працюють правильно. 

## Допоміжний інструментарій Arrays

У бібліотеку ***java.util*** включений клас ***Arrays***, що містить набір допоміжних статичних методів для роботи з масивами. Основних методів шість: ***equals()*** порівнює два масиви (також існує версія ***deepEquals()*** для багатовимірних масивів); ***fill()*** був описаний раніше в цій главі; ***sort()*** сортує масив; ***binarySearch()*** шукає елемент в відсортованому масиві; ***toString()*** створює уявлення масиву в форматі ***String***, a ***hashCode()*** генерує хеш-код масиву. Всі ці методи перевантажені для всіх примітивних типів і ***Object***. Крім того, метод ***Arrays.asList()*** перетворює будь-яку послідовність, або масив в контейнер ***List*** (див. "Контейнери і зберігання об'єктів"). 

Перш ніж обговорювати методи ***Arrays***, слід розглянути ще один корисний метод, який не входить в ***Arrays***.

### Копіювання масиву

Стандартна бібліотека *Java* містить статичний метод ***System.arraycopy()***, який копіює масиви значно швидше, ніж при ручному копіюванні в циклі ***for***. Метод *** System.arraycopy()*** перевантажений для роботи з усіма типами. Приклад для масивів ***int***:

``` java
//: arrays/CopyingArrays.java 
// Using System.arraycopy() 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class CopyingArrays { 
  public static void main(String[] args) { 
    int[] i = new int[7]; 
    int[] j = new int[10]; 
    Arrays.fill(i, 47); 
    Arrays.fill(j, 99); 
    print("i = " + Arrays.toString(i)); 
    print("j = " + Arrays.toString(j)); 
    System.arraycopy(i, 0, j, 0, i.length); 
    print("j = " + Arrays.toString(j)); 
    int[] k = new int[5]; 
    Arrays.fill(k, 103); 
    System.arraycopy(i, 0, k, 0, k.length); 
    print("k = " + Arrays.toString(k)); 
    Arrays.fill(k, 103); 
    System.arraycopy(k, 0, i, 0, k.length); 
    print("i = " + Arrays.toString(i)); 
    // Objects: 
    Integer[] u = new Integer[10]; 
    Integer[] v = new Integer[5]; 
    Arrays.fill(u, new Integer(47)); 
    Arrays.fill(v, new Integer(99)); 
    print("u = " + Arrays.toString(u)); 
    print("v = " + Arrays.toString(v)); 
    System.arraycopy(v, 0, u, u.length/2, v.length); 
    print("u = " + Arrays.toString(u)); 
  } 
} /* Output: 
i = [47, 47, 47, 47, 47, 47, 47] 
j = [99, 99, 99, 99, 99, 99, 99, 99, 99, 99] 
j = [47, 47, 47, 47, 47, 47, 47, 99, 99, 99] 
k = [47, 47, 47, 47, 47] 
i = [103, 103, 103, 103, 103, 47, 47] 
u = [47, 47, 47, 47, 47, 47, 47, 47, 47, 47] 
v = [99, 99, 99, 99, 99] 
u = [47, 47, 47, 47, 47, 99, 99, 99, 99, 99] 
*///:~     
```

В аргументах ***arraycopy()*** передається вхідний масив, початкова позиція копіювання в вхідному масиві, результуючий масив, початкова позиція копіювання в результуючому масиві і кількість скопійованих елементів. Природно, будь-яке порушення меж масиву призведе до виключення. 

Наведений приклад показує, що копіюватися можуть як примітивні, так і об'єктні масиви. Однак при копіюванні об'єктних масивів копіюються тільки посилання, але не самі об'єкти. Така процедура називається поверхневим копіюванням. ***System.arraycopy()*** не виконує ні автоматичної упаковки, ні автоматичної розпакування -
типи двох масивів повинні повністю збігатися.

### Порівняння масивів

Клас ***Arrays*** містить метод ***equals()*** для перевірки на рівність цілих масивів. Метод перевантажений для примітивів і ***Object***. Щоб два масиви вважалися рівними, вони повинні містити однакову кількість елементів, і кожен елемент повинен бути еквівалентний відповідному елементу іншого масиву (перевірка здійснюється викликом ***equals()*** для кожної пари; для примітивів використовується метод ***equals()*** об'єктної «обгортки» - наприклад, ***Integer.equals()*** для ***int***). приклад:

``` java
//: arrays/ComparingArrays.java 
// Using Arrays.equals() 
import java.util.*; 
import static net.mindview.util.Print.*; 
 
public class ComparingArrays { 
  public static void main(String[] args) { 
    int[] a1 = new int[10]; 
    int[] a2 = new int[10]; 
    Arrays.fill(a1, 47); 
    Arrays.fill(a2, 47); 
    print(Arrays.equals(a1, a2)); 
    a2[3] = 11; 
    print(Arrays.equals(a1, a2)); 
    String[] s1 = new String[4]; 
    Arrays.fill(s1, "Hi"); 
    String[] s2 = { new String("Hi"), new String("Hi"), 
      new String("Hi"), new String("Hi") }; 
    print(Arrays.equals(s1, s2)); 
  } 
} /* Output: 
true 
false 
true 
*///:~     
```

Спочатку масиви ***a1*** і ***a2*** повністю збігаються, тому результат порівняння дорівнює ***true***, але після зміни одного з елементів буде отримано результат ***false***. В останньому випадку всі елементи ***s1*** вказують на один об'єкт, тоді як ***s2*** містить п'ять різних об'єктів. Однак перевірка рівності визначається вмістом (з викликом ***Object.equals()***), тому результат дорівнює ***true***.

### Порівняння елементів масивів

Порівняння, виконувані в ході сортування, залежать від фактичного типу об'єктів. Звичайно, можна написати різні методи сортування для всіх можливих типів, але такий код доведеться модифікувати при появі нових типів. 

Головною метою проектування є «відділення того, що може змінитися, від того, що залишається незмінним». В даному випадку незмінним залишається загальний алгоритм сортування, а змінюється спосіб порівняння об'єктів. Замість того, щоб розміщувати код порівняння в різних функціях сортування, ми скористаємося патерном проектування «стратегія». У цьому паттерні змінна частина коду інкапсолюється в окремому класі. Об'єкт стратегії передається коду, який залишається незмінним, і останній використовує стратегію для реалізації свого алгоритму. При цьому різні об'єкти висловлюють різні способи порівняння, але передаються універсальному коду сортування. 

В *Java* функціональність порівняння може виражатися двома способами. Перший заснований на «природному» методі порівняння, який включається в клас при реалізації ***java.lang.Comparable*** - дуже простого інтерфейсу з єдиним методом ***compareTo()***. В аргументі метод отримує інший об'єкт того ж типу. Він видає від'ємне значення, якщо поточний об'єкт менше аргументу, нуль за однакової кількості і позитивне значення, якщо поточний об'єкт більше аргументу.

У наступному прикладі клас реалізує ***Comparable***, а для демонстрації сумісності використовується метод стандартної бібліотеки *Java* ***Arrays.sort()***:

``` java
//: arrays/CompType.java 
// Реалізація класом інтерфейсу Comparable. 
// Implementing Comparable in a class. 
import java.util.*; 
import net.mindview.util.*; 
import static net.mindview.util.Print.*; 
public class CompType implements Comparable<CompType> { 
  int i; 
  int j; 
  private static int count = 1; 
  public CompType(int n1, int n2) { 
    i = n1; 
    j = n2; 
  } 
  public String toString() { 
    String result = "[i = " + i + ", j = " + j + "]"; 
    if(count++ % 3 == 0) 
      result += "\n"; 
    return result; 
  } 
  public int compareTo(CompType rv) { 
    return (i < rv.i ? -1 : (i == rv.i ? 0 : 1)); 
  } 
  private static Random r = new Random(47); 
  public static Generator<CompType> generator() { 
    return new Generator<CompType>() { 
      public CompType next() { 
        return new CompType(r.nextInt(100),r.nextInt(100)); 
      } 
    }; 
  } 
  public static void main(String[] args) { 
    CompType[] a = 
      Generated.array(new CompType[12], generator()); 
    print("before sorting:"); 
    print(Arrays.toString(a)); 
    Arrays.sort(a); 
    print("after sorting:"); 
    print(Arrays.toString(a)); 
  } 
} /* Output: 
before sorting: 
[[i = 58, j = 55], [i = 93, j = 61], [i = 61, j = 29] 
, [i = 68, j = 0], [i = 22, j = 7], [i = 88, j = 28] 
, [i = 51, j = 89], [i = 9, j = 78], [i = 98, j = 61] 
, [i = 20, j = 58], [i = 16, j = 40], [i = 11, j = 22] 
] 
after sorting: 
[[i = 9, j = 78], [i = 11, j = 22], [i = 16, j = 40] 
, [i = 20, j = 58], [i = 22, j = 7], [i = 51, j = 89] 
, [i = 58, j = 55], [i = 61, j = 29], [i = 68, j = 0] 
, [i = 88, j = 28], [i = 93, j = 61], [i = 98, j = 61] 
] 
*///:~ 
```

Визначаючи метод порівняння, ви несете повну відповідальність за прийняття рішення про його результати. У наведеному прикладі в порівнянні використовуються тільки значення ***и***, а значення ***j*** ігноруються. 

Метод ***generator()*** виробляє об'єкт, який реалізує інтерфейс ***Generator***, створюючи анонімний внутрішній клас. Об'єкт будує об'єкти ***CompType***, ініціалізувавши їх випадковими значеннями. В ***main()*** генератор заповнює масив ***CompType***, який потім сортується. Якщо інтерфейс ***Comparable*** не реалізований, то при спробі виклику ***sort()*** згенерується виключення ***ClassCastException***. Це пояснюється тим, що ***sort()*** перетворює свій аргумент до типу ***Comparable***. 

Тепер уявіть, що ви отримали клас, який не реалізує інтерфейс ***Comparable***, або реалізує, але вам не подобається, як він працює, і ви хотіли б задати для типу інший метод порівняння. Для вирішення проблеми створюється окремий клас, який реалізує інтерфейс ***Comparator***. Він містить два методи, ***compare()*** і ***equals()***. Втім, вам практично ніколи не доведеться реалізовувати ***equals()*** - хіба що при особливих вимогах по швидкодії, тому що будь-який створюваний клас неявно успадковує від класу ***Object*** метод ***equals()***.

Клас ***Collections*** містить метод ***reverseOrder()***, який створює ***Comparator*** для порядку сортування, зворотного по відношенню до природного. Він може бути застосований до ***CompType***:

``` java
//: arrays/Reverse.java 
// The Collections.reverseOrder() Comparator 
import java.util.*; 
import net.mindview.util.*; 
import static net.mindview.util.Print.*; 
 
public class Reverse { 
  public static void main(String[] args) { 
    CompType[] a = Generated.array( 
      new CompType[12], CompType.generator()); 
    print("before sorting:"); 
    print(Arrays.toString(a)); 
    Arrays.sort(a, Collections.reverseOrder()); 
    print("after sorting:"); 
    print(Arrays.toString(a)); 
  } 
} /* Output: 
before sorting: 
[[i = 58, j = 55], [i = 93, j = 61], [i = 61, j = 29] 
, [i = 68, j = 0], [i = 22, j = 7], [i = 88, j = 28] 
, [i = 51, j = 89], [i = 9, j = 78], [i = 98, j = 61] 
, [i = 20, j = 58], [i = 16, j = 40], [i = 11, j = 22] 
] 
after sorting: 
[[i = 98, j = 61], [i = 93, j = 61], [i = 88, j = 28] 
, [i = 68, j = 0], [i = 61, j = 29], [i = 58, j = 55] 
, [i = 51, j = 89], [i = 22, j = 7], [i = 20, j = 58] 
, [i = 16, j = 40], [i = 11, j = 22], [i = 9, j = 78] 
] 
*///:~ 
```

Нарешті, ви можете написати власну реалізацію ***Comparator***. У наступному прикладі об'єкти ***CompType*** порівнюються за значеннями ***j*** замість ***и***:

``` java
//: arrays/ComparatorTest.java 
// Реалізація Comparator. 
import java.util.*; 
import net.mindview.util.*; 
import static net.mindview.util.Print.*; 
 
class CompTypeComparator implements Comparator<CompType> { 
  public int compare(CompType o1, CompType o2) { 
    return (o1.j < o2.j ? -1 : (o1.j == o2.j ? 0 : 1)); 
  } 
} 
 
public class ComparatorTest { 
  public static void main(String[] args) { 
    CompType[] a = Generated.array( 
      new CompType[12], CompType.generator()); 
    print("before sorting:"); 
    print(Arrays.toString(a)); 
    Arrays.sort(a, new CompTypeComparator()); 
    print("after sorting:"); 
    print(Arrays.toString(a)); 
  } 
} /* Output: 
before sorting: 
[[i = 58, j = 55], [i = 93, j = 61], [i = 61, j = 29] 
, [i = 68, j = 0], [i = 22, j = 7], [i = 88, j = 28] 
, [i = 51, j = 89], [i = 9, j = 78], [i = 98, j = 61] 
, [i = 20, j = 58], [i = 16, j = 40], [i = 11, j = 22] 
] 
after sorting: 
[[i = 68, j = 0], [i = 22, j = 7], [i = 11, j = 22] 
, [i = 88, j = 28], [i = 61, j = 29], [i = 16, j = 40] 
, [i = 58, j = 55], [i = 20, j = 58], [i = 93, j = 61] 
, [i = 98, j = 61], [i = 9, j = 78], [i = 51, j = 89] 
] 
*///:~ 
```

### Сортування масиву

Засоби сортування дозволяють впорядкувати будь-який масив примітивів, будь-який масив об'єктів, що реалізують ***Comparable***, або асоційованих з об'єктом ***Comparator***. Наступний приклад генерує випадкові об'єкти ***String*** і сортує їх:

``` java
//: arrays/StringSorting.java 
// Sorting an array of Strings. 
import java.util.*; 
import net.mindview.util.*; 
import static net.mindview.util.Print.*; 
 
public class StringSorting { 
  public static void main(String[] args) { 
    String[] sa = Generated.array(new String[20], 
      new RandomGenerator.String(5)); 
    print("Before sort: " + Arrays.toString(sa)); 
    Arrays.sort(sa); 
    print("After sort: " + Arrays.toString(sa)); 
    Arrays.sort(sa, Collections.reverseOrder()); 
    print("Reverse sort: " + Arrays.toString(sa)); 
    Arrays.sort(sa, String.CASE_INSENSITIVE_ORDER); 
    print("Case-insensitive sort: " + Arrays.toString(sa)); 
  } 
} /* Output: 
Before sort: [YNzbr, nyGcF, OWZnT, cQrGs, eGZMm, JMRoE, suEcU, OneOE, 
dLsmw, HLGEa, hKcxr, EqUCB, bkIna, Mesbt, WHkjU, rUkZP, gwsqP, zDyCy, 
RFJQA, HxxHv] 
After sort: [EqUCB, HLGEa, HxxHv, JMRoE, Mesbt, OWZnT, OneOE, RFJQA, 
WHkjU, YNzbr, bkIna, cQrGs, dLsmw, eGZMm, gwsqP, hKcxr, nyGcF, rUkZP, 
suEcU, zDyCy] 
Reverse sort: [zDyCy, suEcU, rUkZP, nyGcF, hKcxr, gwsqP, eGZMm, dLsmw, 
cQrGs, bkIna, YNzbr, WHkjU, RFJQA, OneOE, OWZnT, Mesbt, JMRoE, HxxHv, 
HLGEa, EqUCB] 
Case-insensitive sort: [bkIna, cQrGs, dLsmw, eGZMm, EqUCB, gwsqP, hKcxr, 
HLGEa, HxxHv, JMRoE, Mesbt, nyGcF, OneOE, OWZnT, RFJQA, rUkZP, suEcU, 
WHkjU, YNzbr, zDyCy] 
*///:~ 
```

У вихідних даних алгоритму сортування ***String*** впадає в око те, що алгоритм є лексикографічним, тобто всі слова, що починаються з великих літер, передують будь-яким словами, що починаються з малої літери. Якщо ви хочете, щоб слова групувалися незалежно від регістру символів, використовуйте режим ***String.CASE_INSENSITIVE_ORDER***, як показано в останньому виклику ***sort()*** з наведеного прикладу. 

Алгоритм сортування, що використовується в стандартній бібліотеці *Java*, спроектований в розрахунку на оптимальність для сортуємого типу - швидке сортування для примітивів, надійне сортування злиттям для об'єктів. Зазвичай вам не доводиться турбуватися про швидкодію, якщо тільки профайлер не встановить, що процес сортування гальмує роботу програми. 

### Пошук в відсортованому масиві

Після того, як масив буде відсортований, ви зможете швидко знайти потрібний елемент методом ***Arrays.binarySearch()***. Хоча якщо ви спробуєте викликати ***binarySearch()*** для невідсортованого масиву, то це призведе до непередбачуваних наслідків. У наступному прикладі генератор ***RandomGenerator.Integer*** заповнює масив, після чого той же генератор використовується для отримання шуканих значень:

``` java
//: arrays/ArraySearching.java 
// Using Arrays.binarySearch(). 
import java.util.*;
import net.mindview.util.*;
import static net.mindview.util.Print.*;

public class ArraySearching {
    public static void main(String[] args) {
        Generator<Integer> gen =
                new RandomGenerator.Integer(1000);
        int[] a = ConvertTo.primitive(
                Generated.array(new Integer[25], gen));
        Arrays.sort(a);
        print("Sorted array: " + Arrays.toString(a));
        while(true) {
            int r = gen.next();
            int location = Arrays.binarySearch(a, r);
            if(location >= 0) {
                print("Location of " + r + " is " + location +
                        ", a[" + location + "] = " + a[location]);
                break; // Out of while loop 
            }
        }
    }
} /* Output:
Sorted array: [128, 140, 200, 207, 258, 258, 278, 288, 322, 429, 511, 
520, 522, 551, 555, 589, 693, 704, 809, 861, 861, 868, 916, 961, 998] 
Location of 322 is 8, a[8] = 322 
*///:~
```

Цикл ***while*** генерує випадкові значення як знайдені до тих пір, поки одне з них не буде знайдено в масиві. 

Якщо шукане значення знайдено, метод ***Arrays.binarySearch()*** повертає невід'ємні результат. В іншому випадку повертається від'ємне значення, що представляє позицію елемента при вставці (при збереженні сортування масиву). 

Якщо масив містить повторювані значення, алгоритм пошуку не дає гарантій щодо того, який саме з дублікатів буде виявлений. Алгоритм проектувався не для підтримки дублікатів, а для того, щоб переносити їх присутність. Якщо вам потрібен відсортований список без повторень елементів, використовуйте ***TreeSet*** (для збереження порядку сортування) або ***LinkedHashSet*** (для збереження порядку вставки). Ці класи автоматично беруть на себе всі деталі. Тільки в ситуаціях, критичних за швидкодією, ці класи змінюються масивами з ручним виконанням операцій. 

При сортуванні об'єктних масивів з використанням ***Comparator*** (примітивні масиви не дозволяють виконувати сортування з ***Comparator***) необхідно включати той же об'єкт ***Comparator***, що і при використанні ***binarySearch()*** (перевантаженої версії). Наприклад, програму ***StringSorting.java*** можна модифікувати для виконання пошуку:

``` java
//: arrays/AlphabeticSearch.java 
// Searching with a Comparator. 
import java.util.*; 
import net.mindview.util.*; 
 
public class AlphabeticSearch { 
  public static void main(String[] args) { 
    String[] sa = Generated.array(new String[30], 
      new RandomGenerator.String(5)); 
    Arrays.sort(sa, String.CASE_INSENSITIVE_ORDER); 
    System.out.println(Arrays.toString(sa)); 
    int index = Arrays.binarySearch(sa, sa[10], 
      String.CASE_INSENSITIVE_ORDER); 
    System.out.println("Index: "+ index + "\n"+ sa[index]); 
  } 
} /* Output: 
[bkIna, cQrGs, cXZJo, dLsmw, eGZMm, EqUCB, gwsqP, hKcxr, HLGEa, HqXum, 
HxxHv, JMRoE, JmzMs, Mesbt, MNvqe, nyGcF, ogoYW, OneOE, OWZnT, RFJQA, 
rUkZP, sgqia, slJrL, suEcU, uTpnX, vpfFv, WHkjU, xxEAJ, YNzbr, zDyCy] 
Index: 10 
HxxHv 
*///:~ 
```

Об'єкт ***Comparator*** передається перевантаженому методу ***binarySearch()*** в третьому аргументі. У наведеному прикладі успіх пошуку гарантований, так як дані значення вибираються з самого масиву.

## Резюме

У цьому розділі ви переконалися в тому, що мова *Java* надає непогану підтримку низькорівневих масивів фіксованого розміру. Такі масиви віддають перевагу продуктивності перед гнучкістю. У початковій версії *Java* низькорівневі масиви фіксованого розміру були абсолютно необхідні - не тільки тому, що проектувальники *Java*
вирішили включити в мову примітивні типи (також з міркувань швидкодії), але й тому, що підтримка контейнерів в цій версії була вкрай обмеженою.

У наступних версіях *Java* підтримка контейнерів була значно поліпшена. Зараз контейнери перевершують масиви в усіх відношеннях, крім швидкодії, хоча продуктивність контейнерів була значно поліпшена. З додаванням автоматичної упаковки і параметризації зберігання примітивів в контейнерах вимагає менших зусиль і сприяє подальшому переходу з низькорівневих масивів на контейнери. Так як параметризація відкрила доступ до типізованних контейнерів, в цьому відношенні масиви теж втратили вихідні переваги.

Як було показано в цьому розділі, параметризація погано поєднується з контейнерами. Навіть якщо вам вдасться тим чи іншим способом змусити їх працювати разом, під час компіляції ви будете отримувати попередження.

Всі ці фактори показують, що при програмуванні для останніх версій *Java* слід віддавати перевагу контейнерам перед масивами. На масиви слід переходити тільки при критичних вимогах по швидкодії - і тільки в тому випадку, якщо перехід принесе відчутну користь.
