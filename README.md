### **Структуры**

*Структура* (struct) группирует поля в единую запись. В Go нет классов и объектов, так что структура — наиболее близкий аналог объекта в питоне и js.

Объявим тип `person` на основе структуры с полями `name` и `age`:

```go
type person struct {
    name string
    age  int
}
```

Так создается новая структура типа `person`:

```go
bob := person{"Bob", 20}
fmt.Println(bob)
// {Bob 20}
```

Можно явно указать названия полей:

```go
alice := person{name: "Alice", age: 30}
fmt.Println(alice)
// {Alice 30}
```

Если не указать поле, оно получит нулевое значение:

```go
fred := person{name: "Fred"}
fmt.Println(fred)
// {Fred 0}
```

Оператор `&` возвращает указатель на структуру:

```go
annptr := &person{name: "Ann", age: 40}
fmt.Println(annptr)
// &{Ann 40}
```

В Go иногда создают новые структуры через функцию-конструктор с префиксом `new`:

```go
func newPerson(name string) *person {
    p := person{name: name}
    p.age = 42
    return &p
}
```

Функция возвращает указатель на локальную переменную — это нормально. Go распознает такие ситуации, и выделяет память под структуру в куче (heap) вместо стека (stack), так что структура продолжит существовать после выхода из функции.

```
john := newPerson("John")
fmt.Println(john)
// &{John 42}
```

Если функция-конструктор возвращает саму структуру, а не указатель — удобно использовать префикс `make` вместо `new`:

```go
func makePerson(name string) person {
    p := person{name: name}
    p.age = 42
    return p
}
```

> В реальности чаще не заморачиваются и всегда используют префикс new вне зависимости от того, что возвращает конструктор — значение или указатель на него. Но на курсе я буду соблюдать это разделение: make — значение, new — указатель.
> 

Доступ к полям структуры — через точку:

```go
sean := person{name: "Sean", age: 50}
fmt.Println(sean.name)
// Sean
```

Чтобы получить доступ к полям структуры через указатель, не обязательно разыменовывать его через `*`. Эти два варианта эквивалентны:

```go
sven := &person{name: "Sven", age: 50}
fmt.Println((*sven).age)
// 50
fmt.Println(sven.age)
// 50
```

Поля структуры можно изменять:

```go
sven.age = 51
fmt.Println(sven.age)
// 51
```

