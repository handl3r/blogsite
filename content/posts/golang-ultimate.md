---
title: "Golang Ultimate"
date: 2020-08-16T17:30:47Z
draft: false
categories: [Golang-Basic, Draft]
---
*** <em>Đây là draft nhá</em> :3 ***   
*****Đây là những thứ mình note lại khi học khóa ultimateGo của William Kennedy  
Lesson 1.
Mở rộng đầu óc.
Go có cách riêng của nó.
Cố gắng suy nghĩ đơn giản.
Code phải đơn giản và có nghĩa, dễ đọc.
2 mục tiêu chính:
- code(kiến trúc) đơn giản, có nghĩa
- tránh chi phí ẩn trong các mẩu code nhỏ.
- Luôn giữ trong đầu sự tổng quan của hệ thống đơn giản.

Lesson 2.

Syntax

Nắm rõ tại sao và như thế nào syntax, tránh chi phí ẩn.

2.1 Variable

-data trong bộ nhớ là bit nên để đọc được nó có nghĩa thì cần có 1 type.
-khai báo với var thì khởi tạo với zezo value còn := thì phải gán giá trị.
-string trong go là 2 word, 1 là con trỏ đến địa chỉ đầu và 2 là số bytes của nó
-casting và conversion: casting thì mở rộng ô nhớ hiện tại ví dụ từ 1byte
thì thêm 3 bytes liên tiếp vào sau. -> ko an toàn vì có khả năng ghi đè vào
vùng của struct khác./// conversion thì tạo hẳn 1 vùng nhớ kác rồi copy vào
đó. castiing: [10] extend -> [10][][][]. conversion: [10] make new-> [][][][] copy -> [][10][][]

2.2 Struct

-ví dụ 
type bill {
  flag bool
  id int16
  cost float32
}
. Cost của 1 biến struct trên là 8 chứ không phải7 bytes.
Lí do: ta có các bounary trong bộ nhớ. Yý tưởng là không muốn lưu 1 value mà vượt qua ranh giới đó
-> cần có padding 
[flag][padding][int16][][float32][][][]
-> 8 bytes. Nếu id là int32 -> padding là 3 bytes, cost = 12 bytes.
Để optimize thì ta có thể sắp xếplại thứ tự các biến từ lớn đến bé: cost, id, flag
Đọc thêm về alignment struct trong golang: https://medium.com/@felipedutratine/how-to-organize-the-go-struct-in-order-to-save-memory-c78afcf59ec2

Tuy nhiên cần cân nhắc optimize vì có thể nó làm khó đọc code. Để dễ đọc code thì ta thường hay nhóm
các field có liên quan vào gần nhau.

- vấn đề về explicit và implicit convertion.
ví dụ ta có 2 struct alice va bob y hệt nhau được khai báo. Đây là 2 kiểu named type.
khi đó nếu ta gán a = b -> lỗi vì a không phải kiểu b nên không gán vậy được. Thay vào đó thì ta cần
ép kiểu của b thành a rồi mới gán : b = alice(b) a = b   (explicit)
tuy nhiên nếu ta khai báo 1 kiểu unamed type mà cấu trúc y hệt alice thì không cần conversion (implicit converion)

Code: https://play.golang.org/p/qH5CgqEQEC1


2.3 Pointer (pass by value, pointer sematic)
các vùng nhớ của 1 chường trình gồm segment, stack, heap (?)
-Khi 1 goroutine được thực thi thì nó được cấp 1 frame trong stack của chương trình.và nó chỉ có quyền sửa
đổi các biến trong vùng đó.
- Mỗi thời điểm có 1 goroutine chạy nghĩa là chỉ có 1 frame được activate. goroutine sẽ thay đổi các biến
trên vùng nhớ của nó.
- Môi trường trong từng frame này có thể coi như 1 sandbox độc lập.
- Điều này nảy sinh việc passbyvalue trong go. Khi đó khi ta gọi hàm khác trong hàm thì stack của hàm to
sẽ unactive, cờ active sẽ trỏ vào frame stack của hàm được gọi và goroutine bắt đầu thực thi trên vùng stack
 frame đó. Chính vì vây, biến mà ta truyền vào thực chất được copy giá trị và gán vào 1 biến mới trong vùng
frame stack của hàm được gọi. Mọi thay đổi của biến này không liên quan đến biến gốc trong hàm to.
-> nhu cầu về sủa đổi biến trong frame khác. -> truyền vào địa chỉ củâ biến trong frame khác. 

- Pass by value có cái giá là tồn tại nhiều bản ghi copy  của dữ liệu trong chương trình -> memory,... Đôi khi
rất phức tạp khi update giá trị các biến, balababa
Nhưng bù lại thì nó cung cấp tính isolation cho các vùng nhớ , tính integrity cho các dữ liệu( thứ mà go
rất coi trọng và đặt lên hàng đầu)

---Chính vì vậy mà ta cần cân bằng giữa value sematic và pointer sematic.

Mechanics là cách hoạt động còn sematic là Cachs cư cử (behave)

Code: https://play.golang.org/p/GWLkTqiMLym

2.3 Pointer part 2 sharing data

