# Тестирование кода

### Функциональное тестирование

Проведение функционального тестирования, как правило, связано с созданием специальной группы специалистов, занимающихся
тестированием. На этом этапе приложение развертывается в действующем окружении и проверяется его соответствие ТЗ (
Техническому Заданию) и предъявленным функциональным требованиям. Команда тестеровщиков использует комплекс
автоматизированных и ручных тестов.

Автоматизировать процесс функционального тестирования можно, если приложение включает API (Application Programming
Interface) - интерфейс прикладного программирования, на котором оно построено. Однако наличие интерфейса в приложении (
desktop, web) существенно снижает возможности полной автоматизации данного процесса.

### Интеграционное тестирование

Стратегия интеграционного тестирования основывается на проверке прикладного кода в окружении, близком к фактическому
окружению, но не являющимся им. Главная цель данной стратегии – убедиться в правильности взаимодействия кода с внешними
ресурсами и взаимодействия различных технологий в приложении между собой.

В интеграционном тестировании не требуется использовать фиктивные данные, как при модульном тестировании. Вместо этого в
интеграционных тестах часто используются базы данных, находящиеся в памяти, которые легко можно создавать и уничтожать
во время выполнения тестов. База данных в памяти – это самая настоящая база данных, что дает возможность проверить
правильность работы сущностей JPA. Но все же эта база данных не совсем настоящая – она лишь имитирует настоящую базу
данных для целей интеграционного тестирования.

### Модульное тестирование

Целью _модульного тестирования_ является проверка работы прикладной логики всего приложения или отдельных его частей при
разных исходных данных, и анализ правильности получаемых результатов. Несмотря на то, что цель _модульного тестирования_
выглядит простой и понятной, реализация этого типа тестирования может оказаться очень сложным и запутанным делом,
особенно при наличии «старого» кода. Основные приемы проведения модульного тестирования опираются на следующие базовые
принципы:

* внешние ресурсы не используются, т.е. недопустимо подключение к базам данных, веб-службам и т.п.;
* каждый класс имеет свой тест;
* тестируются только общедоступные методы или интерфейсы, а внутренний код тестируется за счет изменения входных данных;
* для получения данных, требуемых тестируемой логике, должны создаваться фиктивные зависимости.

При проведении модульного тестирования для создания фиктивных классов-зависимостей можно использовать простой, но мощный
фреймворк **Mockito** совместно с **JUnit**.


<details><summary>JUnit - тестирование</summary>

```java
import java.util.Comparator;
import java.util.List;

/**
 * Тестируемый класс - вычисляет максимальное значение из массива чисел
 * @author Aleksandr Kononov
 * @version 1.0
 */
public class DataService {

	// Вариант 1 (классический)
	public static int findMax(List<Integer> numbers) throws Exception {

		// Обработка пустого листа или когда значение null
		if (numbers == null || numbers.size() == 0) {
			throw new Exception("Лист объектов пустой");
		}

		// Классический алгоритм нахождения max значения
		int max = numbers.get(0);
		for (int i = 0; i < numbers.size(); i++) {
			if (max < numbers.get(i)) {
				max = numbers.get(i);
			}
		}
		return max;
	}

	// Вариант 2, через лямдба выражение (более компактный)
	public static int findMaxByStreams(List<Integer> numbers) throws Exception {
		// берем numbers, откроем stream и найдем max из элементов
		// определим порядок - naturalOrder() нахождение max
		// или reverseOrder() нахождение min
		Integer max = numbers.stream().max(Comparator.naturalOrder())
				// лямбда выражение для exception-a
				.orElseThrow(()-> new Exception("Лист объектов пустой"));
		return max;
	}
}

```

### Класс тестов для методов исходного класса DataService.

