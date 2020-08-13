---
title: "Golang Scratch Paper"
date: 2020-08-13T16:56:03Z
draft: false
---
*** <em>Đây là draft nhá</em> :3 ***

# Khai báo biến trong golang

## Khai báo chuẩn bằng từ khóa var và có thể kèm theo initializer:  
```golang
var(
  x int
  y int = 2
  z = "x"
)
```
Biến có thể định kiểu cụ thể. Nếu không thì nó sẽ được định kiểu với
giá trị truyền vào hàm khởi tạo cho nó(=), nếu không có giá trị khởi tạo thì giá trị mặc định là zezo value  

## Khai báo gọn ở trong hàm  
Sử dụng toán tử :=
```x := 2```
chú ý khai báo kiểu này chỉ tồn tại trong hàm. Nếu ở ngoài hàm được coi là error syntax  
Đồng thời cũng có thể gán hàm cho biến:
```var x = func test(){}```

# Constant  

Hằng trong golang chỉ có thể là character, string, boolean, numeric values  
Không thể khai báo bằng :=  
```golang
const Pi = 3.14
const (
  StatusOke       = 200
  StatusCreated   = 201
  StatusAccepted  =202
) 
```
# <strong>Packages</strong>  

## Import 

Chương trình golang tạo nên bởi các packages  
có thể import bằng:  
```golang
import(
  "fmt"
  "math/rand"
) 
```
Với những package không nằm trong các package tiêu chuẩn thì thường đc namespaced bằng
web url.  
Ví dụ:  
```import "github.com/mattetti/goRailsYourself/crypto"```
Sau khi khai báo thì ta cần phải pull package này về bằng command:  
go get github...
command trên sẽ pull thư viện về và đặt trong GOPATH  

Trong GOPATH gồm:  
-bin: Thư mục chứa những file compiled binaries  
-pkg: thư mục chứa các thư viện có sãn đã được biên dịch. Compiler sẽ  
chỉ cần link lại các thư viện này mà không cần compile lại nên rất nhanh  
-src: Gồm tất cả go source code tổ chức bằng import path  

## Export  

Sau khi ta import package vào file thì có thể gọi đến các names mà gói đó exported gồm có các biến, method, function mà cho phép gọi từ ngoài package.  
Trong go quy tắc của những name được exported bắt đầu với chữ in hoa.  
ví dụ math.pi thì ko đc vì nó ko phải tên đc export mà phải là math.Pi  

#Function và return value  

Trong go function có thể khai báo như sau:  
```golang
func test(x string, m, n int) int{
  return x, m
}
```
Kiểu trả về sẽ được khai báo sau tham số của function   
- go hỗ trợ trẩ về nhiều giá trị   
- Có thể khai báo luôn các biến trả về ở trên khai báo hàm. Khi đó chỉ cần gọi   
Return với không tham số truyền vào:   
```golang
func test(x string, m, n int) (x string, m int) {
  return
}
```
Tuy nhiên kiểu khai báo sãn biến trả về này không nên vì thấy khá rối  

# Pointer  

Pointer trong go có thể khai báo:
```golang
var x int = 2
var p *int = &x
```
Giống như trong C p chứa giá trị địa chỉ của giá trị 2. *p là giá trị 2 còn &x là địa chỉ của giá trị 2 được cấp.  
Nói chung ko có gì đặc biệt  
******Note trong go thì truyền arguments bằng giá trị chứ ko phải tham chiếu. Tương tự như C
Nếu muốn truyền tham chiếu thì phải dùng con trỏ hoặc một số cấu trúc dùng tham trị như slices hay maps.
Đồng thời method thì thường xuyên được defined trên pointer.  

# Mutability trong go  

Trong go thì chỉ có constant là immutable  Tuy nhiên bởi vì tham số truyền bằng
giá trị nên một func có thể thay đổi giá trị đó chứ không thể thay đổi giá trị thật mà biến ta truyền vào.

--------------------------Types----------------------------------
# Basic type  

Kiểu dữ liệu chung :  
bool string  
Kiểu số:  
uint(32 or 64 bit(tùy thuộc vào kiến trúc 32 hay 64 bit))  
int tương tự size uint  
uintptr  
int uint 8 hoặc 16 hoặc 32 hoặc 64  
float 32 hoặc 64  
complex 64 hoặc 128  
byte = uint8  
rune = int32 biểu diễn Unicode point  