pointer sematic có chwúc năng để share data over diffrent frame.
Với pointer, ta có thể đọc ghi các biến nằm ngoài active frame. 
Chú ý, các địa chỉ nằm ngoài active frame mà ta có thể thay đổi phải nằm trong frame ở trên nó.
Nghĩa là ví dụ từ main ta gọi hàm test(p *int) thì địa chỉ mà biến p giữ phải nằm tronf frame của main, không
thể nằm ở những frame dưới. nó bởi tất cả những frame bên dưới là những frame có tính tạm. và có thể xóa đi 
để tái sử dụng khi hoàn thành xong chức năng. Điều này cũng xuất phát từ chính cơ chế của stack. 
Nói chung biến mà nó thay đổi phải nằm trong vùng frame được đặt vào stack trước.

Cost cho pointer sematic: tính integrity của data. pointer giống như là 1  mũi tên xuyên qua lá chắn
isolation, integrity của GO

Code: https://play.golang.org/p/I0WIEHkCiTO

2.3 Pointer(escape analysis)

Hãy tưởng tượng ta có ví dụ sau:

type user struct {
  name string
  age int
}

func createUser(name string, age int) *user {
  u := user{
    name: name,
    age: age,
  }
  return &u
}

func main() {
  u := createUser("thai", 2)
}
. Như các ví dụ ở trước đã phân tích:
1, 1 frame được cấp cho main tại đáy stack. Goroutine thực thi trên frame này. Frame này đang được activate 
2, Khi gọi đến hàm createUser() thì 1 frame mới được thêm vào đỉnh stack, Goroutine chuyển qua thực thi trên frame này, Frame này đang được active
3. Khi tạo biến u thì sẽ lưu value của nó trên frame đó.
4. khi return, goroutine chuyển lại thực hiện trên frame đầu tiên của main. Frame đó được activate. biến u tại main sẽ chứa địa chỉtrỏ đến
value u được khỏi tạo trong stack.
-> Phát sinh vấn đề ở đây: Như ta đã phân tích thì chỉ có thể thực hiện tác động đến giá trị trỏ bởi con trỏ mà giá trị đó nằm ở frame được đẩy
vào stack trước. Vì stack sau khi quay lại main thì nó còn dọn dẹp để có thể tái sử dụng cho khi gọi hàm khác. Điều đó dẫn đến nhiều vấn đề sai 
, quá sai.

Điều này đã được Go xử lí thỏa đáng bằng cơ chế escape analysis. Compiler sẽ thực hiện phân tích code và nó sẽ biến nếu thực hiện như trên thì 
có vấn đề. Khi đó thay vì tạo gía trị của user trong hàm createUser() trên frame stack của hàm này thì Nó sẽ tạo giá trị này TRÊN HEAP.
biến u trong hàm lúc này trỏ giá trị của nó vào giá trị ở heap. Khi ta trả về cũng là trả về địa chỉ của giá trị ở heap.

Nhờ có vậy mà Điều này có thể thực hiện trong GO và là tính năng tuyệt vời của GO.

Một chú ý ta cần nắm được là :
-stack thì tự nó có cơ chế dọn dẹp. Đó chính là cơ chế với zezo value. 
-heap được dọn dẹp bởi garbage collector

Một số lưu ý khi viết code: ví dụ nếu ta return &u như trên thì chỉ cần nhìn dòng return là hiểu được code. Tuy nhiên nếu ta khởi tạo kiểu
u := &user{balabala} rồi return u thì dòng return u không đem lại khả năng đọc code như cũ. 
Vậy nên ta nên sử dụng value semactic cho constructor. thay vì dùng pointer sematic. Trừ 1 cách viết chấp nhận được là u:= return &user{balalal}

Code: https://play.golang.org/p/RpE2slqMfR-

2.3 part 4 stack growth
-tại lúc compile time thì những biến có kích thước ko cố định sẽ được khởi tạo trong heap
-Bình thường thì 1 goroutine được cấp 2kb dung lượng stack. Với 1 goroutine thông thường thì đủ. NHưng có những trường hợp cần stack nhiều hơn
thì go có cơ chế để làm điều này. Nó sẽ tạo 1 stack mới, map các giá trị từ stack cũ sang, tuy nhiên với con trỏ thì nó sẽ sửa lại để
trỏ lại đúng vịtrí trên stack mới. Bằng cách này thì stack sẽ được mở rộng. 
-Các goroutine stack không thể có con trỏ trỏ qua nhau chính bởi vấn đề này. Nếu có điều này thì khi ta growth stack 1 goroutine thì 1 đống 
con trỏ trong những stack khác sẽ phải update lại giá trị mới.

--------------------------Remember, Go is about integrity first, it's about minimizing resources second-----------


2.3 pointer part 5 Garbage collection

garbage chạy thuật toán pacing alg để quản lí heap. ví dụ heap 4mb thì mỗi khi mà lượng data trong heap tăng
lên sát 4 thì ngay lập tức barbage sẽ chiếm lấy cpu để chạy thuật toán, cố gắng giải phóng các vùng nhwó
ko có tham chiếu đến trong heap để tăng ko gian trống trong heap.
tri-color: heap giống như 1 đồ thị. Khi mà mỗi node trong đó về cơ bản là 1 value, 1 cờ có 3 màu trắng đen,
xám. Các con trỏ trỏ từ heap, từ global variable, từ trong stack ra heap. Nếu mà các con trỏ này đc giải phóng or trỏ đi chỗ
lhác thì cờ được gán thành màu trắng. Garbage collector sẽ giải phóng nó. Nếu màu đen nghĩa là có reference
trỏ đến -> ko giải phóng.

