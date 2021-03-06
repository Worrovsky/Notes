Наследование 

class A extends B{
    // ...
}

A - подкласс
B - суперкласс
в суперклассе определяются общие методы,
     в подклассе - более специфичные

переопределение методов

подкласс не имеет доступа к закрытым данным суперкласса

ключевое слово super:
    - указание на вызов метода именно суперкласса, 
        когда метод определен с одинаковым именем в супер- и подклассах:
        super.someMethod() 
    - вызов конструктора суперкласса
        super(...)
    ср. с this - ссылка на неявный параметр + вызов другого конструктора

супер- и подклассы в массивах:
    
    a = new A();
    b = new B();
    A[] array = new A[2];
    array[0] = a;
    array[1] = b; // ошибки не будет, хотя массив объявлен через А
    for (A elem : array){
        elem.someMethod();  // в первом случае вызывается метод суперкласса
                            // во втором - подкласса (полиморфизм + динамич. связывание)
        elem.methodFromSubclass(); // здесь будет ошибка, т.к. elem объявлен типом A
                                   // а метод methodFromSubclass не объявлен в А
    }   


принцип подстановки:
    любой объект подкласса можно использовать вместо суперкласса
    пример
        A var;
        var = new A();  // обычное
        var = new B();  // типы не совпадают, но так можно: В - подкласс А
                        // наоборот нельзя

Динамическое связывание:
    1. создается таблица сигнатур методов класса и его суперклассов
    2. проверяется сигнатура вызываемого метода, ищется соответствие
        если только одно - он и вызывается, если нет вообще - ошибка
    3. если метод закрытый (private), статический (static), 
        терминальный(final) или конструктор тогда точно известно что 
        вызывать (статическое связывание). Иначе динамическое, причем 
        будет вызвана версия метода, соответствующая фактическому типу 
        объекта в момент вызова.

Прекращение наследования
    
    терминальные классы, не могут быть суперклассами
        final class T{...}

    терминальные методы, не могут быть переопределены в подклассах
        public final String getName(){...}

Приведение типов для классов
    A var = new B();
    ...
    B var1 = (B) var;  // обычно когда нужно использовать доп. поля/методы подкласса

    общее правило: сужать возможности переменной можно, расширение - указывать явно
        A var = new B();    // здесь сузили возможности: хоть по факту var - это класс B,
                            // доступ - только к элементам класса А
        B var1 = (B) var;   // здесь расширения, явно указывается приведение типов

    суперкласс преобразовать приведением типов в подкласс нельзя
        B b = (B) new A();// error

    проверка на тип:
        if (b instanceof B) {...}
        если выражение = null, результат проверки = false

Абстрактные классы/методы

    в суперклассе можно не реализовывать метод, а объявить его абстрактным 
        (как прототип для последующей реализации в подклассах)
        public abstract void someMethod(){...}
    класс с абстрактными методами можно объявить абстрактным
        abstract class AClass{...}
    абстрактные классы могут содержать обычные методы

    подклассы могут реализовывать абстрактные методы (подкласс - не абстрактный)
        или оставлять не реализованными (подкласс - сам абстрактный)

    класс можно объявить абстрактным, даже если он не содержит абстрактных методов

    создать объект абстрактного класса нельзя

    можно создавать переменные с типом абстрактного класса, присваивая им 
        значения подклассов. При этом можно вызывать "абстрактные" методы (уже реализованные в подклассах)

ключевое слово protected:
    поле: доступ имеют методы подкласса (но не доступ к полям суперкласса)
    методы: подклассы имеют доступ
    protected дает доступ в пределах пакета
    по умолчанию - в пределах пакета


 