# Conversion  

T(v) -> kiểu của v được convert sang kiểu T  
ví dụ:  
```golang
var i int = 1
var x  float32 = float32(i)
```
or  
```golang
i := 1
x := float32(i)
```
# Type Assertion  

Code sample: https://play.golang.org/p/EEOdNgO9Yau  

# Struct  

Struct tương tự như c, ta có thể khai báo như sau:  
```golang
type Person struct {
  Name string
  Age int
}
```

Có con trỏ struct ví dụ var x = &Person{"Thai", 22} thì có thể truy cập vào các biến ở trong struct. Ví dụ:  
```golang
var d = Person{"Thai", 12} 
d.Name -> getName
```

# Initializing  

New  expression sẽ cấp phátmột vùng nhớ zezoed value cho biến và trả về con trỏ đến vùng nhớ đó. Zezo value là giá trị mặc định cho kiểu : ví dụ boolean là false, int là 0, string là "", con trỏ, slide,
... là nil .
ví dụ:
```golang
type Person struct {
  Name string
  Age uint
}
x := new(Person)
y := &Person{}
-> *x = *y
```
Những cách khởi tạo giá trị: 
Nếu dùng new thì giá trị khởi tạo mặc định là zezo value. Ta có thể khởi tạo bằng cách:
```golang 
var x = Person{"Thai", 22}
var x = Person{
  "Thai",
  22,
}
```
# Composition vs Inheritance  

Trong Go không có inheritance, thay vào đó ta có composition(embedding)  

Ví dụ:
```golang
type Person struct {
  Name string
  Age uint
}

type Student struct {
  Person
  ID string
}
```
Bằng cách bên trên thì những thành phần(thuộc tính )của Person sẽ trở thành thuộc tính của Student
Ta có thể truy cập vào các thành phần bằng cách student.Name  student.ID ,...  

Một kiểu khác bên trên là thay vì chỉ truyền vào kiểu Person thì ta có thể truyền vào 1 con trỏ trỏ
Ví dụ:  
```golang
type Student struct {
  Person *Person
  ID string
}
```
Tuy nhiên với cách này thì ta sẽ không thể gọi trực tiếp thành phần của Person mà phải gọi:
```student.Person.Name```   
------------------------------------------------------Collection types------------------------   

# Array  

Khai báo  
```golang
var a [2]string
a := [2]string{"thai", "bui"}
```
ngoài ra cũng có thể dùng ... để khai báo ngầm length:
```a:= [...]string{"s", "a", "e", "d"}```

*****CHú ý là trong golang thì length cũng là 1 phần trong type . nghĩa là type []string là ko tồn tại
phải là [2]string. . Chính vì thế mà length của array là không thể thiếu  và không thể THAY ĐỔI
các array nhiều chiều thì khai báo [2][3]string chẳng hạn

# Slice   
Slice khai báo  
```s := []int{1,2,42}```
or
```s := make([]int, 5, 10)```
Với cách thứ hai thì các phần tử của slide sẽ là giá trị zezo value. đồng thời len và cap của slice là
tham số truyền vào.  

Thực chất slide build on top của array. 1 slide gồm có 3 trường thông tin: con trỏ trỏ đến array, length, cap
length và cap khác nhau ở chỗ length là số phần tử hiện tại của slice còn cap là số phần tử max mà slice đó có
thể chứa.  
Giả sử ta có a := [5]int  s := a[2:4] thì s lúc này có length là 2 và cap là 3. Điều này vì a chỉ có 5 ô nhớ int. s là lát cắt từ phần tử thứ 2 đến hết thứ 3 -> số phần tử max mà slice này có thể mở rộng chỉ có thể kéo đến phần tử index 4 của array. Nếu sang 5 thì sẽ là lỗi truy cập bộ nhớ quá
của array.
Điều này quan trọng khi ta xây dựng các hàm tiện dụng trên slice như append, hay copy.
Nếu số phần tử mà ta thêm vào những hàm append mà cap của slice đó không đủ chứa thì ta phải tạo 1 slice mới có cap to hơnvà thực hiện copy các giá trị của slice cũ vào.  