Phải cân bằng giữa value sematic và pointer sematic khi ta viết code cũng là giúp cho heap đwuọc sử dụng hiệu quả,
garbage collector làm ít việc hơn và chương trình nhanh hơn.


2.4 Constant(xem lại nếu có thời gian)
trong go có 2 kiểu constant là:
-kind constant:
 const x = 1
-> ko có kiểu cụ thể. Tùy vào ngữ cảnh mà n có size khác nhau
độ chính xác 256 bits
implicit conversion
-type constant:
const x int64 = 1
explicit conversion
tham khảo thêm tại https://blog.learngoprogramming.com/learn-golang-typed-untyped-constants-70b4df443b61

constant chỉ tồn tại lúc compile time, lúc runtime thì ko tồn tại -> ko có địa chỉ

Code: https://play.golang.org/p/qDQi4AXvh5V

Lession 3
Data struct

3.2 Array part 1 

Mechanical Sympathy
(thỏa thuận với cơ chế phần cứng)
Tại sao GO chỉ có array, slice, map

Lấy 1 ví dụ : ta cần duyệt qua 1 mảng 2 chiều 1 triệu phần tử bằng 2 cách : row travesal, column travesal
và duyệt qua 1 linked list. Cách nào nhanh nhất.
THứ tự nhanh sẽ là row travesal - > column travesal -> linked list.

Các cơ chế về cached L1, L2, L3, main memory, processor. 
data vận chuyển từ L1 đến processor là cực nhanh, sau đó đến các cache L2, L3 rồi main memory là chậm.
Để chương trình chạy nhanh hơn thì ta cần các data của mình được nạp vào sẵn trong L1 hoặc L2 trước khi nó 
được processor lấy. cache chia thành các cache line, có 1 chương trình nhỏ ở processor(?) hay trong chip
luôn chạy sẵn, nó sẽ quyết định sẽ lấy data nào trong ram vào cache.

Chương trình này sẽ lấy các cache line trong ram vào cache. Nó sẽ ưu tiên các cấu trúc dữ liệu trên RAM mà
có tính predictable stripe (đại loại là ở liền kề nhau trên RAM). Chính vì vậy mà nếu data của ta
nằm liền kề nhau trên RAM thì sẽ được ưu tiên nạp vào cache.

Mechanical sympathy là các cơ chế mà giúp ta deal với phần cứng, os tốt hơn(?) đại loại là thế.
Trong java thì có JVM nó sẽ giúp chương trình của mình deal với mechanical sysmpathy nên dev không cần lo
phần này. Nhưng trong go ta không có 1 con máy ảo lo viêc đó như JVM nên dev phải deal với nó.

Array hay slice cơ bản là 1 tạo ra data struct tuân theo predictable access pattern thứ sẽ giúp phù hợp
với các cơ chế của cache để chương trình của ta nhanh hơn rất nhiều. Chính vì vây mà ta không thích linked
list hya stack, queue, balabala trong GO.Trong Go, slice ở mọi nơi.

Một điều nữa khiến cho linked list chậm là TLB(?) đại loại là bảng phân trang của OS thì linked list nó có
khả năg cao nằm rải rác trong nhiều page -> ko có trong TLB -> truy cập rất chậm vì phải tìm kiếm bala.

Bằng cách tuân thủ predictable access pattern thì peformance sẽ rất tốt.
Trong GO slice ở mọi nơi

Cost: Implement thuật toán lằng nhằng, đôi khi làm tính mở rộng thuật toán khó.

Code: https://play.golang.org/p/6FniMaJP2TZ (chạy trên máy chứ ko chạy trên playground vì time = 0s)

3.2 Array part2 
Sematics

Nói chung là không mix sematics (mix pointer sematic với value sematic) vì nó gây confused
Ví dụ:
for i, v := range &friends {
// do somethings
}
Nhìn ngáo vcl

3.3 Slice part1

empty struct là  1 kiểu mà ko đc cấp phát bộ nhớ. Nghĩa là chả tốn bộ nhớ gì
var es struct{}

Không có gì đặc biệt(24 bytes tương tự như giải thích trong tài liệu Note)
Lưu ý phân biệt giữa nil slice và empty slice

nil slice thì con trỏ có giá trị nil còn empty slice thì con trỏ trỏ vào emty struct (struct{})
var data []string -> data la nil slice -> sử dụng khi ta cần error 
data := []string{} -> data la empty string -> sử dụng khi không có ý định trả về error mà chỉ là nothings
trong collection

Code: https://play.golang.org/p/kACBS0R3LPH

3.3 Slice part 2 Append slice

Khi ta gọi data = append(data, something) ,
Tạo 1 bản copy của data(24 bytes).
Nó sẽ check xem length có bằng cap ko. Nếu không thì đơn giản
là ta gán giá vào element tiếp theo của slice giá trị something.
Nếu có thì khởi tạo mới 1 slice mới với cap là gấp đôi cap slice cũ rồi copy giá trị từ slide cũ vào
Trỏ con trỏ trong slice cũ sang slice vừa tạo.

3.3 Slice part 2 Append slice

Một số ví dụ:
slice1 := []string{"a", "b", "c", "d", "e"}
slice2 := slice1[0:2]
slice2 = append(slice2, "f")
-> slice1[2] = "f"
-> side effect 
Ta có thể chỉ định cap cho slice 2 bằng tham số thứ 3 : slice2 := slice1[0:2:4] -> cap = 2