```java
import static org.assertj.core.api.Assertions.assertThat;

import java.util.Arrays;
import java.util.List;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.Arrays;
import java.util.List;

import java.util.stream.Collectors;
import java.util.stream.Stream;

import org.junit.After;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Test;

/**
 * Класс тестов для методов класса DataService
 * @author Aleksandr Kononov
 * @version 1.0
 */
public class DataServiceTest {

  @Before
  public void setUp() throws Exception {
    System.out.println("Выполнить перед каждым тестом.");
  }

  @After
  public void tearDown() throws Exception {
    System.out.println("Выполнить после каждого теста.\n");
  }

  @BeforeClass
  public static void setUpAllOnce() throws Exception {
    System.out.println("---------------------------------------------" +
            "\nВыполнить один раз перед запуском всех тестов.");
  }

  @AfterClass
  public static void tearDownAllOnce() {
    System.out.println("Выполнить один раз после выполнения всех тестов.");
  }

  // Вариант 1. Протестируем метод нахождения max
  @Test
  public void testFindMax() throws Exception {
    // Создадим лист объектов
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

    // Обратимся к тестируемому методу и подадим на вход наш лист объектов
    int max = DataService.findMax(numbers);

    // Тестируем - подаем max и оцениваем, равно ли 8
    assertThat(max).isEqualTo(8) // цепочки сравнений
            .isNotNull(); // цепочки сравнений и так далее ...
    System.out.println("testFindMax() - пройден");
  }

  // Вариант 1. Протестируем обработку исключения
  @Test(expected = Exception.class) // ожидать какой класс exception-а будет выброшен
  public void testFindMaxException() throws Exception {
    // Создадим пустой лист объектов
    List<Integer> numbers = Arrays.asList();

    // Переменную убрали, поскольку нам важно исполнение функции
    DataService.findMax(numbers);
  }

  // Вариант 1. Протестируем обработку исключения
  @Test(expected = NullPointerException.class) // ожидать какой класс exception-а будет выброшен
  public void testFindMaxException_Null() throws Exception {
    // Создадим лист объектов со значением null
    List<Integer> numbers = Arrays.asList(null);

    // Переменную убрали, поскольку нам важно исполнение функции
    DataService.findMax(numbers);
  }

  // Вариант 2. Протестируем метод нахождения max
  @Test
  public void testFindMaxByStreams() throws Exception {
    // Создадим лист объектов
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

    // Обратимся к тестируемому методу и подадим на вход наш лист объектов
    int max = DataService.findMaxByStreams(numbers);

    // Тестируем - подаем max и оцениваем, равно ли 8
    assertThat(max).isEqualTo(8);
    System.out.println("testFindMaxByStreams() - пройден");
  }

  // Вариант 2. Протестируем обработку исключения
  @Test(expected = Exception.class) // ожидать какой класс exception-а будет выброшен
  public void testFindMaxByStreamsException() throws Exception {
    // Создадим пустой лист объектов
    List<Integer> numbers = Arrays.asList();

    // Переменную убрали, поскольку нам важно исполнение функции
    DataService.findMaxByStreams(numbers);
  }

  // Вариант 2. Протестируем обработку исключения
  @Test(expected = NullPointerException.class) // ожидать какой класс exception-а будет выброшен
  public void testFindMaxByStreamsException_Null_List() throws Exception {
    // Создадим лист объектов со значением null
    List<Integer> numbers = null;

    // Переменную убрали, поскольку нам важно исполнение функции
    DataService.findMaxByStreams(numbers);
  }

  // Вариант 2. Тест - ПРОИЗВОДИТЕЛЬНОСТИ.
  @Test(timeout = 100) // выделим этому тесту 100 мили секунд
  public void performanceFindMaxByStreams() throws Exception {
    // Ввыделим переменную, создадим стрим
    // будем итерировать от 0 c приращением на 1 и присвоим листу объектов
    List<Integer> numbers = Stream.iterate(0, n -> n + 1)
            .limit(2000) // ограничим до 2000
            .collect(Collectors.toList()); // вернем все это в Лист

    // Переменную убрали, поскольку нам важно исполнение функции
    DataService.findMaxByStreams(numbers);
    System.out.println("performanceFindMaxByStreams() - пройден");
  }
}

```

```
Вывод:
---------------------------------------------
Выполнить один раз перед запуском всех тестов.
Выполнить перед каждым тестом.
testFindMax() - пройден
Выполнить после каждого теста.

Выполнить перед каждым тестом.
testFindMaxException() - пройден
Выполнить после каждого теста.

Выполнить перед каждым тестом.
performanceFindMaxByStreams() - пройден
Выполнить после каждого теста.

Выполнить перед каждым тестом.
Выполнить после каждого теста.

Выполнить перед каждым тестом.
Выполнить после каждого теста.

Выполнить перед каждым тестом.
testFindMaxByStreams() - пройден
Выполнить после каждого теста.

Выполнить перед каждым тестом.
Выполнить после каждого теста.

Выполнить один раз после выполнения всех тестов.
```