Nói chung slice build on top của array, cơ bản chỉ là con trỏ trỏ đến array và có trường cap tiện ích giúp tạo nên sự linh động.  

Ví dụ nhẹ cho giải thích trên:  
```golang
s := make([]int, 5, 6)
s = s[0: cap(s)]
```
len(s)-> 6. Tuy nhiên nếu thay cap(s) mà là 7 thì error slice bound ngay.

Hàm append hữu dụng khi ta muốn thêm phần tử cũng như slice khác vào slice.
```golang
cities := []string{"Hanoi", "HCM"}
cities = append(cities, "HD", "HP")
otherCities = []string{"Hue", DN}
cities = append(cities, otherCities...)
```
Muốn append slice vào slice thì cần phải dử dụng ellipsis(...). Bằng cách đó bộ biên dịch có thể hiểu và thực hiện lấy từng giá trị của otherCities thêm vào cities.  

Zezo value của slice là nil với length và capacity là 0.  

# Range trong for loop  

range giúp hỡ trợ ta lặp qua slice hoặc map, cả string nữa
ví dụ
```golang
s := []int{1,2,4,3,43,52,4}

for i, v := range s {
  // do somethings
}
```
Nếu không muốn lấy value thì chỉ cần bỏ v. Nếu chỉ lấy value thì _, v

# Map

Tương tự như trong các ngôn ngữ khác thì go cũng cung cấp kiểu hash:  
```golang
var student_ID = map[string]int{
  "Bui Xuan Thai": 3671
  "X":             3672
}
```
or
```golang
x := make(map[string]int)
x["Bui Xuan Thai"] = 3671
```
Xóa ```delete(x, "Bui Xuan Thai")```
Để kiểm tra sự tồn tai của 1 key trong map thì có 2 cách dựa trên vỉệc khi ta gọi x["key"] mà không tồn tại thì sẽ trả lại zezo value. Tuy nhiên cách này không phải lúc nào cũng áp dụng được vì nếu value thật của nó trùng với zezo value thì ko đc.  
Thay vào đó ta có:  ```elem, ok := x[key]```
nếu có tồn tại thì oke = true elem là giá trị trả về. còn ko thì ok = false và eleme là zezo value.  

***** Chú ý key trong map chỉ có thể sử dụng những kiểu dữ liệu có phép so sánh == như là int, float, complex
string, pointer, interface, struct array. Slice và map thì ko thể.  

------------------------------------------Control flow------------------------------------  

# If statement  

Trong go thì if không cần dấu (). Và có thể khai báo biến tại if:
```golang
if v := foo(); v <= 5 {
  // do somethings
} else {
  // do somethings
}
```
# For loop  

Trong go chỉ có duy nhất loop là for: cấu trúc cơ bản như sau:
```golang
for i := 0; i <= 10; i++ {
  // do somethings
}
```
Trong đó thì {} là bắt buộc  
for loop có thể không cần pre/post statêmnt như sau:
```golang
sum := 1
for ; sum < 1000 {
  sum += sum
}
```
for thay thế cho while:  
```golang
sum := 1
for sum < 1000 {
  sum += sum
}
```
# Switch

Sử dụng:  
```golang
x := 1
switch x {
  case 1, 4, 6:
    //do somethings
  case 2, 3, 5:
    //do somethings
  default :
    //do somethings
}
```
Từ  khóa fallthrough có thể sử đụng để thực thi tất cả các case từ vị trí case hợp lệ.. Khi sử dụng
fallthrough thì thường cần đi kèm với break để ngắt khi cần. Còn khi không định dùng fallthrough thì ko
cần dùng break trong go vì switch case trong go auto dùng break cuối mỗi case.

-------------------------------------------Method-------------------------------------------  

Quan hệ giữa method và function là method thì có một receiver xác định.
Đó cũng là ý tưởng method trong go. ví dụ:
```golang
type Person struct {
  Name string
  Age uint
}

func (p Person) say(statement string) string{
  return fmt.Sprints("%s say: %s", p.Name, statement)
}
```
Khi đó ta có thể gọi method với 1 receiver cụ thể dạng Person  
```golang
p := Person{
  "Thai",
  22,
}
```

khi đó ta có thể gọi ```p.say("Hello World!")```