Code: https://play.golang.org/p/XQF21AZ7FVi

code trên cho thấy sự thay đối khi ta append value vào slice. Đồng thời thấy reference khi ta dùng [:] để
tạo slice

3.4 Slice and references

Cẩn thận với memory leak khi dùng append slice. Ví dụ:
type user {
  likes int
}

func main() {
  users := make([]user, 2)
  shareUser := & users[1]
  shareuser.likes++
  // -> users[1].likes = 1
  users = append(users, user{})
  shareuser.likes++
  // -> shareUser.likes = 2 but users[1].likes = 1
}

3.5 Map

ví dụ: 
func main() {
  users := make(map[string]int)
  users["A"] = 1
  users["B"] = 2
  users["C"] = 3
  for k, v := range users {
    fmt.Printf("users[%s] = %d", k, v)
  }
}

Mỗi lần lặp vào map thì sẽ có 1 thứ tự random.
lặp range trong map thì sẽ random. Để không truy cập random như vậy thì có cách:

func main() {
  users := make(map[string]int)
  users["A"] = 1
  users["B"] = 2
  users["C"] = 3
  keys := []string{}
  for key, _ := range users {
    keys = append(keys, key)
  }
  sort.Strings(keys)
  for _, key := range keys {
    fmt.Printf("users[%s] = %d", key, users[key])
  }
}

Lesson 4

Decoupling

4.1 Methods part1 dclare & receiver behavior

receiver cũng là 1 tham số, nó cũng tuân theo value sematic. Ví dụ như sau:

type user struct {
	name string
	age int
}

func (u user) show() {
	fmt.Printf("Name: %s, Age: %d\n", u.name, u.age)
}

func (u *user) changeName(name string) {
	u.name = name
}

func main() {
	u := user{
		name : "thai",
		age : 22,
	}
	u.changeName("thang")
	fmt.Printf("Name: %s\n\n", u.name)
	
	y := &user{
		name : "thai",
		age : 22,
	}
	y.show()
	
}

Ở ví dụ trên ta thấy có 1 convenient khi ta gọi method, method changeName() yêu cầu rêciver là 1 con trỏ user
nhưng ta lại có thể gọi u.changeName(). tương tự khi ta gọi y.show() trong khi y là 1 con trỏ.
Điều này có thể vì trong GO khi gọi method thì ko quan tâm là dạng con trỏ hay value, điều mà 1 method quan
tâm là rêciver đó có dữ liệu cần từ đâu đó bởi rêciver truyền vào . Như vậy nên ta có thế gọi thế kia.

Code: https://play.golang.org/p/9T02TWlWApQ

4.1 Methods part2 value and pointer semantic

Phần này giúp ta quyết định khi nào dùng value semantic, khi nào dùng pointer semantic với rêciver trong 
method GO
TRong GO ta làm việc với 3 kiểu dữ liệu:
1. Built-in type: numeric, string, bool
2. Reference type: slice, map, channel, interface values, functions
3. struct type
//////////////////////////////////////
Luật đơn giản: built-in type -> value semantic
               reference type -> value semantic(đơn giản là ko có lí do gì để ta lấy address của address)
 trừ khi với slice và map mà ta muốn share nó thì dùng pointer type(?). Trường hợp ngoại lệ: decode và unmarshal

Code: https://play.golang.org/p/aoE7CrK7svI

Với struct thì ta phải tự quyết định semantic nào được dùng khi ta xây dựng struct đó.
Nếu không chắc chắc thì cứ dùng pointer semantic. Còn nếu chắc thì dùng theo ý.
Một số câu hỏi có thể giúp xác định semantic nào cho struct: Ví dụ như type Time. Nếu ta cộng thêm 5 giây
vào 1 value type Time thì nó là cùng 1 thứ hay là 2 thứ khác nhau. Tất nhiên là 2 thực thể khác nhau vì nó
là 2 thời điểm. 2 thằng tồn tại mà không loại trừ nhau. Tương tự như struct user. Tự hỏi nếu ta đối tên của
user thì đó là 2 người hay vẫn là 1 người. Tất nhiên là 1 người. Ta có thể map thực tế vào.

4.1 part 3

Func and method

-Tronf go thực chất ko có method như trong oop. method đều là func hết và tách riêng vs state. Điều này
 có thể chứng minh:
///////
1. method khai báo ngoài data. Có nghĩa là nó không phải 1 khối như oop. trong oop. State đi liền với method
và được khai báo bên trong class. Nhưng trong go, state và method không đi kèm với nhau như 1 khối. Như
ví dụ này :
type user struct {
	name string
	age int
}

func (u user) displayName() {
	fmt.Printf("Name: %s\n", u.name)
}

func (u *user) setAge(age int) {
	u.age = age
}

func inspect(u user) {
	fmt.Printf("Name: %s - Age: %d", u.name, u.age)
}

func main() {
	u := user{"thai", 22}
	u.displayName()
	u.setAge(2)
	inspect(u)
	fmt.Printf("\n---------------\n")
	user.displayName(u)
  (*user).setAge(&u, 2)
}
. Ta thấy nếu ta mà khai báo thêm 1 type : type bill user thì bill có những method của user ko???
Câu trả lời là không. Vì data của user type và method của nó không phải 1 cục.
+ cách gọi thực chất của method.là gọi theo function như trong ví dụ mô tả.Cách gọi method chỉ là 1 sugar 
syntax trong GO. khi xử lí, thay vì gọi như method thì sẽ gọi user.display(u) với u chính là receiver và cũng
là tham số đầu tiên. Chính vì vậy mà u cũng tuân theo value sematic, được copy giá trị.