класс Object
    любой класс - подкласс Object
    при создании класса явно не указывается class NewCl extends Object {...}
    переменной типа Object можно присвоить значение любого типа
    массивы также наследники Object, независимо от типа элементов

    метод equals()
        в классе Object проверяется ссылаются ли переменные на один объект
        в других классах равенство может означать одинаковое состояние разных объектов
        тогда - переопределять метод

        объявление: 
            public boolean equals(Object otherObject){...} // параметр - типа Object
        общий алгоритм:
            - быстрая проверка ссылок
                if (this == otherObject) return true;
            - проверка на null
                if (otherObject == null) return false;
            - проверка на принадлежность одному классу:
                if (getClass() != otherObject.getClass()) return false;
                    здесь жесткая проверка на класс (суперкласс и подкласс - уже разные)  
                if !((otherObject instanceof ClassName)) return false;
                    этим вариантом можно сравнивать подкласс с суперклассом
                    (но не наоборот, что нарушает принцип "если а=б, то и б=а")
            - приведение типа
            - проверка полей
                простые типы - по равенству, сложные - через equals()
            - вызов super.equals(..) если надо

    дескриптор @Override:
        указание на то, что метод переопределяется
        проверка компилятором: в суперклассах будет искаться метод с такой же сигнатурой
        при отсутствии - ошибка. 

    метод hashCode()
        возвращает целое число - хеш-код
        используется например при создании хеш-таблиц
        в класса Object вычисляется на основе адреса памяти
        можно переопределять, если нужно чтобы разные объекты с 
            одинаковыми полями имели одинаковый  х-код
        пример: 7 * getName().hashCode() + 11 * getSalary().hashCode()

        совместимость с методом equals(): если равны объекты, то и их коды д.б. равны

    метод toString():
        строковое представление объекта
        в класса Object - на основе адреса памяти
        в собственных классах - переопределять
        используется неявно при конкатенации, в System.out.println() и подобных

    метод getClass():
        возвращает объект класса Class
        методы: String getName(), Class getSuperClass()

универсальный класс ArrayList
    
    обычные массивы - жестко задан размер при создании
    ArrayList - динамический

    объявление:
        ArrayList <String> myArray = new ArrayList<String>();
        в <> указывается тип элементов
        можно с указанием кол-ва 
        new ArrayList<String>(100);

    объем памяти, выделенной списочному массиву меняется при заполнении:
        если при добавлении очередного элемента - не хватает, выделяется 
        больший объем, в него копируется текущий массив
        в отличие от простого массива, в котором память выделяется один раз при создании

    методы:
        add(e) -                добавляет элемент в конец массива
        add(i, e)               добавляет в позицию i, последующие сдвигаются
        size() -                возвращает фактическое кол-во элементов
        ensureCapasity(100) -   выделяет память на 100 объектов (нет доп. расходов при увеличении массива)
        trimToSize() -          освобождает свободную память, сокращая внутренний массив
                            до текущего размера. Когда точно известно, что не будет увеличен.
        set(i, elem) -          установка i-го элемента (если не существует - ошибка)
        get(i) -                получение i-го элемента
        можно через цикл for each

        toArray(a)              копирование в простой массив а (предварительно создается)               
        remove(i)               удаление i-го, сдвиг остальных, возвращает удаленный

        вопросы быстродействия при вставке/удалении

Интерфейсные классы (классы-оболочки)

    аналоги простых типов
    когда нужен тип-класс, например в ArrayList (тип int нельзя)        
    примеры: Number, Integer, Byte, Float  и т. п.
        ArrayList <Integer> list = new ArrayList<Integer>();

    значения простых типов автоматически преобразуются в интерфейсные:
        Integer i = 3; // Integer i = new Integer(3)
        list.add(3); // list.add(new Integer(3))
    и наоборот
        int n = list.get(i); // int n = list.get(i).intValue()

    являются терминальными

    сравнение:
        оператор == для разных объектов с одинаковым значением даст false 
            (скорее всего: компилятор может один объект создавать)
        использовать equals

    статические методы по преобразованию строковых значений
        parseInt, valueOf

методы с произвольным количеством параметров

    public void f(int...value){

    }

    f(1, 3, 4);

    при вызове параметр value получит массив int[] из параметров