# Tổ chức code  
```golang
package main

// list of package to import
import {
  "fmt"
}

// list of constants
const (
  ConstExample = "This is const"
)

// list of variables
var (
  Exportedvar = 24
  nonExportedVar = "No"
)

//Main type of file, try to keep lowest amount of structs per file when possible
type Person struct{
  Name string
  Age unit
  Location *UserLocation
}

type UserLocation struct {
  City string
  Country string
}

// list of functions
func newPerson(name string, Age uint) *Person {
  return &Person{
    Name: name,
    Age: age,
    &UserLocation{
      City: "HN",
      Country: "VN",
    },
  }
}

// list of methods
func (p Person) say(statement string) string {
  return fmt.Sprintf("%s say %s", p.Name, statement)
}

//func main if main package
func main() {
}
```
# Type aliasing  

Ta hoàn toàn có thể định nghĩa các method trên các type có sẵn.
T có thể alias một kiểu mới từ kiểu cũ:
```golang
type MyString string
func (myStr MyString) Upper() string {
  return strings.ToUpper(string(myStr))
}
```
Note: Chú ý phân biệt derived type và type alias
Code: https://play.golang.org/p/Ftq3eqsXN9D

#  Method receiver

Vấn đề: Một số method ta muốn nó thay đổi giá trị của receiver, một số lại không. Ví dụ như method
Abs() gọi trên 1 số int thì ta muốn nó trả về giá trị là abs mà không thay đổi receiver, Scale() ví dụ là method muốn tác động giá trị của int receiver thành ngược dấu nếu âm.

Điều đó có thể giải quyết bằng 2 cách khi ta định nghĩa method.  
Nếu ta truyền receiver là dạng pointer thì method có thể thay đổi giá trị của receiver.  
Nếu ta truyền theo giá tri thường thì tương tự như argumentsđược truyền vào sẽ copy giá trị và thay đổi trên nó trong khi receiver mà ta gọi thì không thay đổi gì.  
ví dụ:  
```golang
type MyFloat float64

func (mf MyFloat) Abs() uint {
  if mf < 0 {
    return -mf
  }
  return mf
}

func (mf *MyFloat) Scale() uint {
  if *mf < 0 {
    *mf = -*mf
  }
  return *mf
}
```

************** Chú ý go thì luôn pass mọi thứ bằng giá trị kể cả receiver. Kể cả khi ta truyền vào con trỏ
thì nó vẫn copy con trỏ nhưng chính bởi đặc tính lưu trữ địa chỉ của con trỏ mà nó có thể tương tác đến
origin value. Copy con trỏ là rất rẻ nên tốt. Nếu ko có cơ chế này thì ta sẽ phải thực hiện gán biến origin
cho giá trị trả về của hàm. Khi đó copy value lúc gọi hàm với những struct mà to thì sẽ rất tốn tài nguyên.  

--------------------------------Interface---------------------------------  

Cẩn trọng với những gì bạn gửi đi và tự do với những gì bạn nhận vào. Đây
là tư tưởng trong TCP. Ứng dụng vào go.Ta có:  
- Truyền vào một interface, trả về một kiểu cụ thể.

Tư tưởng interface trong go và java và ruby rất khác( quên hết java với ruby rồi :v)
-Trong java. Interface là 1 tập khai báo các method. Thể hiện 1 tập đặc tính nào đó. Tư duy sẽ là có 1 tập
hành vi đều tồn tại trên nhiều đối tượng nhưng lại có những cách ứng xử khác nhau trên từng loại đối tươg.
Từ đó sẽ gộp vào vào thành 1 khai báo riêng và các đối tượng múôn có nó chỉ cần implement nó. Đại loại là
cần tập methods nào thì implement tập đó.  
- Tương tự vậy trong ruby. được gọi là mixin tồn tại dưới dạng module.chứa nhiều biến, method, thậm chí cả
class. Những class muốn có đặc tính đó thường include hoặc extend module vào mình.  
- Tuy nhiên trái ngược lại thì trong golang, thể hiện tư duy ngược lại. Sau khi ta define 1 đống methods
ta nhận ra 1 đống method đó có hành vi về bản chất là giống nhau ở trên nhiều receiver. Ta sẽ gộp
các đối tượng có những hành vi đó vào 1 nhóm.Từ đó có thể định nghĩa ra những function mới xử lí nhóm đó
. Functions này thường là sử dụng kết quả của tập method kia để thực thi bên trong mình.  