Chốt lại method chỉ giúp chung ta tin rằng data có behavior nhưng thực chất thì không.

Code: https://play.golang.org/p/91lwAqC3aOY

/////////////////

- TRong go thì func cũng chỉ là value. CHứng minh:
f1 := u.displayName
f1()
f2 := u.setAge
f2(22)

Ta có thể gắn func với 1 biến và gọi func qua biến đó. Lúc đó biến f1, f2 sẽ như thế nào:
Biến có kích thước 2 word giống string. word đầu trỏ đến vùng code, word thứ 2 trỏ đến value là copy của u.
Tại sao lại là value copy của u. Đơn giản là value sematic
f1 : [pointer to code][pointer to a copy of u]
f2 : [pointer to code][pointer to origin of u] vì nó dùng pointer sematic

Code: https://play.golang.org/p/bxzOKi542xs

4.2 Interface part1
Polymophism
Đa hình là có thể viết một chương trình nhất định và nó hành xử khác nhau phụ thuộc vào data mà nó tính toán
trên đó.

1. interface giúp tạo ra đa hình trong GO. 
ta có thể khai báo interface:
type Reader interface {
  read([]bytes) string
}
Điều này có nghĩa là kiểu Reader được tạo ra không phải dựs trên 1 struct mà dựa trên interface
- Interface là 1 tyoe không có thật, không phải real data như các type khác
type pipe struct {
  name string
}
func (p pipe) read(slice []byte) (x string) {
  // do something
  return
}
bằng cách trên thì pipe đã implement reader interface.
tương tự vs 
type book interface {
  name string
}
func (b book) read(slice []byte) (x string){
  // do something
  return
}

Bằng cách trên thì book và pipe đã implement interface reader với value semantic

func retrieve(r reader) error {
  // do something
  return err
}

Bên trên ta có thể thấy  fun nhận vào type là 1 interface. Tuy nhiên trên thực tế thì type interface là
valueless -> Có vẻ vô lí. 
-THực chất thì dòng khai báo trên có nghĩa là nhận vào bất cứ kiểu concrete nào
mà có full set behavior của reader.

Sau đó trong main ta tạo 1 biến pipe là x: x := pipe{"Thai"} rồi call
retrieve(x)
-> Semantic ở đây là value semantic. x lúc này là 1 interface value 
Như ta đã nói x là valueless vì nó là gía trị của 1 interface . Vậy trong GO thì đã cài đặt nó thế nào.
-Thực tế nó là 2 word con trỏ
+trong đó word thứ 2 thì trỏ đến 1 bản copy của value type pipe mà ta đã tạo.(Bằng cách này, x vẫn là valueless)
nhưng nó lại trỏ đến 1 vùng concrete data, thứ mà có thể tính toán bên trên đó.
+ CÒn word thứ nhất thì trỏ đến 1 bảng là itable(tương tự nhwu vtable trong oop)
Trong bảng itable thì word đầu tiên lưu concrete type của x, phần còn lại thì là con trỏ trỏ đến 
func implement thực sự của method read mà sử dụng cho conrete type.
x : [pointer1][pointer2]
pointer1 -> bản copy of gía trịbiến x (chính là giá trị "Thai")
pointer2 -> itable
itable : [pipe][pointer3]
pointer3 -> [giá trịhàm implement method read cho type pipe]
-Quá trình gọi hàm từ interface type
Khi đó ở trong hàm retrieve mà ta gọi r.read(...) thì nó sẽ tìm từ word thu 2 của biến x trỏ sang iTable.
Tại đây nó sẽ tra được bẳng iTable TÌm được hàm read được implement cho kiểu pipe thực sự ở đâu và gọi đến

(nếu x là type book thì bản copy là bản value của book, itable lúc này word đầu lưu "book", còn lại trỏ 
đến vùng value implement func read của type book(nhớ là func cũng chỉ là 1 value trong go)) 

---- Cost: indirection: con trỏ trỏ đến con trỏ từ value interface đến itable đến func implement,..
và allocation của bản copy concrete data

4.2 Method Set and value

ví dụ ta implement interface reader cho struct user bằng pointer semantic.
sau đó ta define polymophism method với arg là type reader.: test(r reader)
Câu hỏi là u := user{...} thì có thể call được test(u) không. Câu trả lời là không vì method read() mà *user
implement không thuộc về method set của u
Tuy nhiên ta nếu ta truyền vào hàm test &u thì có thể sử dụng đươc cả pointer semantic và value semantic vì
Khi đó biến u có địa chỉ.
Với trường hợp trước đó thì chưa chắc biến u đã có địa chỉ nên không thể sử dụng u thay cho con trỏ u được.
CHính vì thế mà compiler không cho phép để tránh khi ta truyên vào 1 tham số ko thể lấy địa chỉ như
Chẳng hạn vs hằng số-> ko có địa chỉ vì nó chỉ tồn tại lúc compiler time còn lúc runtime thì ko có trong stack
hay heap.
Đại loại là nếu có địa chỉ thì có thể copy giá trị còn có giá trị thì chưa chắc đã có địa chỉ.
Luật:
<-------------------------------
T  _                valuesemantic
*T Pointersemantic  valuesemantic
Luật trên có nghĩa là nếu chọn pointer semantic thì chỉ có thể share. nếu chuyển từ pointer semantic sang 
value semantic thì ko nên. > Nói chung ko bao giờ nên chuyển từ pointer semantic sang value semantic.
Tuy nhiên từ value semantic thì ta có thể share (theo dòng thứ 2) nhưng chỉ nên dùng khi nó thaạt sự quan
trọng và cần thiết

