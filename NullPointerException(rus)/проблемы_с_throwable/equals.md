## `==` vs `equals()` - фундаментальная разница

### `==` - сравнение по ссылке (identity)
```java
Throwable t1 = new Throwable("error");
Throwable t2 = t1;          // та же самая ссылка
Throwable t3 = new Throwable("error"); // другой объект, то же сообщение

System.out.println(t1 == t2);  // true - одна и та же ячейка памяти
System.out.println(t1 == t3);  // false - разные ячейки памяти
```

### `equals()` - сравнение по содержимому (equality)
```java
String s1 = new String("hello");
String s2 = new String("hello");

System.out.println(s1 == s2);      // false - разные объекты
System.out.println(s1.equals(s2)); // true - одинаковое содержимое
```

## Почему в Throwable используется `==` (Identity)

### Проблема с `equals()` в Throwable:
```java
Throwable t1 = new Throwable("error");
Throwable t2 = new Throwable("error"); 

// Если бы использовалось equals():
Set<Throwable> set = new HashSet<>(); // использует equals()
set.add(t1);
set.add(t2);

System.out.println(set.contains(t1)); // true
System.out.println(set.contains(t2)); // true - ОПАСНО!
// Оба исключения считаются "одинаковыми" по equals(), 
// но это РАЗНЫЕ объекты в памяти!
```

### Решение - IdentityHashMap:
```java
// IdentityHashMap использует == вместо equals()
Set<Throwable> dejaVu = Collections.newSetFromMap(new IdentityHashMap<>());

Throwable t1 = new Throwable("error");
Throwable t2 = new Throwable("error"); // то же сообщение, но другой объект
Throwable t3 = t1; // та же ссылка

dejaVu.add(t1);

System.out.println(dejaVu.contains(t1)); // true - тот же объект
System.out.println(dejaVu.contains(t2)); // false - другой объект
System.out.println(dejaVu.contains(t3)); // true - та же ссылка
```

## Практический пример с циклическими ссылками:

```java
public class IdentityVsEquals {
    public static void main(String[] args) {
        // Создаем циклическую ссылку
        Throwable t1 = new Throwable("First");
        Throwable t2 = new Throwable("Second");
        t1.initCause(t2);
        t2.initCause(t1); // цикл: t1 -> t2 -> t1
        
        // Тестируем разные подходы
        testWithEquals();   // Неправильный подход
        testWithIdentity(); // Правильный подход
    }
    
    static void testWithEquals() {
        System.out.println("=== HashSet (использует equals()) ===");
        Set<Throwable> badSet = new HashSet<>();
        
        Throwable t1 = new Throwable("Error");
        Throwable t2 = new Throwable("Error"); // Другой объект, то же сообщение
        
        badSet.add(t1);
        System.out.println("Содержит t1: " + badSet.contains(t1)); // true
        System.out.println("Содержит t2: " + badSet.contains(t2)); // true - ЛОЖНОЕ СРАБАТЫВАНИЕ!
    }
    
    static void testWithIdentity() {
        System.out.println("\n=== IdentityHashMap (использует ==) ===");
        Set<Throwable> goodSet = Collections.newSetFromMap(new IdentityHashMap<>());
        
        Throwable t1 = new Throwable("Error");
        Throwable t2 = new Throwable("Error"); // Другой объект, то же сообщение
        Throwable t3 = t1; // Та же ссылка
        
        goodSet.add(t1);
        System.out.println("Содержит t1: " + goodSet.contains(t1)); // true
        System.out.println("Содержит t2: " + goodSet.contains(t2)); // false - КОРРЕКТНО!
        System.out.println("Содержит t3: " + goodSet.contains(t3)); // true - КОРРЕКТНО!
    }
}
```

## Почему именно `==` важно для обнаружения циклических ссылок:

```java
Throwable t1 = new Throwable("Main error");
Throwable t2 = new Throwable("Cause error"); 
Throwable t3 = new Throwable("Main error"); // Случайно такое же сообщение

t1.initCause(t2);
t2.initCause(t1); // Создаем циклическую ссылку t1 -> t2 -> t1

// При обходе с IdentityHashMap:
Set<Throwable> visited = Collections.newSetFromMap(new IdentityHashMap<>());

visited.add(t1);    // Добавили объект t1
visited.add(t2);    // Добавили объект t2  
// При переходе от t2 обратно к t1:
visited.contains(t1); // true - обнаружили цикл!

// А если бы использовали HashSet:
Set<Throwable> badVisited = new HashSet<>();
badVisited.add(t1);
badVisited.add(t2);
// Случайно добавим объект с таким же сообщением:
badVisited.add(t3); // t3 имеет то же сообщение "Main error"

// Проблема: t1 и t3 считаются "одинаковыми" по equals()
badVisited.contains(t1); // true - но это может быть ЛОЖНОЕ срабатывание!
```

## Ключевые отличия в таблице:

| Аспект | `==` (Identity) | `equals()` (Equality) |
|--------|-----------------|---------------------|
| **Что сравнивает** | Ссылки на объекты | Содержимое объектов |
| **Когда true** | Один и тот же объект в памяти | Объекты "равны" по логике |
| **Скорость** | Быстрее (просто сравнение указателей) | Медленнее (может быть сложная логика) |
| **Использование в Throwable** | Для обнаружения циклических ссылок | Для сравнения сообщений/содержимого |

## Вывод:

**`==` гарантирует, что мы отслеживаем именно те же самые объекты в памяти**, а не просто объекты с одинаковым содержимым. Это критически важно для обнаружения настоящих циклических ссылок, когда объект ссылается сам на себя через цепочку причин.

Вот почему в `Throwable.printStackTrace()` используется `IdentityHashMap` - чтобы точно отслеживать, какие конкретные объекты исключений мы уже посетили, а не просто исключения с одинаковыми сообщениями.