************** Ta có thể tưởng tượng như sau: tất cả các type là 1 tập hợp. interfaces là các tập hợp con  

Như ví dụ về Reader và Writer ở dưới đây thì ReadWriter là giao của 2 tập hợp đó. 
```golang
ví dụ:
type Person struct {
  FirstName string
  LastName string
}

type Customer struct {
  FirstName string
  LastName string
  ID string
}

func (p *Person) Name() string {
  return p.FirtName + p.LastName
}

func (c *Customer) Name() string {
  return c.FirstName + c.LastName
}
```
Trên kia ta đã khai báo 2 struct có method Name(). Bây giờ ta muốn lnhóm 2 struct này vào 1 nhóm để có thể
define 1 function mà in ra câu chào dựa trên kết quả của method Name().Ta sẽ gom 2 struct này vào 1 nhóm gọi là
interface Namer như sau: 
```golang
type Namer interface {
  Name()
}
```
Định nghĩa function sử dụng interface này:  
```golang
func Greet(n Namer) string {
  return fmt.SPrintf("Dear %s", n.Name())
}
```
Sau đó ta có thể gọi function này:  
```golang
p := &Person("Bui", "Thai")
c := &Cusmtomer("John", "Smith", "1251")
```
# Statisfying  Interface  

Ta có thể tạo ra 1 interface bằng cachs tổng hợp những interface khác:  
```golang
type Reader interface {
  Read(b []byte) (n int, err error)
}

type Writer  interface {
  Write(b []byte) (n int, err error)
}

type ReadWriter interface {
  Reader
  Writer
}
```
# Returning Errors  


-------------------------------------Concurrency--------------------  

Keep in mind this is Concurrency,-----------------NOT PARALLEL----------  
# Goroutines  
Một go routine là một luồng nhẹ được quản lí bởi Go runtime. Cũng tương tự như khái niệm luồng trong các ngôn
ngữ khác.
Một ví dụ về goroutine như sau:
```golang
func say(s string) {
   for i := 0; i < 5; i++ {
    time.Sleep(100 * time.Millisecond)
    fmt.Println(s)
  }
}

func main() {
  go say("This is inside goroutine")
  say("This is in main goroutine")
}
```
Với ví dụ trên, Để tạo một go routine thì ta dùng từ khóa go ở trước hàm thực thi trong goroutine. Không có gì khó khăn.  
Vì đây là conccurency nên tại 1 thời điểm chỉ có một goroutine được hoạt động. Tất cả các luồng khác phải bị block. Tương tự như trong hệ điều hành
Chính vì vậy để hiểu tại sao cần time.Sleep() thì ta phân tích. hàm sleep này sẽ đưa goroutine hiện tại vào trạng thái block. Nghĩa là nó từ
bỏ quyền tiếp tục thực thi.Phải làm như thế thì mới có thể nhường cpu cho goroutine khác thực thi. Lịch khi đó sẽ  đến lượt goroutine khác chạy.
Chính vì vậy. Nếu ta bỏ hàm Sleep kia đi thì giả sử khi hàm say trong main goroutine chạy thì nó sẽ chạy một mạch, goroutine mà ta vừa tạo ra
sẽ không có quyền được thực thi cho đến khi main từ bỏ quyền hoặc làm xong việc. Tuy nhiên, khi main goroutine vừa chạy xong thì nó lại kết thúc
Thế là end luôn chương trình. Vì thế mà goroutine ta vừa tạo ra không có cơ hội thực thì thì đã bị end theo main. Kết quả là ta thấy in ra
toàn this is main goroutine.  

# Channel  

Khái niệm về channel ở đây tương tự như pipple trong linux hoặc chanel trong web socket. Về ý tưởng là tương đương.  
************** Chú ý: Trong go thì không giao tiếp để chia sẻ vùng nhớ mà thay vào đó là CHIA SẺ VÙNG NHỚ ĐỂ GIAO TIẾP.
channel được khởi tạo để goroutine truyền các giá trị đến nhau. Ta có thể tưởng tượng channel là 1 cái ống dẫn, 2 đầu là 2 goroutine.
Các bên có thể đặt giá trị vào đầu channel để gửi giá trị đến đầu kia cho goroutine kia lấy.  