4.2 Interface part 3 Storage by value
ví dụ như sau:
type user struct {
	name string
	age uint
}

type namer interface {
	showName()
}

func (u user) showName() {
	fmt.Printf("Name: %s\n", u.name)
}

func main() {
	u := user{"Thai", 22}
	namers := []namer{
		u,
		&u,
	}
	u.name = "Thang"
	for _, v := range namers {
		v.showName()
	}
}
. Kết quả :
Name : Thai
Name: Thang

Giải thích: literal constructor của user ta dùng value semantic. Khi ta truyền u vào thì nó sẽ tạo ra 1 bản
copy của u . namers cấu trúc nhwu sau:
index0: [user][pointer to value copy of u]
index1: [user][pointer to origin u]
Chính vì vậy khi ta thay đổi name của u là giá trị mới thì bản copy của nó ko hề thay đổi.
Tiếp theo range cũng dùng value semantic. và phần tử u mà nó làm việc với index 0 trong namers chỉ là bản
copy mà ko thay đổi theo  origin u -> kết quả như vậy.

4.3 Embedding

Khi ta nhúng 1 type vào trong type khác(chỉ nhúng type, chứ ko phải biến) thì là embedding. Khi đó thì
các behavior của type con sẽ promo lên trên type chứa nó.
Ta cần phân biệt:
type user struct {
  name string
  age uint
}
type admin struct {
  u user
  permission string
}
trường hợp trên không phải embedding. quan hệ này là quan hệ subtype

Tuy nhiên nếu ta thay đổi:
type user struct {
  name string
  age uint
}
type admin struct {
  user
  permission string
}
Thì đây mới chính là embedding. quan hệ là inner type và outter type
Để hiểu về promotion khi embedding, Đầu tiên ta xét ví dụ về subtype:
type user struct {
	name string
	age uint
}

type admin struct {
	u user
	permission string
}

func (u *user) displayName() {
	fmt.Printf("User Name : %s\n", u.name)
}

func main() {
	ad := admin{
		u: user{"Thai", 22},
		permission: "super",
	}
  ad.u.displayName()
	ad.displayName() // error vì đây ko phải embedding. 
}
Với trương hợp trên thì method của user không promo lên admin vì quan hệ ở đây là subtype, u chỉ là subfield
trong admin. Nên nếu muốn gọi method displayName() thì phải gọi trực tiếp qua ad.u.displayName().

Tuy nhiên nếu ta thay đổi thành embedding:
type user struct {
	name string
	age uint
}

type admin struct {
	user
	permission string
}

func (u *user) displayName() {
	fmt.Printf("User Name : %s\n", u.name)
}

func main() {
	ad := admin{
		user: user{"Thai", 22},
		permission: "super",
	}
  ad.user.displayName()
	ad.displayName()
}
Thì lúc này không phải  là quan hệ subtype mà là innter type và outter type. Lúc này các methods của user
sẽ promo lên admin nên có thể gọi trực tiếp ad.displayName() hoặc gọi ad.user.displayName()

Embedding nảy sinh vấn đề về override method.Nếu trong trường hợp ad cũng có method là displayName() thì sao?
Trong trường hợp này thì nó sẽ override method của innertype:type user struct {
	name string
	age uint
}

type admin struct {
	user
	permission string
}

func (u *user) displayName() {
	fmt.Printf("User Name : %s\n", u.name)
}

func (ad *admin) displayName() {
	fmt.Printf("Admin Name : %s\n", ad.name)
}

func main() {
	ad := admin{
		user: user{"Thai", 22},
		permission: "super",
	}
	ad.displayName()
	ad.user.displayName()
}
ad.displayName() sẽ gọi đến method của ad. Ta vẫn có thể sử dụng method của user bằng cách chỉ định trực
tiếp: ad.user.displayName()

4.4 Exporting
balabala. Ko có gì đặc biệt.


Lesson 5. Composition

5.1 Grouping type

- Nhóm các đối tượng theo hành vi của nó chứ ko phải trạng thái(interface nhóm theo methods set).
- Chỉ định nghĩa kiểu mới nếu nó là mới và ko trùng lặp.
- Embedding là để promo beahvior chứ ko phải để tận dụng state của inner type
- Alias để chia sẻ các state chứ ko chia sẻ behavior
Phân tích
Một số tư duy từ OOP có thể dẫn đến những đoạn code sau:

type Animal struct {
  Name string
  isMammal bool
}
func (a *Animal) Speak() {
  fmt.Printf("UGHm My name is %s, it is %t I am a mammal\n", a.Name, a.isMammal)
}
type Dog struct {
  Animal
  PackFactor int
}
func (d *Dog) Speak() {
  fmt.Printf("Gru gru, balabala")
}
type Cat struct {
  Animal
  ClimbFactor int
}
func (c *Cat) Speak() {
  fmt.Printf("Meow Meow, balabalaban")
}