</details>


<details><summary>Mockito фреймворк</summary>

Фреймворк _Mockito_ предоставляет ряд возможностей для создания заглушек вместо реальных классов или интерфейсов при
написании _JUnit_ тестов.

определить в зависимостях:

```xml

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.10.0</version>
    <scope>test</scope>
</dependency>
```

возможности _Mockito_:

* создание заглушек для классов и интерфейсов;
* проверка вызыва метода и значений передаваемых методу параметров;
* использование концепции «частичной заглушки», при которой заглушка создается на класс с определением поведения,
  требуемое для некоторых методов класса;
* подключение к реальному классу «шпиона» _spy_ для контроля вызова методов.

Аннотации _Mockito_:

    @Mock   - создает экземпляр заглушку класса (тоже самое делает статический метод mock())
    @Spy    - создает шпионский объект, следит за реальными методами.
    @Captor - служит для захвата аргументов совместно с методом verify()
    @InjectMocks - создает экземпляр объекта тестирования и вводит поля, аннотированные с помощью @Mock или @Spy в закрытые поля объекта тестирования.

Методы _mock_ объекта возвращают значения по умолчанию:

    false - для boolean,
    0     - для int,
    null  - для остальных объектов,
    и пустые коллекции.

_Mockito_ разница между _doReturn()_ и _when()_:

Оба подхода ведут себя по-разному, если вы используете шпионский объект (с аннотацией @Spy ) вместо макета (с аннотацией
@Mock ):

* when(...) thenReturn(...) делает реальный вызов метода непосредственно перед тем, как будет возвращено указанное
  значение. Поэтому, если вызываемый метод вызывает исключение, вам приходится иметь с ним дело / издеваться над ним и
  т. д. Конечно, вы все равно получите свой результат (то, что вы определяете в thenReturn(...) )
* doReturn(...) when(...) вообще не вызывает метод .

#### создания заглушки Mockito

Чтобы создать _Mockito_ объект можно использовать либо аннотацию _@Mock_, либо метод _mock_.