Hai bên gửi và nhận sẽ block mãi cho đến khi channel sẵn sàng. Điều này cho goroutine đồng bộ mà không cần đến lock hay condition variable.
Về ý tưởng là như vậy.  

******Chú ý quan trọng với unbuffered channel: Khi 1 goroutine cố đọc data từ một channel nhưng channel không có giá trị nào cho nó. Nó sẽ block goroutine hiện tại và unblock goroutine khác với hi vọng một số goroutine khác sẽ đặt value vào trong channel. Bởi vậy thao tác đọc này sẽ bị chặn. Giống như hành vi
gửi data vào một channel thì goroutine hiện tại cũng bị block và unblock goroutine khác với hi vọng có một goroutine khác đọc giá trị từ channel.  

Ví dụ về sử dụng channel: 
```golang
func work(myChannel chan int) {
  data := 10
  myChannel <- data
}

func main() {
  myChannel := make(chan int)
  go work(myChannel)
  fmt.Println(<- myChannel)
}
```
Như ví dụ,ta đã khỏi tạo channel bằng hàm make. Mỗi channel chỉ vận chuyển một kiểu dữ liệu nhất định nên ta phải chỉ định kiểu khi khởi tạo.
Một ví dụ khác về tính tổng của 1 slide như sau: 
```golang
func sum(c chan int, s []int) int {
  sum := 0
  for _, v := range s {
    sum += v
  }
  c <- sum
}

func main() {
  s := make([]int)
  for i := 0; i < 100000; i++ {
    s[i] = i
  }
  myChannel := make(chan int)
  go sum(myChannel, s[:len(s)/2])
  go sum(myChannel, s[len(s)/2]:)

  firstSum, lastSum := <- myChannel, <- myChannel
  fmt.Println(firstSum + lastSum)
}
```
# Buffered channel  

Một buffed channel là 1 channel có cap cố định. Ta có thể coi nó như là 1 vùng buffer.  
Hành vi gửi data vào 1 buffed channel sẽ bị block nếu như buffered channel đầy(vùng buffer bị đầy).  
Hành vi nhận data sẽ bị block nếu buffered channel đang rỗng.  

Khởi tạo 1 buffered channel, ta thêm tham số về cap của channel:  
```myChannel := make(chan int, 100)```
-> Channel được tạo có thể chứa 100 số int.
Ví dụ sử dung :  
```golang
myChannel := make(chan int, 2)
myChannel <- 2
myChannel <- 3
fmt.Println(<- myChannel)
fmt.Println(<- myChannel)
```
-> Kết quả sẽ in ra 2 3  
Như đã nói ở trên thì channel vừa tạo chỉ có thể chứa max là 2 số int. Nếu ta tiếp tục gửi 1 giá trị nữa
vào channel khi nó đang chứa max 2 giá trị mà chưa được làm mới hay xóa 1 giá trị thì sẽ gặp deadlock
Ví dụ như ở bên trên ta tiếp tục đẩy thêm giá trị 4 vào ngay sau giá trị 3 được đẩy vào channel thì sẽ
gặp deadlock. Điều này xảy ra vì khi đó chính main channel sẽ bị block(vào trạng thái sleep). Go scheduler
sẽ tìm kiếm một goroutine khác để chạy nhưng vì chỉ có main goroutine nên ctrình rơi vào deadlock.  

Tuy vậy nếu trong trường hợp trên mà ta đẩy thêm giá trị trong 1 goroutine khác thì sẽ không sao:  
```golang
myChannel := make(chan int, 2)
myChannel <- 1
myChannel <- 2
x := func() { myChannel <- 3  }
go x()
fmt.Println(<- myChannel)
fmt.Println(<- myChannel)
fmt.Println(<- myChannel)
```
Với ví dụ này thì goroutine chạy hàm x() sẽ chờ cho đến khi ít nhất 1 giá trị đang ở trong channel được lấy
ra thì mới đẩy giá trị 3 vào.chính vì vậy mà không bị deadlock. Buffer channel có 2 thông số là len và cap tương tự như slice.  