func main() {
  animals := []Animal{ Dog{balabal}, Cat{balabal}, DOg{balbsda}}
}

Phân tích đoạn code trên cho thấy. người viết cố tình áp tư duy của oop vào GO:
1. Tạo ra lớp abstract là Animal với các thuộc tính, hành vi. Sau đó tạo 2 lớp con là Dog và Cat 
cố gắng tạo ra kế thừa bằng embedding lớp  Animal vào. 
Về cơ bản thì Dog và Cat lúc này thừ kế state của Animal và thừa kê vả behavior Speak() của nó, được phép
override method Speak() theo cách riêng.

Vấn đề: trong hàm main tạo slice []Animal với các element là Dog và Cat là sai. Trong Go không có kế thừa
như trong các ngôn ngữ OOP. Ở đây Animal là Animal, Dog là Dog, Cat là Cat, Dog hay Cat không phải là 1 Animal
. Chính vì vậy tư suy OOP áp dụng vào GO là sai.

Trong GO ta chia nhóm của các đôi tượng theo hành vi của nó, không phải trạng thái của nó. Ví dụ với 1 nhóm
20 người, nếu chia theo trạng thái(who we are), ví dụ như anh em hoawc chiều cao,... Nhóm sẽ rất hạn chế số lượng.
Vì gần như ko có ai là anh em. Tuy nhiên nếu ta chia theo hành vi(what we do) thì có thể chia cực kì đa dạng: nhóm người
có thể thở(tất cả), nhóm người có thể code, balabal -> Sự đa dạng khi chia nhóm theo hành vi. Điều đó phát 
sinh ra interface trong go giúp chia nhóm theo hành vi.

Một số điểm xấu trong thiết kế trên: 
1.Animal type được tạo ra cung cấp 1 lớp trừu tượng để tái sử dụng state
2. Chương trình gần như chẳng bao giờ cần tạo ra và sử dụng giá trị của kiểu Animal
3. Speak() của Animal được tạo ra là function tổng quát và gần như chẳng bao giờ được gọi.

Vậy ta có thể thay đổi thiết kế trên như thế nào:

type Speaker interface {
  Speak()
}
type Dog struct {
  Name string
  IsMammal bool
  PackFactor int
}

func (d * Dog) Speak() {
  // fmt.Printf("Gru guru abbababalala")
}
type Cat struct {
  Name string
  IsMammal bool
  ClimbFactor int
}
func (c *Cat) Speak() {
  // fmt.Printf("Meow babbalablablbal")
}

Bằng cách trên ta có thể tạo nhóm qua interface là tạo speakers := []Speaker{Dog{...}, Cat{...}}
Tuy nhiên đoạn code trên ta thấy có vấn đề về DRY khi mà ta lặp lạo=i các dòng code, thuộc tính Name,
Ismammal. Tuy nhiên việc này đem lại nhiều lợi ích hơn cho GO : dễ debug, dế test.

---------------Guideline khi khai báo type:
Khai báo 1 type mà biểu diễn thứ gì đó mới hoặc ko trùng lặp
Chắc chắn rằng chúng sẽ đc tạo vào sử dụng chứ ko phải để reuse state như trên.
embed type để tái sử dụng behaviors mà mình cần.
Không abstract hay alias thứ mà data thực sự chính là nó.(trong ví dụ dưới đây, handler thực chất là int
nên ko có lí do gì phải alias nó cả. khi dử dụng thì chỉ cần testMethod(handler int))
alias chỉ nhằm mục đích chia sẻ state, không liên quan đến behavior. chính vì vậy mà trong go khi ta alias:
type handler int
thì kiểu handler ko có các behavior có sẵn của int.
------------------

5.2 Decoupling part1

3 layer api:
unit test cho primitive layer -> primitive layer (làm việc với toàn concrete data)
-> unit test cho lower level -> lower layer
-> unit test cho high level -> high level

decoupling(tạo các interface, balabala) ở bứơc refactor code  

5.2 Decoupling part 2.
Solve problem in concrete data first then do: What can be decoupled

Bài toán: Giả sử ta có 2 hệ thống Xenia và Pillar có database khác nhau.
Pillar có db mà có api sẵn dùng để thêm,... dữ liệu.
Vấn đề: cần chuyển data từ db của Xenia sang Pillar ví dụ mỗi 5 phút.(dùng cron job)