```java
// Пример
public interface ICalculator {
    public double add(double d1, double d2);

    public double subtract(double d1, double d2);

    public double multiply(double d1, double d2);

    public double divide(double d1, double d2);
}

//---------------------------------------------------
public class Calculator {
    ICalculator icalc;

    public Calculator(ICalculator icalc) {
        this.icalc = icalc;
    }

    public double add(double d1, double d2) {
        return icalc.add(d1, d2);
    }

    public double subtract(double d1, double d2) {
        return icalc.subtract(d1, d2);
    }

    public double multiply(double d1, double d2) {
        return icalc.multiply(d1, d2);
    }

    public double divide(double d1, double d2) {
        return icalc.divide(d1, d2);
    }

    public double double15() {
        return 15.0;
    }
}

// Тест
@RunWith(MockitoJUnitRunner.class)
public class Test_Mockito {
    @Mock
    ICalculator mcalc;

    // используем аанотацию @InjectMocks для создания mock объекта
    @InjectMocks
    Calculator calc = new Calculator(mcalc);

    @DisplayName("Определение поведения - when(mock).thenReturn(value)")
    @Test
    public void CalcAddTest() {
        // определяем поведение калькулятора для операции сложения
        when(calc.add(10.0, 20.0)).thenReturn(30.0);

        // проверяем действие сложения
        assertEquals(calc.add(10, 20), 30.0, 0);
        // проверяем выполнение действия
        verify(mcalc).add(10.0, 20.0);

        // определение поведения с использованием doReturn 
        doReturn(15.0).when(mcalc).add(10.0, 5.0);

        // проверяем действие сложения
        assertEquals(calc.add(10.0, 5.0), 15.0, 0);
        verify(mcalc).add(10.0, 5.0);
    }

    @DisplayName("Подсчет количества вызовов - atLeast, atLeastOnce, atMost, times, never")
    @Test
    public void CountMethodTest() {
        // определяем поведение (результаты)
        when(mcalc.subtract(15.0, 25.0)).thenReturn(10.0);
        when(mcalc.subtract(35.0, 25.0)).thenReturn(-10.0);

        // вызов метода subtract и проверка результата
        assertEquals(calc.subtract(15.0, 25.0), 10, 0);
        assertEquals(calc.subtract(15.0, 25.0), 10, 0);

        assertEquals(calc.subtract(35.0, 25.0), -10, 0);

        // проверка вызова методов
        verify(mcalc, atLeastOnce()).subtract(35.0, 25.0);
        verify(mcalc, atLeast(2)).subtract(15.0, 25.0);

        // проверка - был ли метод вызван 2 раза?
        verify(mcalc, times(2)).subtract(15.0, 25.0);
        // проверка - метод не был вызван ни разу
        verify(mcalc, never()).divide(10.0, 20.0);

        /* Если снять комментарий со следующей проверки, то
         * ожидается exception, поскольку метод "subtract"
         * с параметрами (35.0, 25.0) был вызван 1 раз
         */
        // verify(mcalc, atLeast (2)).subtract(35.0, 25.0);

        /* Если снять комментарий со следующей проверки, то
         * ожидается exception, поскольку метод "subtract"
         * с параметрами (15.0, 25.0) был вызван 2 раза, а
         * ожидался всего один вызов
         */
        // verify(mcalc, atMost (1)).subtract(15.0, 25.0);
    }

    @DisplayName("Обработка исключений - when(mock).thenThrow()")
    @Test
    public void CalcRuntimeExceptionTest() {
        when(mcalc.divide(15.0, 3)).thenReturn(5.0);

        assertEquals(calc.divide(15.0, 3), 5.0, 0);
        // проверка вызова метода
        verify(mcalc).divide(15.0, 3);

        // создаем исключение
        RuntimeException exception = new RuntimeException("Division by zero");
        // определяем поведение
        doThrow(exception).when(mcalc).divide(15.0, 0);

        assertEquals(calc.divide(15.0, 0), 0.0, 0);
        verify(mcalc).divide(15.0, 0);
    }

    @DisplayName("Использование шпиона spy на реальных объектах")
    @Test
    public void testSpy() {
        Calculator scalc = spy(new Calculator());
        when(scalc.double15()).thenReturn(23.0);

        // вызов метода реального класса
        scalc.double15();
        // проверка вызова метода
        verify(scalc).double15();

        // проверка возвращаемого методом значения
        assertEquals(23.0, scalc.double15(), 0);
        // проверка вызова метода не менее 2-х раз
        verify(scalc, atLeast(2)).double15();
    }

    @DisplayName("Использование интерфейса org.mockito.stubbing.Answer<T>")
    @Test
    public void testThenAnswer() {
        // определение поведения mock для метода с параметрами
        when(mcalc.add(11.0, 12.0)).thenAnswer(answer);
        assertEquals(calc.add(11.0, 12.0), 23.0, 0);
    }

    // метод обработки ответа
    private Answer<Double> answer = new Answer<Double>() {
        @Override
        public Double answer(InvocationOnMock invocation) throws Throwable {
            // получение объекта mock 
            Object mock = invocation.getMock();
            System.out.println("mock object : " + mock.toString());

            // аргументы метода, переданные mock
            Object[] args = invocation.getArguments();
            double d1 = (double) args[0];
            double d2 = (double) args[1];
            double d3 = d1 + d2;
            System.out.println("" + d1 + " + " + d2);

            return d3;
        }
    };

    @DisplayName("Проверка вызова метода с задержкой, timeout")
    @Test
    public void testTimout() {
        // определение результирующего значения mock для метода
        when(mcalc.add(11.0, 12.0)).thenReturn(23.0);
        // проверка значения
        assertEquals(calc.add(11.0, 12.0), 23.0, 0);

        // проверка вызова метода в течение 10 мс
        verify(mcalc, timeout(100)).add(11.0, 12.0);
    }

    @DisplayName("Использование в тестах java классов")
    @Test
    public void testJavaClasses() {
        // создание объекта mock
        Iterator<String> mis = mock(Iterator.class);
        // формирование ответов
        when(mis.next()).thenReturn("Привет").thenReturn("Mockito");
        // формируем строку из ответов
        String result = mis.next() + ", " + mis.next();
        // проверяем
        assertEquals("Привет, Mockito", result);

        Comparable<String> mcs = mock(Comparable.class);
        when(mcs.compareTo("Mockito")).thenReturn(1);
        assertEquals(1, mcs.compareTo("Mockito"));

        Comparable<Integer> mci = mock(Comparable.class);
        when(mci.compareTo(anyInt())).thenReturn(1);
        assertEquals(1, mci.compareTo(5));
    }

}
```

</details>