#  Range and close

Goroutine sender  có thể close channel để chắc chắn sẽ không có giá trị nào được gửi nữa.  
Receicer có thể kiểm tra xem channel đã bị đóng hay chưa bằng:  
```v, ok  <- channel```
Nếu đã đóng thì v là zezo value, ok = false
Chưa thì v là value đwuọc lấy ra và ok = true

Code: https://play.golang.org/p/3h57D09V21q

for i := range channel sẽ thực hiện lặp đi lặp lại lấy dữ liệu từ channel cho đến khi nó đóng.
****** Chú ý: Chỉ có sender nê  đóng channel, gửi data trên một closed channel sẽ bị panic.
 ĐÓng channel chỉ cần thiết khi mà receiver phải được thông báo rằng sẽ không có giá trrị nào đến nữa.

Code: https://play.golang.org/p/9dFUcVouXon

# Select  

Các case trong select nếu match thì sẽ được thực thi
ví dụ sử dụng:
```golang
func fibonacci(outputChannel, quitChannel chan int) {
  x, y := 0, 1
  for {
    select {
    case outputChannel <- x:
      x, y = y, x + y
    case <-quitChannel:
      fmt.Println("quit")
      return
    }
  }
}

func main() {
  outputChannel := make(chan int)
  quitChannel := make(chan int)
  go func() {
    for val := range outputChannel {
      fmt.Println(val)
    }
  }()
  go func() {
    time.Sleep(10 * time.Millisecond)
    quitChannel <- 2
  }
  fibonacci(outputChannel, quitChannel)
}
```
Hàm hữu dụng: ```time.Tick()``` ```time.After()``` 
2 hàm này trả về 1 channel sau số thời gian ta truyền vào.

Code time.After: https://play.golang.org/p/mda2t2IQK__X  

Nếu select không có case dèault thì nó sẽ block cho đến khi có 1 case thỏa mãn.  
Nếu có default case thì nó sẽ kiểm tra các case kia. Nếu không thỏa mãn thì gouroutine sẽ không bị block như trường hợp ko có case nào thỏa mãn.
Ví dụ block này khá giống với khi ta call các service trong thực tế. Ta muốn nhận về kết quả của service mới nhanh nhất. Code: https://play.golang.org/p/DBPx5Sg9YQW  

Vậy còn nếu trong truòng hợp ta muốn nhận chờ đợi và nhận tất cả kết quả trả về từ mọi service được gọi thì sao:  
WaitGroup là thứ ta cần:  
Code ví dụ: https://play.golang.org/p/hDwtsRYueye  
Kiểu dữ liệu sync.WaitGroup là 1 struct có counter. Trước mỗi khi ta tạo 1 goroutine làm gì đó thì ta call wg.Add(1). Như vậy thì thành phần 
counter này tăng lên 1. Khi goroutine làm xong việc thì call wg.Done(). thành phần counter này sẽ giảm đi 1.
Và ở cuối cùng ta gọi wg.Wait(). Như vậy thì goroutine call Wait() sẽ chờ tất cả các goroutine kia xong. goroutine gọi hàm wait sẽ bị block
cho đến khi tất cả các goroutine kìa hoàn thành. Khi đó thành phần counter trong wg sẽ = 0.

# Channel 1 chiều  
Có những trường hợp ta chỉ muốn trong goroutine này chỉ có thể đọc hoặc ghi vào channel. Chính vì vậy mà goroutine cung cấp cho ta cách thức đó:
ví dụ func receiveOnly(c <-chan int) {} -> channel c ở trong goroutine này chỉ có thể read data.  
Code: https://play.golang.org/p/eLLmELyqDVJ  
6. <strong>Mutex trong go</strong>
Các goroutine dùng stack khác nhau, Tuy nhiên với biến heap thì hoàn toàn có thể xảy ra race condition nếu như nhiều goroutine muốn sử dụng nó.
Để tránh race condition thì ta có thể sử dụng mutex được cung cấp trong package sync. Tương tự như mutex trong các ngôn ngữ khác.
Code: https://play.golang.org/p/KJ7NL3yQhuS


/ Design patterns

Tại sao lại phải sửa lại cho draft đẹp đẽ chứ :3