[песочница](https://go.dev/play/p/j99vztQHYg2)

### **Составные структуры**

Структуры могут включать другие структуры:

```go
type person struct {
    firstName string
    lastName  string
}

type book struct {
    title  string
    author person
}
```

```go
b := book{
    title: "The Majik Gopher",
    author: person{
        firstName: "Christopher",
        lastName:  "Swanson",
    },
}
fmt.Println(b)
// {The Majik Gopher {Christopher Swanson}}
```

Если вложенная структура не представляет самостоятельной ценности, можно даже не объявлять отдельный тип:

```go
type user struct {
    name  string
    karma struct {
        value int
        title string
    }
}
```

```go
u := user{
    name: "Chris",
    karma: struct {
        value int
        title string
    }{
        value: 100,
        title: "^-^",
    },
}
fmt.Printf("%+v\n", u)
// {name:Chris karma:{value:100 title:^-^}}
```

Благодаря шаблону `%+v`, `Printf()` печатает структуру вместе с названиями полей.

Поле структуры может ссылаться на другую структуру:

```go
type comment struct {
    text   string
    author *user
}
```

```go
chris := user{
    name: "Chris",
}
c := comment{
    text:   "Gophers are awesome!",
    author: &chris,
}
fmt.Printf("%+v\n", c)
// {text:Gophers are awesome! author:0xc0000981e0}
```

[песочница](https://go.dev/play/p/w91APw8y4O2)

### **Методы**

Go позволяет определять *методы* на типах.

Метод отличается от обычной функции специальным параметром — *получателем*. В определении метода получатель указывается сразу после ключевого слова `func`. В данном случае — получатель типа `rect`:

```go
type rect struct {
    width, height int
}

func (r rect) area() int {
    return r.width * r.height
}
```

Метод вызывается для получателя через точку, как в питоне или js:

```go
r := rect{width: 10, height: 5}
fmt.Println("rect area:", r.area())
// rect area: 50
```

Получателем может быть не значение заданного типа, а указатель на это значение:

```go
type circle struct {
    radius int
}

func (c *circle) area() float64 {
    return math.Pi * math.Pow(float64(c.radius), 2)
}
```

```go
cptr := &circle{radius: 5}
fmt.Println("circle area:", cptr.area())
// circle area: 78.54
```

При вызове метода Go автоматически преобразует значение получателя в указатель или указатель в значение, как того требует определение метода. Любой из перечисленных вариантов будет работать:

```go
rptr := &r
r.area()
rptr.area()

c := *cptr
c.area()
cptr.area()
```

Считается хорошим тоном во всех методах использовать или только значение, или только указатель, но не смешивать одно с другим. Обычно используют указатель: так Go не приходится копировать всю структуру, а метод может ее изменить.

```go
// Если метод принимает получателя как значение, изменить его не получится
func (r rect) scale(factor int) {
    r.width *= factor
    r.height *= factor
}

fmt.Println("rect before scaling:", r)
// rect before scaling: {10 5}

r.scale(2)

fmt.Println("rect after scaling:", r)
// rect after scaling: {10 5}
```

```go
// Если метод принимает получателя как указатель, его можно изменить
func (c *circle) scale(factor int) {
    c.radius *= factor
}

fmt.Println("circle before scaling:", c)
// circle before scaling: {5}

c.scale(2)

fmt.Println("circle after scaling:", c)
// circle after scaling: {10}

```

Вопрос «значение или указатель для получателя» так популярен, что у сообщества есть [отдельный гайдлайн](https://go.dev/wiki/CodeReviewComments#receiver-type) на эту тему.

[песочница](https://go.dev/play/p/IHQozTj86Wf)

### **Определяемые типы**

На предыдущем шаге мы создали структурный тип и методы для него. Но новый тип не обязательно создавать на основе структуры — можно использовать любые базовые типы.

Создадим тип «ИНН» на основе строки:

```go
type inn string
```

Тип `inn` (он называется *определяемым типом*, defined type) получил свойства базового типа `string`. Добавим ему новое поведение с помощью метода:

```go
func (id inn) isValid() bool {
    if len(id) != 12 {
        return false
    }
    for _, char := range id {
        if !unicode.IsDigit(char) {
            return false
        }
    }
    return true
}
```

```go
inn1 := inn("111201284667")
fmt.Println("inn", inn1, "is valid:", inn1.isValid())
// inn 111201284667 is valid: true

inn2 := inn("ohmyinn12345")
fmt.Println("inn", inn2, "is valid:", inn2.isValid())
// inn ohmyinn12345 is valid: false

```

Это чем-то похоже на наследование, но механизм более примитивный. Если создать новый определяемый тип на основе `inn` — он унаследует структуру и свойства `inn`, но не методы:

```go
type otherid inn
```

```go
other := otherid("111201284667")
fmt.Println("other inn", other, "is valid:", other.isValid())
// ОШИБКА: other.isValid undefined
```

[песочница](https://go.dev/play/p/qHqsPg9JHj6)

### **Композиция**

В Go нет наследования. Вместо него активно используется композиция — когда новое поведение собирают из кирпичиков существующего.

Есть тип «счетчик»:

```go
type counter struct {
    value uint
}
```

Его можно увеличивать на единицу:

```go
func (c *counter) increment() {
    c.value++
}
```

Или на указанное число:

```go
func (c *counter) incrementDelta(delta uint) {
    c.value += delta
}
```

Мы хотим замерять использование сервисов. Чтобы не дублировать существующие функции, добавим счетчик в тип «использование сервиса»:

```go
type usage struct {
    service string
    counter counter
}

func makeUsage(service string) usage {
    return usage{service, counter{}}
}
```

Будем мерить использование сервиса, увеличивая его счетчик:

```go
usage := makeUsage("find")
usage.counter.increment()
usage.counter.increment()
usage.counter.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.counter.value)
// find usage: 3
```

Для типа «просмотры страницы» тоже добавим счетчик:

```go
type pageviews struct {
    url *url.URL
    counter counter
}

func makePageviews(uri string) pageviews {
    u, err := url.Parse(uri)
    if err != nil {
        log.Fatal(err)
    }
    return pageviews{u, counter{}}
}
```

И будем мерить просмотры:

```go
pv := makePageviews("/doc/find")
pv.counter.incrementDelta(100)
fmt.Printf("%s views: %d\n", pv.url, pv.counter.value)
// /doc/find views: 100
```

[песочница](https://go.dev/play/p/kgFKZlGFRTJ)

### **Встраивание**

Все хорошо, но несколько неудобно было писать `usage.counter.increment()` на предыдущем шаге. По-хорошему, `usage`  *расширяет* `counter` — отношение между ними больше похоже на наследование, чем на композицию. В Go в таких случаях используют *встраивание* (embedding). Посмотрим, как оно работает.

Есть тип «счетчик», такой же, как на предыдущем шаге:

```go
type counter struct {
    value uint
}
func (c *counter) increment() {
    c.value++
}
func (c *counter) incrementDelta(delta uint) {
    c.value += delta
}
```

Мы хотим замерять использование сервисов. *Встроим* счетчик в тип «использование сервиса»:

```go
type usage struct {
    service string
    counter
}

func makeUsage(service string) usage {
    return usage{service, counter{}}
}
```

Благодаря встраиванию, поля и методы счетчика доступны прямо на `usage`, без обращения к полю `counter`:

```go
usage := makeUsage("find")
usage.increment()
usage.increment()
usage.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.value)
// find usage: 3
```

Аналогично с типом «просмотры страниц»:

```go
type pageviews struct {
    url *url.URL
    counter
}

func makePageviews(uri string) pageviews {
    u, err := url.Parse(uri)
    if err != nil {
        log.Fatal(err)
    }
    return pageviews{u, counter{}}
}
```

Поля и методы счетчика доступны прямо на `pageviews`:

```go
pv := makePageviews("/doc/find")
pv.incrementDelta(100)
fmt.Printf("%s views: %d\n", pv.url, pv.value)
// /doc/find views: 100
```

[песочница](https://go.dev/play/p/E_7DlwchsjN)

Есть тип «счетчик»:

```go
type counter struct {
    value uint
}

func (c *counter) increment() {
    c.value++
}
```

Можно создать значение `c` типа `counter` и вызвать метод `c.increment`:

```go
c := new(counter)

c.increment()
c.increment()
c.increment()

fmt.Println(c.value)
// 3
```

А можно вызвать метод `c.increment` как функцию:

```go
c := new(counter)
inc := c.increment

inc()
inc()
inc()

fmt.Println(c.value)
// 3
```

Здесь функция `inc` работает как замыкание — она имеет доступ к внутренним полям `c`, хотя снаружи они не видны.

Такая штука называется *метод-значение* (method value). Используется редко. Но может пригодиться, чтобы разрешить клиенту вызывать метод структуры, не давая при этом доступ к ее полям.

[песочница](https://go.dev/play/p/TX2ZEucLYMx)

### **Метод-выражение**

Можно сделать еще более причудливый финт. Использовать метод вообще без привязки к конкретному значению, как обычную функцию:

```go
inc := (*counter).increment

first := new(counter)
inc(first)
inc(first)
inc(first)

second := new(counter)
inc(second)

fmt.Println(first.value)
// 3
fmt.Println(second.value)
// 1
```

Здесь функция `inc` принимает первым аргументом получателя метода — значение `x` типа `*counter` — и дальше работает как метод, увеличивая значение `x.value`.

Такая штука называется *метод-выражение* (method expression). На практике встречается еще реже, чем метод-значение.

[песочница](https://go.dev/play/p/s8tstQne3lV)

### **Дополнительное чтение**

[Struct](https://go.dev/ref/spec#Struct_types)

[Properties of types and values](https://go.dev/ref/spec#Properties_of_types_and_values) • [type declarations](https://go.dev/ref/spec#Type_declarations) • [conversions](https://go.dev/ref/spec#Conversions)

[Method declarations](https://go.dev/ref/spec#Method_declarations) • [method expressions](https://go.dev/ref/spec#Method_expressions) • [method values](https://go.dev/ref/spec#Method_values)

[Pointers vs. Values](https://go.dev/doc/effective_go#pointers_vs_values)


---

### **Интерфейсы**

*Интерфейс* в Go — это набор сигнатур методов (то есть список методов без реализации). Можно воспринимать как контракт по которому есть требования и поставщик (структура) должна их реализовывать. 

Интерфейс геометрической фигуры:

```go
type geometry interface {
    area() float64
    perim() float64
}
```

Реализуем интерфейс в типе «прямоугольник». Реализовать интерфейс = реализовать его методы. Действует «утиный» принцип, как в питоне: если у типа есть перечисленные в интерфейсе методы — значит, он реализовал интерфейс. Явно указывать, что `rect` реализует `geometry`, не требуется:

```go
type rect struct {
    width, height float64
}

func (r rect) area() float64 {
    return r.width * r.height
}

func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}
```

Аналогично для типа «круг»:

```go
type circle struct {
    radius float64
}

func (c circle) area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c circle) perim() float64 {
    return 2 * math.Pi * c.radius
}
```

Если у переменной интерфейсный тип, она поддерживает все методы, заданные на интерфейсе. Благодаря этому функция `measure()` работает с любой фигурой, реализующей интерфейс `geometry`:

```go
func measure(g geometry) {
    fmt.Printf("%T: %+v\n", g, g)
    fmt.Println("area:", g.area())
    fmt.Println("perimiter:", g.perim())
}
```

Раз типы `circle` и `rect` реализуют интерфейс `geometry`, мы можем передать их экземпляры в функцию `measure()`:

```go
r := rect{width: 3, height: 4}
c := circle{radius: 5}

measure(r)
// main.rect: {width:3 height:4}
// area: 12
// perimiter: 14

measure(c)
// main.circle: {radius:5}
// area: 78.53981633974483
// perimiter: 31.41592653589793
```

[песочница](https://go.dev/play/p/bgOHM6Q3_Li)

### **Встраивание интерфейса**

Иногда при композиции хочется дать доступ к поведению, но скрыть структуру. В этом поможет *встраивание интерфейса* (interface embedding).

Есть тип «счетчик»:

```go
type counter struct {
    val uint
}
func (c *counter) increment() {
    c.val++
}
func (c *counter) value() uint {
    return c.val
}
```

Мы хотим встраивать счетчик в другие типы, но не давать прямой доступ к полю `val` — чтобы менять значение счетчика можно было только через методы.

Определим интерфейс счетчика:

```go
type Counter interface {
    increment()
    value() uint
}
```

И вместо конкретного типа `counter` встроим интерфейс `Counter`, который он реализует:

```go
type usage struct {
    service string
    Counter
}
```

В конструкторе будем создавать конкретное значение типа `counter`, но потребителям об этом знать не обязательно:

```go
func newUsage(service string) *usage {
    return &usage{service, &counter{}}
}
```

Поскольку мы встроили интерфейс, прямого доступа к `counter.val` больше нет. Можно использовать только методы интерфейса:

```go
usage := newUsage("find")
usage.increment()
usage.increment()
usage.increment()
fmt.Printf("%s usage: %d\n", usage.service, usage.value())
// find usage: 3
```

[песочница](https://go.dev/play/p/ZGBZWZJ_MVm)

### **Пустой интерфейс**

Если у интерфейса нет методов, его называют *пустым* (empty):

```go
interface{}
```

Пустой интерфейс может содержать значение любого типа (ведь у каждого типа есть как минимум 0 методов). Пустые интерфейсы используют, если тип значения заранее не известен. Например, функция из пакета `fmt`:

```go
func Println(a ...interface{}) (n int, err error)
```

`fmt.Println()` умеет печатать что угодно, поэтому принимает значения типа `interface{}`.

Начиная с Go 1.18 для `interface{}` ввели псевдоним `any`. Разницы между ними нет (псевдоним — это буквально тот же самый тип), но многие теперь предпочитают `any` за его краткость и выразительность.

```go
func repr(val any) string {
    return fmt.Sprintf("%#v", val)
}

func main() {
    var num int = 42
    fmt.Println(repr(num))
    // 42

    var thing interface{} = "shy string"
    fmt.Println(repr(thing))
    // "shy string"
}
```

[песочница](https://go.dev/play/p/-jx-g7HUdUP)

### **Приведение типа**

*Приведение типа* (type assertion) извлекает конкретное значение из переменной интерфейсного типа:

```go
var value any = "hello"
str := value.(string)
fmt.Println(str)
// hello
```

Если тип конкретного значения отличается от указанного, произойдет ошибка:

```go
flo := value.(float64)
// ошибка
```

Чтобы проверить тип конкретного значения, используют опциональный флаг, который сигнализирует — правильный тип или нет:

```go
str, ok := value.(string)
fmt.Println(str, ok)
// hello true

flo, ok := value.(float64)
fmt.Println(flo, ok)
// 0 false
```

### **Переключатель типа**

Приведение типа можно использовать вместе со `switch`. Такая конструкция называется *переключателем типа* (type switch):

```go
var value any = "hello"

switch v := value.(type) {
case string:
    fmt.Printf("%#v is a string\n", v)
case float64:
    fmt.Printf("%#v is a float\n", v)
default:
    fmt.Printf("%#v is a mystery\n", v)
}
// "hello" is a string
```

`v` внутри сработавшей ветки переключателя имеет конкретный тип вместо `any` (в примере — `string`).

[песочница](https://go.dev/play/p/kEq0UXg2KrE)

### **Интерфейсы и nil**

Внутри Go переменная типа `interface` представлена как пара `(type, value)`, где value — конкретное значение, а type — тип этого значения. Например:

```go
// переменная интерфейсного типа
var ivar interface{}

ivar = "hello"
// ivar представлена парой (string, "hello")

ivar = 3.14
// ival представлена парой (float64, 3.14)
```

Когда мы вызываем метод на интерфейсной переменной, Go вызывает соответствующий метод `value`:

```go
type greeter interface {
    greet()
}

type english struct {
    name string
}
func (e *english) greet() {
    fmt.Println("Hello", e.name)
}

var ivar greeter = &english{"world"}
// type == *english, value == &english{"world"}
ivar.greet()
// вызывает value.greet() и печатает "Hello world"
```

Пока интерфейсной переменной не присвоено значение, у нее и `type`, и `value` равны `nil`, поэтому сама переменная считается равной `nil`:

```go
var ivar any
// type == nil, value == nil
// поэтому ivar == nil
fmt.Println(ivar == nil)
// true
```

Но как только интерфейсной переменной присвоили значение, `type` перестает быть `nil`. Поэтому переменная больше не равна `nil`, даже если `value` равно `nil`:

```go
var e *english
fmt.Println(e == nil)
// true

ivar = e
// type == *english, value == nil
// поскольку type != nil, то ivar != nil
fmt.Println(ivar == nil)
// false
```

Пока интерфейсная переменная равна `nil`, вызвать метод на ней не получится (ведь `type` неизвестен):

```go
var ivar greeter
ivar.greet()
// panic: runtime error: invalid memory address or nil pointer dereference
```

Но когда тип известен — вызвать метод на интерфейсной переменной можно, даже если `value` равно `nil`. Поэтому в методах стоит учитывать, что получатель может быть `nil`:

```go
type greeter interface {
    greet()
}

type english struct {
    name string
}

// e может быть nil!
func (e *english) greet() {
    if e == nil {
        fmt.Println("I'm nil :(")
        return
    }
    fmt.Println("Hello", e.name)
}

var ivar greeter
ivar = (*english)(nil)
ivar.greet()
// I'm nil :(
```

Это контринтуитивная штука, поэтому ее имеет смысл запомнить.

[песочница](https://go.dev/play/p/l0AayMUNPfO)

### **Дополнительное чтение**

[Interfaces](https://go.dev/ref/spec#Interface_types)

[Type assertions](https://go.dev/ref/spec#Type_assertions)

[Interfaces and other types](https://go.dev/doc/effective_go#interfaces_and_types)

[Embedding](https://go.dev/doc/effective_go#embedding)

---

### **Ошибки**

В Go нет исключений и блока try-catch, как в питоне или js. Вместо этого функции явно возвращают ошибку отдельным значением. Благодаря этому ошибки невозможно проигнорировать, а разработчики продумывают поведение программы в случае проблем.

Ошибки принято возвращать последним значением с интерфейсным типом `error`:

```go
func sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, errors.New("expect x >= 0")
    }
    // `nil` в качестве ошибки указывает, что ошибок не было.
    return math.Sqrt(x), nil
}
```

Проверим работу `sqrt()` на положительном и отрицательном значениях. Обратите внимание, как мы получаем результат и проверяем ошибку внутри условия `if` — это стандартная практика в Go.

```go
for _, x := range []float64{49, -49} {
    if res, err := sqrt(x); err != nil {
        fmt.Printf("sqrt(%v) failed: %v\n", x, err)
    } else {
        fmt.Printf("sqrt(%v) = %v\n", x, res)
    }
}
// sqrt(49) = 7
// sqrt(-49) failed: expect x >= 0
```

[песочница](https://go.dev/play/p/ai4OtMECoDO)

### **Собственный тип ошибки**

Чтобы создать собственный тип ошибки, достаточно реализовать метод `Error()`.

```go
// фрагмент кода стандартной библиотеки
type error interface {
    Error() string
}
```

Создадим ошибку, которая описывает проблему поиска `substr` в строке `src`:

```go
type lookupError struct {
    src    string
    substr string
}

func (e lookupError) Error() string {
    return fmt.Sprintf("'%s' not found in '%s'", e.substr, e.src)
}
```

Напишем функцию `indexOf()`, которая возвращает индекс вхождения подстроки `substr` в строку `src`. Если вхождения нет, возвращает ошибку типа `lookupError`:

```go
func indexOf(src string, substr string) (int, error) {
    idx := strings.Index(src, substr)
    if idx == -1 {
        // Создаем и возвращаем ошибку типа `lookupError`.
        return -1, lookupError{src, substr}
    }
    return idx, nil
}
```

Проверим работу `indexOf()` для разных подстрок.

```go
src := "go is awesome"
for _, substr := range []string{"go", "js"} {
    if res, err := indexOf(src, substr); err != nil {
        fmt.Printf("indexOf(%#v, %#v) failed: %v\n", src, substr, err)
    } else {
        fmt.Printf("indexOf(%#v, %#v) = %v\n", src, substr, res)
    }
}
// indexOf("go is awesome", "go") = 0
// indexOf("go is awesome", "js") failed: 'js' not found in 'go is awesome'
```

Поскольку `indexOf()` возвращает общий тип `error`, чтобы получить доступ к конкретному объекту ошибки, придется использовать приведение типа:

```go
_, err := indexOf(src, "js")
if err, ok := err.(lookupError); ok {
    fmt.Println("err.src:", err.src)
    fmt.Println("err.substr:", err.substr)
}
// err.src: go is awesome
// err.substr: js
```

[песочница](https://go.dev/play/p/BoQblqUhep8)

### **Defer**

*Defer* позволяет отложить выполнение кода до момента завершения функции. Обычно его используют, чтобы освободить ресурсы, выделенные внутри функции (открытые файлы, соединения и тому подобное). В питоне в таких случаях применяют контекстные менеджеры, а в js конструкцию try-finally.

Допустим, мы хотим создать файл, записать в него что-то и закрыть. Вот как поможет `defer`:

```go
func main() {
    f, err := createFile("/tmp/defer.txt")
    if err != nil {
        fmt.Println("Error creating file:", err)
        return
    }

    defer closeFile(f)    // (1)

    if err := writeFile(f); err != nil {
        fmt.Println("Error writing to file:", err)
        return            // (2)
    }

    fmt.Println("Success!")
}
```

После того как файл открыт, мы с помощью `defer` указываем, что необходимо вызвать отложенную функцию `closeFile()` ➊. Она выполнится в самом конце, при завершении функции `main()`. Причем отложенная функция отработает в любом случае — даже если во время записи в файл произошла ошибка и сработал досрочный `return` ➋.

Допустим, создание файла пройдет успешно, а при записи случится ошибка:

```go
func createFile(name string) (*os.File, error) {
    fmt.Println("Creating file...")
    // ...
}

func writeFile(f *os.File) error {
    fmt.Println("Writing to file...")
    // эмулируем неминуемую ошибку
    return fmt.Errorf("oh no, it all went wrong!")
}

func closeFile(f *os.File) {
    fmt.Println("Closing file...")
    // ...
}
```

`closeFile()` все равно отработает:

```
Creating file...
Writing to file...
Error writing to file: oh no, it all went wrong!
Closing file...

```

[песочница](https://go.dev/play/p/SJU27B-8D3b)

### **Panic**

Если во время выполнения программы происходит неисправимая ошибка, срабатывает *паника* (panic). Это аналог исключения в питоне или js.

Допустим, мы написали функцию, которая возвращает символ строки по индексу, но забыли проверить, что индекс попадает в границы:

```go
func getChar(str string, idx int) byte {
    return str[idx]
}
```

Если вызвать `getChar()` с некорректным индексом — сработает паника:

```go
c := getChar("hello", 10)
// panic: runtime error: index out of range [10] with length 5
```

Панику можно вызвать и вручную с помощью одноименной встроенной функции:

```go
panic("oops")
```

Так редко делают — в Go принято возвращать ошибку из функции, а не паниковать.

[песочница](https://go.dev/play/p/ni8dgzRPB7V)

### **Recover**

Раз есть непредвиденные ошибки (паника), должен быть и способ их поймать. В Go для этого используется встроенная функция `recover()`. Посмотрим, как она работает.

Мы все так же забыли проверить, что индекс попадает в границы:

```go
func getChar(str string, idx int) byte {
    return str[idx]
}
```

Но зная свою забывчивость, решили отловить любые непредвиденные ошибки:

```go
func protect(fn func()) {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("ERROR:", err)
        } else {
            fmt.Println("Everything went smoothly!")
        }
    }()
    fn()
}
```

`protect()` первым делом объявляет анонимную отложенную функцию, которая сработает после того, как будет выполнена `fn()`. Если срабатывает паника, вызывается отложенная функция. Внутри нее `recover()` возвращает ошибку, которая вызвала панику. Если паники не было, отложенная функция тоже вызывается, но `recover()` внутри возвращает `nil`.

Здесь сработает паника:

```go
protect(func() {
    c := getChar("hello", 10)
    fmt.Println("hello[10] = ", c)
})
// ERROR: runtime error: index out of range [10] with length 5
```

А здесь функция отработает без ошибок:

```go
protect(func() {
    c := getChar("hello", 4)
    fmt.Println("hello[4] =", c)
})
// hello[4] = 111
// Everything went smoothly!
```

Возможно, вы заметили, что ручной вызов `panic()` в сочетании с `defer()` и `recover()` можно использовать, чтобы эмулировать конструкцию try-catch. В Go так не принято. Всегда старайтесь явно возвращать ошибки из функции, а на вызывающей стороне проверять их.

[песочница](https://go.dev/play/p/YP_kCsf4p-K)

### **Обертывание ошибок**

Допустим, есть функция, которая извлекает значение по ключу из карты и возвращает ошибку, если ключ не найден:

```go
var errNotFound error = errors.New("not found")

func getValue(m map[string]string, key string) (string, error) {
    val, ok := m[key]
    if !ok {
        return "", errNotFound
    }
    return val, nil
}
```

И есть тип `languages` с информацией о языках. Он возвращает описание языка по названию:

```go
type languages map[string]string

func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", err
    }
    return descr, nil
}
```

```go
var langs languages = languages{
    "go":     "is awesome",
    "python": "is everywhere",
    "php":    "just is",
}

func main() {
    descr, err := langs.describe("java")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(descr)
}
```

Внутри у `languages` карта, а метод `languages.describe()` использует `getValue()`, чтобы получить информацию о языке по названию. Получив ошибку, метод транслирует ее клиенту. В примере для `"java"` напечатается такой результат:

```
not found
```

Формально все верно. Но `errNotFound` — низкоуровневая ошибка общего назначения. Она ничего не говорит о проблеме с поиском языка. Клиент хотел бы больше информации.

Можно создать новую ошибку в `describe()` с помощью `fmt.Errorf()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", fmt.Errorf("error describing %s: unknown language", lang)
    }
    return descr, nil
}
```

```
error describing java: unknown language
```

Но создав новую ошибку, мы потеряли информацию о первоначальной `errNotFound`. Вдруг клиенту она важна?

Можно *обернуть* (wrap) исходную ошибку в новую с помощью `fmt.Errorf()`  и спецификатора `%w`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", fmt.Errorf("error describing %s: %w", lang, err)
    }
    return descr, nil
}
```

Теперь метод возвращает ошибку-матрешку: снаружи у нее информативная `error describing...`, а внутри исходная `errNotFound`. В сложных программах таких «обертываний» может быть много, пока ошибка поднимается от самых нижних слоев кода к уровню API или UI.

### **Обертывание собственных ошибок**

Если вместо `fmt.Errorf()` мы захотим использовать собственный тип ошибки — дело усложнится. Допустим, хотим записывать в ошибку название языка отдельным полем:

```go
type languageErr struct {
    lang string
}
 
func (le languageErr) Error() string {
    return fmt.Sprintf("%s language error", le.lang)
}
```

Чтобы `languageErr` могла выступать оберткой для других ошибок, придется сделать еще две вещи:

1. Добавить отдельное поле для внутренней ошибки (ее будем оборачивать).
2. Добавить метод `Unwrap()`, который возвращает внутреннюю ошибку («разворачивает»).

```go
type languageErr struct {
    lang string
    err  error
}

func (le languageErr) Error() string {
    return fmt.Sprintf("%s language error: %v", le.lang, le.err)
}

func (le languageErr) Unwrap() error {
    return le.err
}
```

Теперь можно создать новую `languageErr` как обертку над исходной ошибкой в методе `describe()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", languageErr{lang, err}
    }
    return descr, nil
}
```

Сама по себе слоеная ошибка — только половина дела. Вторая половина — научиться клиенту с ней работать.

### **errors.Is()**

Получив ошибку-матрешку, клиент может проверить, есть ли на каком-то слое интересующая его проблема. Для этого используют функцию `errors.Is()`:

```go
func (l languages) describe(lang string) (string, error) {
    descr, err := getValue(l, lang)
    if err != nil {
        return "", languageErr{lang, err}
    }
    return descr, nil
}

// ...

func main() {
    descr, err := langs.describe("java")
    if errors.Is(err, errNotFound) {
        fmt.Println("this is an errNotFound error")
        // do something about it...
    }
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(descr)
}

```

```bash
this is an errNotFound error
java language error: not found
```

Неважно, сколько в ошибке слоев. Если на каком-то из них встретилось значение `errNotFound` — `errors.Is()` вернет `true`.

[песочница](https://go.dev/play/p/bX5LaSSHHQa)

### **errors.As()**

Раньше мы использовали приведение типа, чтобы получить доступ к ошибке конкретного типа вместо абстрактного `error`:

```go
descr, err := langs.describe("java")
if langErr, ok := err.(languageErr); ok {
    fmt.Println("Language error:", langErr.lang)
}

```

```
Language error: java
```

Но это работает только для ошибки самого верхнего уровня. До ошибки из середины «матрешки» через приведение типа не добраться. А вот через `errors.As()` — можно:

```go
descr, err := langs.describe("java")
// обернем еще раз, чтобы languageErr
// оказалась внутрь матрешки
err = fmt.Errorf("wrap once more: %w", err)

var langErr languageErr
if errors.As(err, &langErr) {
    fmt.Println("Language error:", langErr.lang)
}
```

```
Language error: java
```

`errors.As()` проверяет каждый слой ошибки, и если видит там искомый тип `languageErr` — заполняет значение `langErr` по переданному указателю, и возвращает `true`. Если искомого типа нет — возвращает `false`.

[песочница](https://go.dev/play/p/mDY0C-7E32A)

Итого по обертыванию:

- Простой способ создать новую ошибку на основе существующей — `fmt.Errorf()` и спецификатор `%w`
- Если нужен собственный тип ошибки, придется добавить в него поле типа `error` и метод `Unwrap()`
- `errors.Is()` проверяет конкретную ошибку на каждом слое.
- `errors.As()` заполняет ошибку конкретного типа, если он встречается на одном из слоев.

Важное различие между `Is` и `As`: `errors.Is` ищет ошибку по совпадению *значения* ошибки, а `errors.As` по совпадению *типа* ошибки. Прежде чем двигаться дальше, проработайте примеры выше и убедитесь, что вы в этом разобрались.

### **Комбинация ошибок**

Если во время выполнения функции или метода произошла ошибка, обычно мы сразу ее возвращаем. Но иногда, если ошибок может быть несколько, полезно собрать их все и потом уже вернуть. В старых версиях Go сделать это можно было только вручную — например, завести срез с ошибками, заполнить его и вернуть. А в Go 1.20+ появился простой способ комбинации ошибок через `errors.Join()`:

```go
errRaining := errors.New("it's raining")
errWindy := errors.New("it's windy")
err := errors.Join(errRaining, errWindy)
```

Теперь `err` — это одновременно `errRaining` и `errWindy`. Стандартные функции `errors.Is()` и `errors.As()` умеют с этим работать:

```go
if errors.Is(err, errRaining) {
    fmt.Println("ouch! it's raining")
}
// ouch! it's raining

if errors.Is(err, errWindy) {
    fmt.Println("ouch! it's windy")
}
// ouch! it's windy
```

`fmt.Errorf()` тоже научилась комбинировать ошибки:

```go
err := fmt.Errorf("reasons to skip work: %w, %w", errRaining, errWindy)
fmt.Println(err)
// reasons to skip work: it's raining, it's windy
```

Чтобы принимать множественные ошибки в собственном error-типе, достаточно вернуть `[]error` вместо `error` из метода `Unwrap()`:

```go
type RefusalErr struct {
    reasons []error
}

func (e RefusalErr) Unwrap() []error {
    return e.reasons
}

func (e RefusalErr) Error() string {
    return fmt.Sprintf("refusing: %v", e.reasons)
}

err := RefusalErr{[]error{errRaining, errWindy}}
if errors.Is(err, errRaining) {
    fmt.Println("ouch! it's raining")
}
// ouch! it's raining
```

[песочница](https://go.dev/play/p/CftXuesNA1q)

### **Дополнительное чтение**

Спецификация: [errors](https://go.dev/ref/spec#Errors) • [defer](https://go.dev/ref/spec#Defer_statements) • [panic / recover](https://go.dev/ref/spec#Handling_panics)

[Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)

[Effective Go: Errors](https://go.dev/doc/effective_go#errors)

[Errors are values](https://go.dev/blog/errors-are-values)

[Working with Errors in Go](https://go.dev/blog/go1.13-errors)

---

Go — язык, который ценит простоту, но при работе со сложной бизнес-логикой требуется структурированный подход. Domain-Driven Design (DDD) — это именно такой подход, фокусирующийся на ядре приложения — его доменной области. На собеседованиях вопросы про DDD помогают понять, умеет ли кандидат моделировать сложную бизнес-логику и выстраивать коммуникацию с экспертами предметной области. Давайте разберём ключевые идеи DDD и их применение в Go.

### Domain-Driven Design (DDD)

Domain-Driven Design (Проектирование, управляемое предметной областью) — это методология разработки ПО, предложенная Эриком Эвансом, которая ставит во главу угла **домен** (предметную область) приложения. Основная цель — справиться со сложностью путём тесного сотрудничества разработчиков и экспертов предметной области, используя общий язык (**Ubiquitous Language**).

**Ключевые концепции DDD (Тактические паттерны)**:

- **Entities (Сущности)**: Объекты, обладающие уникальной идентичностью, которая сохраняется во времени (например, User, Order). В Go это обычно структуры с полем ID.
- **Value Objects (Объекты-значения)**: Объекты, определяемые своими атрибутами, не имеющие собственной идентичности (например, Address, Money). Часто делаются неизменяемыми (immutable). В Go это структуры, сравниваемые по значению полей.
- **Aggregates (Агрегаты)**: Кластер из одной или нескольких сущностей и объектов-значений, который рассматривается как единое целое. У агрегата есть **корень** (Aggregate Root) — сущность, через которую происходит всё взаимодействие с агрегатом. Это обеспечивает целостность данных внутри агрегата (например, Order с его OrderItems).
- **Repositories (Репозитории)**: Абстракция для доступа к данным агрегатов, имитирующая коллекцию объектов в памяти. Скрывает детали хранения (БД, файлы и т.д.). В Go это интерфейсы.
- **Domain Services (Доменные Сервисы)**: Логика, которая не принадлежит естественным образом ни одной сущности или объекту-значению. В Go это могут быть функции или методы структур-сервисов.
- **Domain Events (События Домена)**: Отражают значимые события, произошедшие в домене (например, OrderPlaced, UserRegistered).

**Пример в Go**:

Представим систему управления заказами.

```go

import "errors"

// Value Object: Money представляет сумму денег (неизменяемый)
type Money struct {
	Amount   int
	Currency string
}

func NewMoney(amount int, currency string) Money {
	// Здесь могут быть проверки
	return Money{Amount: amount, Currency: currency}
}

// Entity: OrderItem представляет позицию в заказе
type OrderItem struct {
	ProductID string
	Price     Money
	Quantity  int
}

// Aggregate Root: Order представляет заказ
type Order struct {
	ID         string
	CustomerID string
	Items      []OrderItem
	TotalPrice Money
	// ... другие поля статуса и т.д.
}

// Фабричный метод для создания нового заказа
func NewOrder(id, customerID string) *Order {
	return &Order{
		ID:         id,
		CustomerID: customerID,
		Items:      make([]OrderItem, 0),
		TotalPrice: NewMoney(0, "USD"), // Пример валюты по умолчанию
	}
}

// Метод на агрегате для добавления позиции (инкапсулирует логику)
func (o *Order) AddItem(productID string, price Money, quantity int) error {
	if quantity <= 0 {
		return errors.New("quantity must be positive")
	}
	if price.Currency != o.TotalPrice.Currency && o.TotalPrice.Amount != 0 {
        return errors.New("cannot mix currencies in an order")
    }
    if o.TotalPrice.Amount == 0 { // Если первая позиция, устанавливаем валюту заказа
        o.TotalPrice.Currency = price.Currency
    }

	item := OrderItem{
		ProductID: productID,
		Price:     price,
		Quantity:  quantity,
	}
	o.Items = append(o.Items, item)
	o.recalculateTotal() // Обновляем общую стоимость
	// Здесь можно генерировать Domain Event: OrderItemAdded
	return nil
}

func (o *Order) recalculateTotal() {
	total := 0
	for _, item := range o.Items {
		total += item.Price.Amount * item.Quantity
	}
	o.TotalPrice.Amount = total
}

package ports // Или infrastructure/repository

import "domain"

// Repository: интерфейс для работы с хранилищем заказов
type OrderRepository interface {
	Save(order *domain.Order) error
	FindByID(id string) (*domain.Order, error)
}

package application

import (
    "domain"
    "ports"
)

// Application Service: Оркестрирует использование доменных объектов
type OrderService struct {
    repo ports.OrderRepository
    // Здесь могут быть другие зависимости, например, для генерации ID
}

func NewOrderService(repo ports.OrderRepository) *OrderService {
    return &OrderService{repo: repo}
}

// Пример Use Case: Создать новый заказ
func (s *OrderService) CreateNewOrder(customerID string) (*domain.Order, error) {
    orderID := "some_generated_id" // В реальности генерация ID
    order := domain.NewOrder(orderID, customerID)
    err := s.repo.Save(order)
    if err != nil {
        return nil, err
    }
    // Здесь можно публиковать Domain Event: OrderCreated
    return order, nil
}

// Пример Use Case: Добавить товар в заказ
func (s *OrderService) AddItemToOrder(orderID, productID string, amount int, currency string, quantity int) error {
	order, err := s.repo.FindByID(orderID)
	if err != nil {
		return err // Заказ не найден
	}

	price := domain.NewMoney(amount, currency)
	err = order.AddItem(productID, price, quantity)
	if err != nil {
		return err // Ошибка бизнес-логики
	}

	// Сохраняем измененный агрегат
	return s.repo.Save(order)
}

package main // Или infrastructure/persistence

import (
	"fmt"
	"domain"
    "ports"
    "application"
)

// Пример реализации репозитория (InMemory)
type InMemoryOrderRepo struct {
	orders map[string]*domain.Order
}

func NewInMemoryOrderRepo() *InMemoryOrderRepo {
	return &InMemoryOrderRepo{orders: make(map[string]*domain.Order)}
}

func (r *InMemoryOrderRepo) Save(order *domain.Order) error {
	r.orders[order.ID] = order // Клонирование может быть полезно для избежания мутаций
	fmt.Printf("Order %s saved\n", order.ID)
	return nil
}

func (r *InMemoryOrderRepo) FindByID(id string) (*domain.Order, error) {
	if order, exists := r.orders[id]; exists {
		return order, nil // Возвращаем копию для защиты? Зависит от стратегии.
	}
	return nil, fmt.Errorf("order with id %s not found", id)
}

func main() {
	repo := NewInMemoryOrderRepo()
	orderService := application.NewOrderService(repo)

	// Используем сервис для создания заказа
	order, err := orderService.CreateNewOrder("customer-123")
	if err != nil {
		fmt.Println("Error creating order:", err)
		return
	}
	fmt.Printf("Created order: %s for customer %s\n", order.ID, order.CustomerID)

	// Используем сервис для добавления товара
	err = orderService.AddItemToOrder(order.ID, "product-abc", 1000, "USD", 2) // 2 штуки по $10.00
    if err != nil {
        fmt.Println("Error adding item:", err)
        return
    }

	err = orderService.AddItemToOrder(order.ID, "product-xyz", 550, "USD", 1) // 1 штука по $5.50
    if err != nil {
        fmt.Println("Error adding item:", err)
        return
    }

	// Получаем заказ снова, чтобы увидеть изменения
	updatedOrder, _ := repo.FindByID(order.ID)
	fmt.Printf("Updated order %s total price: %d %s\n", updatedOrder.ID, updatedOrder.TotalPrice.Amount, updatedOrder.TotalPrice.Currency)
    fmt.Printf("Items in order: %d\n", len(updatedOrder.Items))
}
```

**Почему это полезно для собеседований**:

- Показывает, что вы можете **моделировать сложную бизнес-логику** и выделять ядро системы.
- Демонстрирует понимание **Ubiquitous Language** и важность коммуникации с бизнесом.
- Помогает объяснить, как **защитить бизнес-правила** внутри агрегатов.
- Позволяет структурировать код так, чтобы он **отражал предметную область**, делая его более понятным и поддерживаемым.
- Подчеркивает умение работать с **абстракциями** (репозитории, доменные сервисы).
- Легко объяснить: "Я использую DDD, чтобы изолировать и защитить сложную бизнес-логику, сделав код ближе к реальным бизнес-процессам".