Các task cần giải quyết ko phải vấn đề về performent trước mà là về concreate problem trước.
Cần phải chuyển data từ Xenia sang Pillar trước. Các vấn đề về decoupling, system change, babalâla ở
mãi sau đó mà ta chưa cần giải quyết.
1. how to connect to database
2. What data want to move from last 5 minutes
3.khả năng connect của pillar
4. làm sao d dể lưu trữ data nhận được vào trong db của pillar.
. Ta sẽ xử lí theo từng layer của API:
1. Primite layer
1.1 Define Xenia struct gồm host,.. -> giải quyết đc vấn vấn đề 1,
2.2 pull từng mảnh data nhỏ bằng method Pull(*d Data)-> vấn đề về hiệu năng Nhưng ở primite layer, ta chưa 
quan tâm về  hiệu năng. Trên thực tế thì ta chưa biết về hiệu năng, tốc độ mãi đến khi nó chạy. Nếu khi 
chạy mà hiệu năng nó đã đáp ứng tốt thì việc ta quan tâm vào hiệu năng từ bây giờ là thừa.
Ở primite layer, mọi thứ quan tâm là chương trình chạy được và có test.
-> giải quyết đc vấn đề 2.
Ta xây dựng method Stỏe nhận vào từng data -> giải quuyết đc cả 4 vấn đề.
-> hoàn thành primite layer với unit test
2. Lower layer
Ở layer này ta quan tâm đến làm sao để move 1 tá data thay vì từng data như ở primite
ý tưởng Ta define struct System và sử dụng compositing hoặc embeđing cả 2 struct Xenia và Pillar 
-> hệ thống biết cách để pull và store data(nhắc lại embedding, compositing vì behaviors chứ ko phải state)
Tiếp theo ta tạo ra 1 function pull như trong code và tài sử dụng method Pull() cho từng mảnh data trong 1
tá data mà pull nhận vào.
****Tại sao lại là function mà ko phải là method. Vì khi struct thay đổi thì có vấn đề. và nếu dùng method thì
thừa thông tin, ví dụ để gửi email, func cần email, tên người gửi, chứ ko phải toàn bộ thông tin trong
struct user, -> dùng func

3. High layer
Cần nhóm các thứ vừa xậy dựng lại với nhau -> func Copy, nhận vào  System và batch(số lượng data 1 lần)
Func này làm tất cả các việc mà ta đã định nghĩa ở trên

Code: https://play.golang.org/p/inHvXESl2So
---
Các vấn đề tiếp theo: Hệ thống thay đổi.Nếu có thêm1 hệ thống Bob muốn move data vào trong pillar và có 
hệ thống alice muốn xenia hoặc bob move data vào.

5.2 Decoupling part 3

Từ phần trước ta đã có concrete api. Với các vấn đề tiếp theo đã nêu ra, ta cần phải decouple hệ thống:Thêm
interface.

Ta thêm các interface Puller và Storer.Từ đó có thể giúp thỏa mãn vấn đề có thêm các hệ thống như alice,...
Code: https://play.golang.org/p/-wqle1kL3U2

Thay thế system với PullStorer interface
Code: https://play.golang.org/p/gVtI5YQgT0a

Sự hạn chế của code trên là ta vẫn chưa thực sựmở rộng được các loại puller và storer tham gia vào system.
Ta cần thay đổi thành phần của System thành Puller và Storer
Code: https://play.golang.org/p/1Mtaq85FeAI

Note: Khi ta truyền interface vào hàm ví dụ như Copy trong code thì nó không truyền interface value bởi
như đã thảo luận ở trước thì interface là valueless. Thực tế, nó sẽ truyền data mà nó giữa như trong cài 
đặt đã bàn ở phần trước.

Tiếp theo ta review lại code:
-Sự tồn tại của PullStorer là ko cần thiết
Code: https://play.golang.org/p/FTxnVih1xUj
- Sự tồn tại củâ system cũng ko cần thiết vì nó làm giảm khả năng đọc của code. 
Thay vào đó, ta có thể thiết kế high level : Copy(p Puller, s Storer, batch int)
Thiết kế này dễ đọc hơn.
Code: ...

Thứ tự khi ta code:
1. Layering
2. Testing through data
3. building ontop of each other
4. implement a concrete solution first
5. Decoupling
6. Read ability code review to find where can make faud or misuse

5.3 COnversion Assertions

Sự khác nhau giữa conversion và assertion:
- Conversion: để convert từ concrete type sang con conrete type.
- Assertion: convert từ interface sang 1 concrete type.
Code: https://play.golang.org/p/Kf4qgMpZ2Nw
Code thêm: https://play.golang.org/p/-TY6ZhpiKIH

Nếu ta biết concrete data mà 1 biến interface thực sự giữa là loại concrete data type nào. Ta có thể
sử dụng type assertion để convert từ interface sang đích xác type đó.



trong déign API ta có thể tạo ra default behavior và sau đó overide nó như sau:
Code: https://play.golang.org/p/zi5BmkGron-

5.4 Interface Pollution
Code: https://play.golang.org/p/K3w2eX7V1j2

Lesson 6: Error handling

6.1 Default Error value 

Code: https://play.golang.org/p/KkXbec2MzOF

6.2 Variable error

Code: https://play.golang.org/p/JQUJbS20MrE

6.3 Find the bug

Chú ý khi ta so sánh 2 variable interface
Vì var interface là valueless nên khi ta so sánh 2 biến này thì thực chất là so sánh concrete data mà nó
giữ. Ta có ví dụ như trong code sau:
Code: https://play.golang.org/p/tSf0yAxPaFj
Biến x lúc này là 2 word. Word 1 trỏ tới itable nơi giữ concrete data và các methò set mà nó implement
Word 2 trỏ tới địa chỉ của các value mà nó giữ(bởi vì ta sử dụng pointer semantic khi implement ShowName())
. Kết quả là 2 biến x, y có địa chỉ khác nhau.
Khi so sánh 2 biến n1, n2 thì là so sánh 2 địa chỉ với nhau -> khác.
Tuy nhiên nếu ta implement method ShowName() cho user bằng value semantic thì word thứ 2 của biến interface
trỏ đến value của biến user mà ta tạo. Khi so sánh thì sẽ là so sánh 2 value này chứ ko phải địa chỉ 
-> giống nhau vì đều là {"thai", 22}


































